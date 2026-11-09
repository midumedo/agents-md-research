# What AGENTS.md Actually Does to Your Coding Agent

- 作者：Agentic Academy
- 发布：2026（对 arXiv 2602.11988 的深度解读）
- 原文：https://agentic-academy.ai/posts/agents-md-context-files-evaluation/
- 本地：摘录，非全文

## 一句话

这份解读把"AGENTS.md 有用吗"拆成了机制问题：agent 不是没读，是**太听话**——照做带来的步数与成本，超过了它带来的正确率。

## 核心论点

1. **它先纠正了一个流行误解**：失败不是因为 agent 忽略文件。数据显示 agent 精确执行了文件里的指令——文件提到某个工具，该工具的使用频率高 2.5 倍。
2. **真正的失败模式是冗余**：LLM 生成的文件，内容多是 agent 本来能读 README 与代码就知道的东西。研究者把 markdown 文档从环境里移除后，同一批 LLM 生成文件反而提升 2.7%，超过了人写文件。
3. **人写文件的问题是"更彻底的探索"**：文件把人写文件推向更多测试、更多文件遍历，平均多 3.92 步。指令被忠实执行，但路径更贵。
4. **架构结论**：AGENTS.md 是"每轮都在场的环境输入"，适合放**最小、高信号、agent 无法从仓库自行发现**的内容；不适合放通用工程卫生、README 复述、LLM 生成的文档摘要。
5. **规模含义**：20% 的推理成本溢价，在几百个仓库、CI 流水线里是实打实的账单。

## 可引用的原句

> "You're not giving the agent a map — you're handing it a summary of a map it already has access to, and then charging it to read both."

> "The instruction is being followed exactly as intended. The outcome is not what was intended."

## 与这批样本的关系

- 它解释了我样本里的一个反常现象：`adk-python` 30 行、`openai-python` 59 行，反而比 431 行的 `adk-go` 更像"活的规范"——因为短文件只放不可推导的东西。
- 但要注意：这篇是二手解读，个别数字与论文原文有出入（论文：LLM 生成 −0.5% / −2%，人写 +2.4%；不是文中说的 −2~3% 与 +4%）。**引用数字请用论文原文**，见 `../pdf/2602.11988.pdf`，转好的全文在 `../md/2602.11988.md`。
- 它提出的"ambient, always-present input"是个好用的分类词：AGENTS.md 是环境输入，skills 是按需输入，两者不该混。
