# How I Use Claude Code

- 作者：Simon Willison
- 发布：2026（具体日期未在转载页标出）
- 原文：simonwillison.net（本地摘录自转载页，原文域名未核实，引用时请回查作者站点）
- 本地：摘录，非全文

## 一句话

他的 `CLAUDE.md` 不是设计出来的，是**把"最常打断 Claude 的原因"一条条写下来**攒出来的。

## 核心论点

1. **刻意保持简单**：不用 skills、不用 MCP、不用子 agent、不并行开 worktree。他要一直盯着 agent 在干什么，随时打断。
2. **规则来自打断记录**：
   > "Over time I've turned the most common reasons to interrupt Claude into CLAUDE.md rules, and that does help to reduce the number of times I have to step in."
3. **归属判断**：要记一条经验时先问"这是团队该遵守的规则吗？"——是，就写进 `CLAUDE.md`（进 git、所有人可见）；只关乎个人习惯的，放私有 memory。
4. 他实际的规则分四类：
   - **沟通风格**：少词、直接、不奉承（"stop telling the human they are 'absolutely right'"）。
   - **准确性**：绝不把猜测当事实；没证据就说不知道；先查证再断言。
   - **不多做**：让 review 就只 review，别顺手跑一堆命令。
   - **绝不碰生产**：不写、不读、不"就查一下"；需要生产数据就写好代码交给人跑。

## 可引用的原句

> "Agents do not run anything against production — not writes, not reads, not 'just a quick look'."

> "Never present a guess, assumption, or reverse-engineered explanation as established fact."

## 与这批样本的关系

- 这是"事故驱动"最朴素的一手样本：**规则不是想出来的，是被打断出来的**。与 `分析-活文档在改什么.md` 的结论一致。
- 他的规则几乎全是"负规则"（不要做什么），没有一句项目概览——和 `openai-agents-python` 的写法同源。
- 反向证据：他明确说"不用 skills / 子 agent"，说明"薄地图 + 技能路由"不是唯一解，取决于人是否愿意全程盯着。
