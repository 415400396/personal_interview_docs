# 城市诊断 Agent 系统设计

更新日期：2026-09-04

用途：这是技术面试深挖材料，不是已上线项目证明。

> 本文保留为十分钟复习版。面向生产约束的详细设计已经拆分到 [城市异动诊断 Agent 完整建设蓝图](../city-agent/README.md)，包括现状差距、产品范围、LangGraph State/节点/边、数据与工具契约、微调门槛、离线与线上评测、SLO、最早约 20 周且以阶段闸门为准的实施计划和运维 Runbook。两处冲突时，以详细蓝图和事实台账为准。

## 1. 四类组件的边界

| 组件 | 解决的问题 |
|---|---|
| Function Calling | 模型表达要调用哪个工具、参数是什么 |
| MCP | 标准化工具发现、Schema、连接和能力暴露 |
| Workflow | 管理节点、状态、分支、预算、重试、暂停和恢复 |
| 规则/统计引擎 | 负责数值计算、硬约束、风险边界和执行资格 |

MCP 不等于 Agent，Function Calling 不等于执行，Workflow 也不应把所有业务判断交给模型。

## 2. 最小可用状态

```text
request_id
trigger / scope
data_quality
evidence_refs / derived_fact_summary
hypotheses
evidence_hypothesis_relations
tool_trace
recommendations
risk_level
product_review_status
downstream_approval_status
execution_status
observation_status
```

关键约束：

- `Evidence` 只由受控工具或确定性计算产生；`derived_fact_summary` 必须能回指 `evidence_id`；
- hypothesis 必须引用 evidence_id；
- recommendation 必须通过规则；
- 每次工具调用保存数据时间、参数摘要、耗时和结果；
- 产品审核与下游 BPM 分开建模，二者都绑定 recommendation_version 与 config_version；
- 回调恢复时先做取消、幂等、权限与配置版本检查，审批通过不等于执行已生效。

Graph State 是事实引用、推断结果和流程控制状态的混合容器，不是事实源；不可变 Evidence 存在独立事实存储中，State 只保存引用和可验证摘要。

## 3. 建议的只读工具

工具名、输入输出 Schema、超时、缓存和失败语义以[数据、工具与模型策略](../city-agent/03-数据工具与模型策略.md)第 6.1 节的 canonical registry 为唯一来源。首版固定为指标与基线、供需与能力、配置与变更、批准的知识、相似案件、Dry Run、通知草稿和后验效果等 11 个受控领域工具；天气、活动、发布等只是相应 Summary 的受控字段，不为每种数据新造一个任意查询工具。

工具网关必须限制身份、租户与城市范围、指标白名单、时间跨度、行列、超时、脱敏和审计；模型不能提交任意 SQL。MCP/HTTP 工具只负责只读取证和模拟。产品审核后的 BPM 提交、执行、终态查询及通知发送由应用侧 Action Adapter 负责，不暴露为 LLM 可选择的模型工具。

## 4. 有界 Agent Loop

```text
范围解析
→ 数据质量闸门
→ 确定性异常检测
→ 最多 N 个候选假设
→ 有预算的工具下钻
→ 证据评分
→ 建议或 ABSTAIN
→ Dry Run
→ 产品审核
→ Action Adapter 可靠提交下游 BPM
→ 执行终态回查
```

防止路径震荡：

- 每个假设声明需要和反驳证据；
- 同一参数工具调用去重；
- 限制总步数、单工具重试和总耗时；
- 新证据必须减少不确定性，否则 early stop；
- 多次冲突后转人工，而不是继续循环。

## 5. 置信度与 ABSTAIN

不要让 LLM 自报一个百分比。程序先根据以下部分计算可校准的 `evidence_score`：

```text
数据完整度 + 指标一致性 + 基线稳定性 + 证据/反证覆盖 + 历史校准误差
```

HIGH/MEDIUM/LOW 档位由程序把 `evidence_score` 映射得到，而不是靠固定证据条数或模型自信。建设初期只把该分数用于排序和拒答；有代表性验证集后，再用 Brier/ECE、risk-coverage 曲线及城市规模/根因等切片的置信区间校准阈值。数据过期、口径异常、关键反证缺失或证据冲突触发 ABSTAIN，不生成修改命令。

## 6. 推荐与执行安全

- LLM 只给可能的动作方向和理由；动作资格、最终方向、数值或区间由确定性规则/优化器计算；
- 建议包含范围、原因、证据、预期、风险和回滚条件；
- Dry Run 前检查当前配置、在途审批和冷却期；
- 产品审核通过后，由服务端 Action Adapter 做原子动作预留并可靠提交下游 BPM；模型没有提交权限；
- 用案件、不可变建议版本、目标和动作生成幂等键，数据库原子 reservation 后再经 Outbox 提交；
- 同一幂等键必须绑定同一 payload hash，下游也保存该键；
- DB 状态未知时先查执行状态与最终配置，不能盲目重写；
- 回滚先生成建议，不让 Agent 自动执行高风险回滚。

## 7. 降级设计

| 故障 | 降级 |
|---|---|
| 数据分区未就绪 | ABSTAIN，返回缺失数据与预计恢复时间 |
| 单工具失败 | 有限重试，随后降置信或终止 |
| LLM 不可用 | 只返回确定性异常事实 |
| BPM 不可用 | 保存 Dry Run，不执行 |
| 执行状态未知 | 查询幂等状态与最终配置 |
| 效果证据不足 | 延长观察或转人工 |

## 8. 评测

### 离线数据

每个历史案例保存告警时刻可见快照、真实异常类型、关键证据、专家结论、是否该调、允许范围、审批和后续效果。证据必须满足 `event_time <= decision_cutoff`，且规范化后的 `available_at/ingested_at <= decision_cutoff`，避免事后补录数据穿越；allowed lateness 只能改变数据质量状态，不能放入当时尚不可见的数据。

### 指标

- 范围解析正确率；
- 工具选择和参数正确率；
- 关键证据引用覆盖率；
- 异常类型 Top-K；
- 建议合法率和越界阻断率；
- ABSTAIN 正确率；
- 平均步数、工具数、延迟和成本；
- 重复 BPM/重复执行；
- 建议采纳率与带对照的业务效果。

采纳率不是越高越好；证据不足时拒绝行动是正确行为。

## 9. 分阶段落地

1. P0–P2：冻结 Baseline、Episode/Evidence 契约，并完成可恢复的只读 Workflow 骨架；
2. P3：只读诊断，只输出事实、证据、Runbook 和 ABSTAIN；
3. P4：加固共享校验、版本、幂等和执行终态，输出建议与 Dry Run，不提交；
4. P5–P6：先 Replay/Shadow，再向人展示 Pilot 结果；
5. P7：人工确认后做小范围 Canary，执行终态可证后才进入观察；
6. P8：用同期对照评估效果，决定 Scale 或回退，并持续维护 Badcase。

## 10. 面试官最可能追问

- 为什么这里需要 Agent，而不是固定 Workflow；
- 什么条件下应退化成 Workflow；
- MCP 与普通内部 API 有什么实际差别；
- 工具很多时怎样选择和控制 Context；
- 怎样验证证据没有被模型编造；
- 如何终止循环、处理路径震荡；
- 审批跨小时后如何恢复；
- 怎样避免重复提交和重复写；
- 如何设计离线 Gold Set 和线上指标；
- 如何证明调整有效，而不是天气或供给变化。
