# Research Refine V2 改动说明

## 为什么要改

第一版在实际使用里暴露出两个核心问题：

1. 随着 review / refinement 轮数增加，方案容易只追着上一轮 reviewer 的意见跑，逐渐偏离最初真正想解决的技术问题。
2. 输出内容偏“研究计划书”，方法层面不够深，容易停留在“要做一个模块”的命名层，而不是把模块怎么实现、怎么训练、怎么接入下游讲清楚。

这次更新的目标，就是把 skill 从“泛化的研究方案整理器”改成“守住原始问题、把方法做实”的方法细化工具。

## 跟第一版的核心区别

## 1. 新增 Problem Anchor，防止问题漂移

第一版的问题：每轮 refinement 主要根据上一轮 review 局部修补，容易忘记最初的 bottom-line problem。

现在的做法：

- 在 round 0 先冻结 `Problem Anchor`
- 明确写出 bottom-line problem、must-solve bottleneck、non-goals、constraints、success condition
- 后续每一轮 refinement 都必须原样保留这个 anchor，并先做一次 `Anchor Check`

这样 reviewer 就算提出偏题建议，后续也能识别为 drift，而不是直接把题做偏。

## 2. 每一轮都输出完整 proposal，不再只修局部

第一版虽然也会迭代，但实践中后续轮次更像是在“补 review”，而不是重新检查“问题 -> 方法 -> 验证”是否还是一个完整闭环。

现在每个 `round-N-refinement.md` 都必须包含完整文档：

- Problem Anchor
- Technical Gap
- Method Thesis
- Proposed Method
- Claim-Driven Validation Plan

目的就是避免模型只盯着上一轮 reviewer 的局部意见，导致主线断掉。

## 3. 输出重心从“实验较全”改为“方法优先”

第一版更像标准 research proposal：问题、方法、实验、风险都讲，但方法细节密度还不够高。

现在会强制展开这些方法信息：

- 系统图和模块数据流
- 关键 representation / embedding / control signal 设计
- 模块输入输出、训练信号、loss、训练阶段
- 如何接到 base model / downstream pipeline
- inference 时具体怎么跑
- failure mode、diagnostic、fallback
- novelty 到底落在什么机制上

也就是说，目标不再是“告诉你要做 BNP”，而是“告诉你 BNP 里的表示、loss、训练方式、接入点可能该怎么设计”。

## 4. 实验从“大而全”改为 claim-driven minimal validation

第一版容易把实验部分写得很大，baseline、dataset、ablation 越列越多，反而稀释方法主线。

现在默认只保留 1-3 个核心实验块，每个实验都必须直接服务于某个核心 claim，并明确：

- 这条 claim 是什么
- 最小必要 baseline / ablation 是什么
- 决定性 metric 是什么
- 预期应该看到什么方向的证据

实验现在是“证明方法成立”，而不是“把 proposal 写得像完整论文 appendix”。

## 5. Codex reviewer 的评分标准改成方法导向

第一版 reviewer 更接近常规 proposal review，容易把注意力放到 completeness、实验覆盖度、是否还缺更多 baseline。

现在的评分维度改成：

- Problem Fidelity
- Method Specificity
- Technical Novelty
- Feasibility
- Validation Focus
- Venue Readiness

并且 overall score 明确加权到 `Method Specificity` 和 `Technical Novelty` 上，逼 reviewer 优先指出：

- 方法哪里还不够具体
- 创新到底是不是机制级创新
- 哪些建议会让方案 drift

## 6. 明确允许 pushback，而不是默认照单全收

第一版虽然也允许 pushback，但没有把“review 可能把问题带偏”作为主流程的一部分。

现在 pushback 是正式机制：如果 reviewer 的建议会改变问题本身，或者忽略了已有 grounding，就应该在 refinement 里明确写出来，而不是机械接受。

## 最终想达到的效果

相较第一版，V2 更希望输出这样的文档：

- 原始问题始终不丢
- 方法细节足够具体，能直接进入实现讨论
- 创新点落在真实机制上，而不是命名
- 实验只服务于验证核心 claim
- reviewer 循环是在“磨方法”，不是把方案写得越来越像 checklist

如果后面还要继续迭代，最值得观察的不是“文档变长了没有”，而是：

1. round 越多时，问题锚点是否依然稳定
2. 方法段是否真的比第一版更具体、更可实现
3. reviewer 的批评是否更多集中在机制改进，而不是泛泛地补实验
