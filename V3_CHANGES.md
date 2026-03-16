# Research Refine V3 改动说明

## 为什么还要继续改

V2 相比第一版已经解决了“问题漂移”和“方法不够具体”两个核心问题，但实际使用里还暴露出三个新的短板：

1. 模型会为了拿更高分，把方法越改越复杂，最后变成模块堆叠或贡献点并列。
2. 方法有时还是偏 old school，更像是在拼装模块，而不是站在当下 foundation model 时代去想问题。
3. `research-refine` 结束后，还缺一个能直接承接论文验证工作的详细实验规划步骤。

V3 的目标，就是把 skill 从“方法要更具体”进一步推进到“方法既具体，又简洁、聚焦、现代，还能自然接上实验执行”。

## 1. 新增 elegance-first 约束，专门抑制复杂度膨胀

V3 明确加入了两条新的 paper taste 规则：

- **The smallest adequate mechanism wins**：不是模块越多越好，而是越少的新增机制越好
- **One paper, one dominant contribution**：优先保留一个主贡献，最多一个依附它的 supporting contribution

对应的流程修改包括：

- 新增 `MAX_PRIMARY_CLAIMS = 2`
- 新增 `MAX_NEW_TRAINABLE_COMPONENTS = 2`
- proposal 模板加入 `Contribution Focus`
- proposal 模板加入 `Complexity Budget`
- 明确要求写出 `Tempting additions intentionally not used`
- refinement 阶段新增 `Simplicity Check`

这意味着 V3 不再鼓励“reviewer 说什么就再补一个模块”，而是要求先回答：这个东西是不是必须加？能不能通过更好的接口、loss、distillation 或 inference policy 来解决？

## 2. 把“创新性”从模块堆叠改成 contribution quality

V2 里的 `Technical Novelty` 仍然可能被模型误解成“多做点东西就更 novel”。

V3 把 reviewer 维度改成了 `Contribution Quality`，定义里明确要求同时考虑：

- 是否存在一个真正占主导的 mechanism-level contribution
- 相比最近工作到底新在哪里
- 是否足够简洁优雅
- 是否出现了 parallel contribution sprawl

同时 reviewer 现在必须额外输出：

- `Simplification Opportunities`
- 哪些 reviewer 建议本身会引入不必要复杂度

也就是说，review loop 不只是继续加东西，还必须学会“删东西”和“合并东西”。

## 3. 引入 frontier-aware route selection，减少 old-school 方案

V3 不再默认沿着“搭几个模块”这条路往下走，而是在定方法前要求先比较两种路线：

- **Route A: Elegant minimal route**
- **Route B: Frontier-native route**

这里的 frontier-native route 指的是更自然地使用：

- LLM
- VLM
- Diffusion
- RL
- distillation
- inference-time scaling
- reward modeling

但 V3 也明确强调：**现代技术是 prior，不是 decoration**。如果某个问题根本不需要硬塞 LLM / RL，就不应该为了显得新而强行加入。

reviewer 现在也会额外输出 `Modernization Opportunities`，用来指出：

- 哪些 old-school 部分可以被更自然的 foundation-model-era primitive 替代
- 或者当前方法其实已经足够现代，不需要继续“蹭热点”

## 4. 输出模板改成更聚焦的论文叙事

新的 proposal 模板新增了这些关键部分：

- `Method Thesis`
- `Contribution Focus`
- `Complexity Budget`
- `Modern Primitive Usage`
- `Novelty and Elegance Argument`
- `Experiment Handoff Inputs`

相比 V2 的写法，V3 更像是在逼模型回答以下问题：

- 这篇 paper 的一句话 thesis 到底是什么？
- 哪个是主贡献？
- 哪些东西故意不做？
- 现代技术具体扮演什么角色？
- 为什么不是一个 old-school module stack？
- 后续实验到底需要证明什么？

## 5. reviewer 评分维度更新为 7 个

V3 的 reviewer 维度现在是：

1. `Problem Fidelity`
2. `Method Specificity`
3. `Contribution Quality`
4. `Frontier Leverage`
5. `Feasibility`
6. `Validation Focus`
7. `Venue Readiness`

总分权重也调整成更偏向：

- 是否守住原始问题
- 方法是否足够具体
- 贡献是否既 novel 又 focused
- 是否用了更符合当下时代的技术路径

这比 V2 更不容易把“复杂”误判成“高级”。

## 6. 新增 `experiment-plan` skill，补齐 refine 之后的下一步

V3 新增了一个配套 skill：`experiment-plan`。

它专门解决一个实际问题：方法打磨完之后，接下来论文要怎么设计实验？

新 skill 的职责包括：

- 把最终 proposal 拆成 `claim -> evidence -> run order`
- 设计 main paper 需要的核心实验块
- 设计 novelty isolation、simplicity check、frontier necessity check
- 给出 must-run vs nice-to-have 的优先级
- 估算资源预算、风险和执行节奏
- 生成 `EXPERIMENT_PLAN.md` 和 `EXPERIMENT_TRACKER.md`

这样整个流程就从：

```text
模糊方向 -> research-refine -> 方法定型
```

扩展成：

```text
模糊方向 -> research-refine -> 方法定型 -> experiment-plan -> 实验执行
```

## 7. 组合流程更新

V3 推荐的技能串联顺序变成：

```text
/idea-creator
  -> /research-refine
  -> /experiment-plan
  -> /run-experiment
  -> /auto-review-loop
```

也就是说，`research-refine` 现在负责把方法想清楚，`experiment-plan` 负责把“如何证明这件事”想清楚。

## 8. 补充：新增 `research-refine-pipeline` skill，把两段流程一条龙串起来

为了减少手动切换步骤，本次补充新增了 `research-refine-pipeline`：

- 它会先跑 `research-refine`，再基于最终 proposal 跑 `experiment-plan`
- 如果已有 `FINAL_PROPOSAL.md` 与当前问题一致，也允许直接复用再进入实验规划
- 它会额外产出 `PIPELINE_SUMMARY.md`，把最终 thesis、主贡献、must-prove claims 和最优先实验汇总到一起

这样用户现在既可以分阶段使用，也可以直接走：

```text
模糊方向 -> research-refine-pipeline -> 最终方法 + 实验路线图
```

## 本次修改涉及的文件

- `research-refine/SKILL.md`
- `research-refine/agents/openai.yaml`
- `experiment-plan/SKILL.md`
- `experiment-plan/agents/openai.yaml`
- `research-refine-pipeline/SKILL.md`
- `research-refine-pipeline/agents/openai.yaml`
- `README.md`
- `V3_CHANGES.md`

## 最终希望达到的效果

相较 V2，V3 更希望输出这样的方案：

- 方法更聚焦，不会越改越散
- 创新点更突出，不会一篇 paper 里并列很多贡献
- 方法更现代，不会停留在 old-school 模块拼装
- 实验更像在服务论文主张，而不是服务 checklist
- refine 完之后，可以自然进入执行阶段，而不是停在“写得挺好但不知道怎么验证”
- 用户既能分阶段工作，也能直接使用一条龙 pipeline
