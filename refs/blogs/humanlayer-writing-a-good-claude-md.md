# Writing a good CLAUDE.md

- 作者：HumanLayer（Kyle / @0xblacklight）
- 发布：2025-11（原文注明同样适用于 AGENTS.md）
- 原文：https://www.humanlayer.dev/blog/writing-a-good-claude-md
- 本地：摘录，非全文

## 一句话

指令是有预算的：前沿模型稳定跟得住约 150–200 条，而 Claude Code 的系统提示词已经用掉约 50 条——所以文件越短越好，短到 60 行也不奇怪。

## 核心论点

1. **LLM 是无状态的**：会话开始时对代码库一无所知，`CLAUDE.md` 是唯一每次都会进上下文的东西。
2. **它是在给 agent 做 onboarding**：讲清 WHAT（技术栈与目录地图）、WHY（各部分干什么）、HOW（怎么验证、用什么包管理器）。
3. **Claude 真的会忽略它**。harness 会在注入时附一句系统提醒：
   > "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task."
   所以文件里"只对某些任务有用"的内容越多，整份文件被忽略的概率越高。
4. **指令预算**：前沿思考型模型约 150–200 条线性衰减；小模型指数衰减。Claude Code 系统提示词已占约 50 条，接近三分之一。
5. **长度**：社区共识 <300 行，越短越好；HumanLayer 自己的根文件 **< 60 行**。
6. **渐进披露**：把任务专属内容拆到 `agent_docs/*.md`，根文件只留索引与"什么时候去读哪个"。
7. **指向，不要复制**：不放代码片段，放 `file:line`；片段会过期，指针不会。
8. **别让 LLM 当 linter**：风格规则应交给格式化工具（Biome/ESLint），LLM 又贵又慢，还会挤占指令预算。
9. **别用 `/init` 自动生成**：一行坏指令会影响研究、规划、实现三个阶段，错误会级联。

## 可引用的原句

> "Never send an LLM to do a linter's job."

> "Prefer pointers to copies."

> "At HumanLayer, our root CLAUDE.md file is less than sixty lines."

## 与这批样本的关系

- 这份是 `ax`（108 行）、`openai-python`（59 行）的"为什么能这么短"的理论解释：预算有限，所以每条都要挣来。
- "指向，不要复制"正好对应 `dsh` 的 52 条外链、`minimax-code` 的"单源真相表"。
- 它和 `2602.11988` 的结论可以互相印证：文件里重复代码库已有的信息，既费钱又不提升成功率。
