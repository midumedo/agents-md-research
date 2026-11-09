# Effective context engineering for AI agents

- 作者：Anthropic 应用 AI 团队（Prithvi Rajasekaran、Ethan Dixon、Carly Ryan、Jeremy Hadfield 等）
- 发布：2025-09-29
- 原文：https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- 本地：摘录，非全文

## 一句话

上下文是有限资源且有边际递减：目标是"最小的高信号 token 集"，不是最全的说明。

## 核心论点

1. **context rot**：上下文里 token 越多，模型准确回忆的能力越弱，所有模型都有这个特性。
2. **注意力是 n² 的**：n 个 token 产生 n² 个两两关系，越长越摊薄。所以"长度 vs 聚焦"是结构性张力，不是调参问题。
3. **system prompt 要站在"正确的高度"**：太低是硬编码的 if-else，太high是假设了共享上下文的空话。做法是分节（背景 / 指令 / 工具说明 / 输出格式）。
4. **工具要少而不重叠**："如果人类工程师都判断不出该用哪个工具，别指望 agent 能。"
5. **示例要典型，不要堆边缘情况**。
6. **just-in-time 检索优于预取**：让 agent 持有轻量标识符（路径、查询、链接），运行时再取。
7. **混合策略**：Claude Code 把 `CLAUDE.md` 预先放进上下文，同时用 `glob`/`grep` 即时检索——固定的小地图 + 按需的探索。
8. **长任务的三种手段**：压缩（compaction）、结构化笔记（`NOTES.md`）、子 agent（子 agent 烧几万 token，只回 1–2k token 摘要）。

## 可引用的原句

> "Context engineering is the art and science of curating what will go into the limited context window from that constantly evolving universe of possible information."

> "good context engineering means finding the *smallest possible* set of high-signal tokens that maximize the likelihood of some desired outcome."

> "System prompts should be extremely clear and use simple, direct language that presents ideas at the *right altitude* for the agent."

## 与这批样本的关系

- 这是"薄地图 + 按需加载"这条路线最权威的背书：`ax`（108 行）、`adk-python`（30 行）走的就是这条路。
- "最小高信号"直接否定了"概览 + 技术栈 + 目录树"的写法——那些是 agent 自己能检索到的。
- 注意它的立场：这是模型厂商在讲**怎么用上下文**，不是在做对照实验；结论强，证据弱于 `2602.11988` 那种论文。
