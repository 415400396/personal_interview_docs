# ML、LLM 与系统基础复习边界

目标是服务 AI 应用 / Agent 和 Applied ML 面试，不在第一阶段追求基座模型研究岗或底层 Infra 的全部深度。

## 1. Applied ML：必须到项目级

### 问题与数据

必须会：

- 业务目标怎样映射到样本、Label 和模型输出；
- 样本粒度、观察窗口、预测窗口和可用时间截点；
- 训练/验证/测试切分，尤其是时间 OOT；
- 选择偏差、幸存者偏差、重复实体和未来信息泄漏；
- 缺失、异常、长尾、类别不平衡与分布漂移；
- 数据质量和 Join 唯一性检查。

项目锚点：时效 Label 语义迁移、日期 OOT、同工单对照和短时桶退化。

### 模型与实验

必须会：

- baseline、单变量实验、消融与超参数搜索；
- 过拟合、偏差/方差、正则化和早停；
- 树模型如何分裂、处理非线性与特征交互；
- 特征重要性、permutation 和 SHAP 的区别与局限；
- 总体、分层、同样本和长尾指标；
- 离线、服务和业务三层指标分离。

不需要把 XGBoost 每个参数都背下来；需要能解释最终参数影响、为何选择稳定方案、如何验证。

### 常用指标边界

| 指标 | 适合 | 风险 |
|---|---|---|
| MAE | 回归平均绝对误差、解释直观 | 对大错惩罚线性，可能掩盖分桶差异 |
| RMSE | 更重视大误差 | 对长尾敏感，易被少数异常支配 |
| MAPE | 相对误差 | 真值接近 0 时不稳定 |
| Quantile Loss | 分位数和非对称风险 | 需要校准与业务解释 |
| Spearman | 排序一致性 | 不表示数值校准正确 |
| Precision/Recall/F1 | 分类与不均衡任务 | 阈值变化会改变结果 |
| ROC-AUC | 全阈值排序 | 极不均衡时可能过于乐观 |
| PR-AUC | 稀有正例 | 对正例定义和基准比例敏感 |
| Recall@K | 是否召回相关证据 | 不评价排序位置和最终回答 |
| NDCG@K | 分级相关性的排序质量 | 依赖可靠相关性标注 |

## 2. LLM 基础：应用岗到能解释取舍

### Transformer

必须会：

- Token → Embedding → Attention/MLP 残差块 → 输出；
- scaled dot-product attention 的 Q/K/V、形状和复杂度；
- causal mask 与 padding mask；
- 残差、LayerNorm、Pre-Norm/Post-Norm；
- 训练时 teacher forcing 与自回归推理的差异；
- Temperature、top-p、max tokens 对稳定性和延迟的影响。

### 推理结构

至少能比较：

- MHA/MQA/GQA：质量、KV Cache 和并行取舍；
- KV Cache：为何降低重复计算、为何占显存；
- RoPE 与长上下文扩展的基本思想；
- MoE：稀疏激活、路由、负载均衡和部署复杂度；
- 量化：显存/吞吐收益与精度风险；
- batching、流式输出、TTFT 与 tokens/s。

对 CUDA Kernel、FlashAttention 实现和大规模并行，只在目标 JD 要求时深化。

## 3. 后训练：主线到 L2

| 方法 | 核心目标 | 需要能说明的限制 |
|---|---|---|
| SFT | 用示范数据学习任务行为 | 数据质量、遗忘、分布覆盖；不自动提供最新知识 |
| LoRA | 低秩增量参数高效微调 | rank/目标层选择、额外适配器管理 |
| QLoRA | 量化底座上训练 LoRA | 省显存但有量化和训练复杂性 |
| DPO | 用偏好对直接优化策略相对参考模型 | 偏好数据偏差、参考模型和过优化 |
| PPO | 基于 reward 的在线策略优化 | critic、采样和训练稳定性成本高 |
| GRPO | 对同 prompt 一组回答做相对优势优化 | 需要可靠 reward、组采样成本与 reward hacking |

项目回答：阈值结构化任务首先选择 Prompt、知识范围、Schema、Java 校验和 Eval，因为核心风险是事实、完整性和写安全；微调只有在高质量数据规模足够、错误稳定且模型外约束无法解决时才值得投入。

对外边界：可以讨论方法，不要把概念学习说成做过大规模 RLHF/GRPO。

## 4. RAG 和微调如何选

| 问题 | 优先手段 |
|---|---|
| 知识频繁更新、需要引用 | RAG/工具 |
| 实时业务状态 | 数据库/API 工具 |
| 固定格式与局部行为 | Structured Output、Prompt、规则；必要时 SFT |
| 稳定领域风格或复杂行为迁移 | SFT/LoRA |
| 不可违反的数值/权限约束 | 程序规则 |
| 多步且路径确定 | Workflow |
| 多步且需根据观察探索 | 有界 Agent |

通常组合使用，而非二选一。

## 5. MySQL

应用岗至少会：

- B+Tree、聚簇/二级索引、联合索引最左前缀；
- explain、回表、覆盖索引和常见索引失效；
- ACID、隔离级别、MVCC、快照读/当前读；
- 行锁、间隙锁、死锁与重试；
- 大事务和超长批量 SQL 的风险；
- 唯一键、幂等表、状态机、outbox；
- 分页、批量写与事务边界。

项目锚点：审批通过不等于写库成功；大批量展开需要明确拆批、原子性、重试和最终状态。

## 6. Redis

至少会：

- 常用结构与适用场景；
- 缓存穿透、击穿、雪崩；
- cache-aside 的一致性窗口；
- 过期、淘汰、热 Key 和大 Key；
- 分布式锁的边界，为什么不能只靠过期时间；
- 持久化与主从/集群的基本取舍；
- 缓存不是权威业务真相时怎样回源。

项目锚点：在线特征快照必须带时间，不能把不同时间读取的向量直接比较。

## 7. MQ 与异步

至少会：

- at-most-once、at-least-once、effectively-once；
- producer confirm、消费 ack、重试、死信；
- 幂等键、去重表和状态条件更新；
- 顺序消息、消息堆积、背压；
- 事务消息或 outbox；
- callback 快速 ACK 与后台执行的状态区分。

一个更可靠的审批执行设计：

```text
审批回调
→ 校验终态与幂等键
→ 事务内写 execution_task + outbox
→ ACK
→ MQ/worker 消费
→ 分批确定性执行
→ 记录每批结果
→ 汇总 SUCCESS / PARTIAL / FAILED
→ 最终回查与通知
```

## 8. 并发、限流与故障

需要能讨论：

- 线程池大小、队列、拒绝策略和上下游背压；
- timeout budget 和取消传播；
- 重试风暴、指数退避和 jitter；
- 限流、熔断、隔离舱与降级；
- 同步/异步、串行/并行的正确性约束；
- P50/P95/P99 与平均延迟；
- 可观测性三件套：logs、metrics、traces。

Agent 场景还要加：模型并发限制、Token 预算、工具调用扇出、长任务 checkpoint 和用户取消。

## 9. Python 面试恢复

优先：

- list/dict/set、排序 key、heapq、deque、bisect；
- 迭代器/生成器、装饰器和 context manager；
- shallow/deep copy、可变默认参数、闭包；
- asyncio 基本模型、Task、gather、timeout 和 semaphore；
- typing、dataclass/Pydantic、异常分类；
- pytest 思维、mock 外部服务；
- NumPy/PyTorch shape 与广播。

Java 深度根据 JD 补：集合、并发、线程池、JVM、Spring 事务、MyBatis、接口和异常设计。主线面试编码优先使用 Python。

## 10. 暂不投入过多时间

除非目标 JD 明确要求：

- CUDA/Triton Kernel 手写和算子极致优化；
- Megatron/DeepSpeed/FSDP 大规模训练实现；
- PPO/GRPO 完整训练工程；
- Kubernetes 控制面、虚拟化、存储和网络深层实现；
- 多 Agent 框架横向测评；
- 每个新模型和框架的 API。

判断标准：它能否提高目标岗位通过率、补当前证据缺口，或形成一个可展示作品。不能就先放入待学，而不是进入本周计划。
