# 个人项目复盘与面试准备

这是我的长期职业档案：保存做过什么、结果如何证明、面试怎样讲、下一步学什么。离开当前电脑后，只需克隆本仓库，就能继续维护简历、复习计划和求职进度。

当前求职主线：**AI 应用研发 / Agent 开发**；Applied ML 是第二优势。`main.tex` 已完成第一版重写与滴滴项目事实校准，南湖经历的量化结果仍待本人补证；版式待本人用 Overleaf 检查。

## 30 秒导航

| 我现在想做什么 | 先看 | 之后更新 |
|---|---|---|
| 改简历或准备投递 | [事实与指标台账](docs/evidence/01-事实指标与待补证台账.md) → [简历素材与修改记录](docs/resume/01-简历素材与修改记录.md) → [`main.tex`](main.tex) | 新数字先进入 `docs/evidence/`，确认后再写进简历 |
| 复盘滴滴项目 | [三个项目总览](docs/01-滴滴三个项目总览.md) → [项目总结目录](docs/didi/) | 项目事实更新到 `docs/didi/`，指标同步到 `docs/evidence/` |
| 复习或准备面试 | [复习系统入口](docs/study/README.md) → [知识地图](docs/knowledge/) → [面试材料](docs/interview/) | 学习进展记到 [复习进度](tracking/02-复习进度.md)，面试后使用[复盘模板](templates/面试复盘模板.md) |
| 换电脑后继续维护 | [当前状态](tracking/00-当前状态.md) → [换电脑续接说明](docs/handoff/01-换电脑后如何继续.md) → [`AGENTS.md`](AGENTS.md) | 先更新当前状态，再让新的 Codex 按协作规则接手 |

### 三条最常用路线

1. **今天开始复习**：打开[复习系统入口](docs/study/README.md)，按七天启动计划执行，再更新[复习进度](tracking/02-复习进度.md)。
2. **明天有面试**：先读对应的[项目总结](docs/didi/)，再练[高频追问题库](docs/interview/02-高频追问题库.md)和[STAR 故事库](docs/interview/01-STAR故事库.md)。
3. **针对一个 JD 改简历**：先用[JD 评估模板](templates/JD评估模板.md)判断匹配度，再核对[事实台账](docs/evidence/01-事实指标与待补证台账.md)，最后修改 `main.tex`。

## 三个滴滴项目怎样看

- **全国换电工单时效预测**：已交付项目，重点复习 Label、时序防泄漏、XGBoost、同样本评估、分桶分析与线上一致性；入口见[项目总结](docs/didi/01-全国换电工单时效预测.md)。
- **车辆阈值智能调整系统**：已实现核心执行链路，重点复习 LLM 结构化输出、确定性规则、配置语义、审批执行、审计与离线评测；入口见[项目总结](docs/didi/02-车辆阈值智能调整系统.md)。
- **城市异动诊断 Agent**：当前是基于既有动作底座的下一阶段方案，不能当成上线成果。[面试摘要](docs/didi/03-城市异动诊断Agent.md)负责讲清现状和边界，[完整建设蓝图](docs/city-agent/README.md)负责未来设计与实现。

精确数字只认[事实与指标台账](docs/evidence/01-事实指标与待补证台账.md)，避免多个摘要各自维护后发生漂移。

## 目录说明

```text
.
├── README.md                 # 给自己的一页总入口
├── AGENTS.md                 # 给未来 Codex 的事实边界和协作规则
├── main.tex                  # 当前简历源文件；用 Overleaf 编译
├── docs/
│   ├── didi/                 # 三个项目的面试版总结
│   ├── city-agent/           # 城市诊断 Agent 的未来建设蓝图
│   ├── evidence/             # 数字、口径、证据等级和待补项
│   ├── resume/               # 简历素材、取舍依据和修改记录
│   ├── study/                # 七天、六周、题库与能力改善计划
│   ├── knowledge/            # Agent/RAG/Eval、ML、LLM、后端复习
│   ├── interview/            # STAR、技术追问和行为面准备
│   ├── reflection/           # 只供个人看的工作反思
│   ├── market/               # JD 方法与带日期的市场快照
│   ├── handoff/              # 换电脑和离职前续接说明
│   └── archive/              # 原始资料的迁移索引与历史摘要
├── tracking/                 # 当前状态、学习、投递和项目进度
└── templates/                # JD、面试、周复盘和项目管理模板
```

补充说明：`docs/knowledge/` 用于复习通用原理，不代表项目已经实现；城市 Agent 的完整实施方案以 `docs/city-agent/README.md` 为唯一入口。

## 长期维护规则

- **事实口径**：每个数字都要能解释样本、时间、对照、计算方法和限制；不确定就标记待确认。
- **状态边界**：方案不能写成上线成果，离线回放不能写成生产 SLA，前后变化不能直接写成因果收益。
- **保密边界**：公司源码、完整 SQL、原始日志、内部地址、凭证和真实业务数据不进入 Git，只保留脱敏方法、聚合指标和个人贡献。
- **更新顺序**：获得新证据时先更新 `docs/evidence/`；项目叙事随后更新；确认能承受追问后才同步到 `main.tex`。
- **进度续接**：每次集中维护结束前更新 `tracking/00-当前状态.md`。Git 提交和推送仍需本人当次明确授权。

## 更多入口

- 工作反思与改善：[年度复盘](docs/reflection/01-第一段互联网工作复盘.md) / [三个月改善计划](docs/study/05-三个月能力改善计划.md) / [能力计分板](tracking/03-能力改善计分板.md)
- 城市 Agent 实现：[完整蓝图](docs/city-agent/README.md) / [LangGraph 架构](docs/city-agent/02-LangGraph架构与状态机.md) / [实施与运维](docs/city-agent/05-实施路线图与运维手册.md) / [实施进度](tracking/04-城市Agent实施进度.md)
- 求职市场：[JD 筛选方法](docs/market/01-岗位方向与JD筛选.md) / [带日期的岗位与面经快照](docs/market/02-2026-09-04岗位与面经快照.md) / [投递台账](tracking/01-投递台账.md)
- 离职前检查：[最小资料清单](docs/handoff/02-离职前最小资料清单.md) / [原始资料迁移状态](docs/evidence/04-原始资料来源与迁移状态.md)
