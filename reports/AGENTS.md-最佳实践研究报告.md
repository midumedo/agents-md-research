# 从信息约束到可验证规范：基于第一性原理的 AGENTS.md 最佳实践研究

**日期**：2026-11-07
**执行模式**：完整（Workflow A：初始调研 → 大纲规划 → 逐章深度调研与审稿 → 报告框架 → 发布终校）
**研究团队**：深度研究团队（顾全之 · 研究主编 / 谭溯源 · 课题研究员 / 季要纲 · 研究编辑 / 明鉴秋 · 草稿审稿人 / 程文成 · 报告撰写人 / 傅梓铭 · 报告发布员）
**证据规模**：5 章正文，54 条去重来源，5 章均经 1 轮审稿通过

---

## 目录

- 引言
- 1. 编码代理冷启动信息约束与持久化指令文件的目标失败模式
- 2. 上下文注意力预算、Context Rot 与指令遵循衰减的因果机制
- 3. 模型团队一手证据的 t0–t3 强度分层与等价指令文件差异
- 4. 描述性统计与对照实验的规范性边界及指令膨胀现状缺口
- 5. 最小高信号指令集规范、确定性执行分工与渐进式验证闭环
- 结论
- 参考文献
- 待完善事项

---

## 引言

AGENTS.md 已从项目约定变为跨厂商标准：OpenAI 将其捐赠给 Agentic AI Foundation，官方站点称 60,000+ 项目采用 ([AGENTS.md](https://agents.md/); [OpenAI](https://openai.com/index/agentic-ai-foundation/))。但「被广泛写下」不等于「被有效管理」——用户的关切正在于此：现有仓库大多并未做好 AGENTS.md 的管理。于是须先回答一个方法论问题：从现存文件统计出的高频写法，能否直接当作应然规范？

答案是否定的。持久化指令文件面对的是有明确边界的冷启动约束：代理每次会话都以全新上下文窗口开始，跨会话知识只能靠注入延续，其价值取决于是否提供模型无法从代码推断、且可预测地被发现的信息 ([Claude Code Memory](https://code.claude.com/docs/en/memory))——这应由约束反推，而非由频次归纳。而横截面研究普遍内嵌「现存写法具代表性」这一未检验前提：22,854 仓库普查得到 26.3% 采纳率、74.0% 指令不附理由 ([arXiv:2610.31715](https://arxiv.org/abs/2610.31715))，MSR 2026 的 466 项目研究更把效果测量列为 future work ([arXiv:2510.21413](https://arxiv.org/abs/2510.21413))；以类别频次推导「循证模板」者，预设前提干脆是「普遍即应写」 ([arXiv:2611.02383](https://arxiv.org/abs/2611.02383))。这类论文尤须甄别预设前提：它们只答「现在写什么」，不答「写了是否有用」。

本报告因此不统计表象，而从两条不随模型代际归零的约束——冷启动信息缺口与注意力预算稀缺——出发，依次推演目标失败模式、Context Rot 因果机制、厂商一手证据的 t0–t3 分层、描述性统计的规范性边界，直到最小高信号指令集与验证闭环。三个判断先行预告：官方仓库文件只说明「实际怎么写」，不构成效果证据 ([Best Practices](https://code.claude.com/docs/en/best-practices))；过期与冲突指令会制造可测的遵循失效 ([arXiv:2610.30571](https://arxiv.org/abs/2610.30571))；额外上下文并非无条件增益 ([arXiv:2602.11988](https://arxiv.org/abs/2602.11988))。

---

## 1. 编码代理冷启动信息约束与持久化指令文件的目标失败模式

### 一、论点：冷启动是信息约束，不是文档习惯

大语言模型推理不携带跨会话状态。Anthropic 官方文档的一手表述是「Each Claude Code session begins with a fresh context window」，并把跨会话知识明确交给两类外部机制承载——人写的 `CLAUDE.md`（或 `AGENTS.md`）与 auto memory，二者「Both are loaded at the start of every conversation. Claude treats them as context, not enforced configuration」（[Claude Code Memory 文档](https://code.claude.com/docs/en/memory)）。由此可作第一性原理推导：一次会话开始时，模型可见的世界严格等于本次被注入的 token 序列；任何未被放入上下文的项目事实，对本次推理在信息论上等价于不存在。Anthropic 将 context 定义为「finite resource with diminishing marginal returns」，主张目标是「the smallest possible set of high-signal tokens」，并把 `CLAUDE.md` 描述为「naively dropped into context up front」、与 just-in-time 检索互补的混合策略（[Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)）。持久化指令文件因此不是「给 agent 读的说明书」，而是**冷启动瞬间被确定性注入的最小信息基线**（此处「确定性」仅指注入时机与字节顺序确定，不指模型必然遵循；执行效力问题见第 2 章）；其随长度衰减的因果机制留第 2 章。

三家官方对注入时机的描述彼此印证：Codex「builds an instruction chain when it starts (once per run)」并按 32 KiB 上限截断（[Codex AGENTS.md 指南](https://developers.openai.com/codex/guides/agents-md)）；Gemini CLI 将各层文件「concatenates the contents of all found files, and sends them to the model with every prompt」（[GEMINI.md 文档](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html)）；GitHub Copilot 写明「Instructions are automatically added to requests that you submit to Copilot」（[GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)）。

这也解释了为何必须与 README.md 刻意分离。官方规范写明「README.md files are for humans」，分离目的是「Give agents a clear, predictable place for instructions」（[AGENTS.md 官网](https://agents.md/)）：README 供人按需检索，AGENTS.md 供尚不具备项目知识的 agent 在检索之前被注入。OpenAI 的捐赠动机原文更直接——「Codex needed a predictable way to find project-specific instructions—such as coding conventions, build steps, and testing requirements」（[OpenAI, 2025-12-09](https://openai.com/index/agentic-ai-foundation/)）。「predictable」指向的是**发现路径的确定性**，而非信息量。

### 二、论据：目标失败模式是「不可推断信息」与「可预测路径」的双重缺口

**其一，模型无法从代码推断什么。** Anthropic 现行最佳实践给出判据：「Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session」，并列出「从未进入仓库」的信息——团队会议上的决策、只存在某人脑中的部署流程、集成测试所需的环境变量；以及「虽在仓库里、但每次会话都要重新挖掘」的信息（[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)）。后者正是冷启动的重复发现成本。

**其二，由对照实验反推该承载的信息类型。** ETH Zurich 的研究是本章最关键的规范性支点：实验表明提供 context file「does not generally improve task success rates, while increasing inference cost by over 20% on average」，且「instructions in the context files are well followed by coding agents, repository overviews, although popular and recommended by model providers, are not helpful」，作者结论为「context files are useful for specifying non-standard coding practices」（[arXiv:2602.11988, v3 2026-09](https://arxiv.org/abs/2602.11988)）。**预设前提与边界须记录**：该研究在两种互补设置（既有 SWE-bench 任务配 LLM 生成文件，也有开发者已提交文件的仓库 issue 集合）上测的是任务成功率与推理成本，作者自述「any attempts to improve performance should be rigorously evaluated before deployment」；故结论适用于「可被模型从代码推断的冗余内容（尤其仓库概览）无增益」，不可外推为「所有指令文件无效」。其价值恰在于反推：既然指令被遵循而概览无用，瓶颈就不在「模型不听话」，而在「写进去的信息是否携带增量信息」。Anthropic 独立印证同一方向——「the single most useful line in your CLAUDE.md is the command that runs it」（能返回 pass/fail 的验证命令），且现行版已把「Describe your architecture and file layout in CLAUDE.md」列入「What you can stop doing」（[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)）；鉴于厂商建议随模型代际自我反转，以官方现行文档为准，早期转述仅作历史对比。

**其三，描述性统计只能描述现状。** MSR 2026 的 466 项目研究发现「there is no established content structure yet」，呈现方式在描述性／规范性／禁止性／解释性／条件性间高度分化（[arXiv:2510.21413, v4 2026-02](https://arxiv.org/abs/2510.21413)）。**前提与边界**：这是横截面采用度与内容形态调查，无对照、无效果测量，作者把「which structural or presentational modifications can positively affect the quality」明列为后续方向；故它只能说明「现状尚无共识结构」，不能推导「该写什么」的规范结论，系统甄别留第 4 章。

### 三、分析：等价性、层级位置与现状缺口

**等价性。** AGENTS.md / CLAUDE.md / GEMINI.md / .cursorrules / .github/copilot-instructions.md 属同一类「Agent 上下文指令文件」：仓库级、版本控制、自然语言、会话开始时注入。最硬的一手并列证据来自 GitHub 官方文档，它在同一节把 `.github/copilot-instructions.md`（repository-wide instructions）、`AGENTS.md`（就近优先）与「a single `CLAUDE.md` or `GEMINI.md` file stored in the root」并列为 Copilot 的仓库指令来源，onboarding 目标写作「how a cloud agent seeing it for the first time can work most efficiently」（[GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)）。可互换性亦有多方佐证：Gemini CLI 的 `context.fileName` 可设为 `["AGENTS.md", "CONTEXT.md", "GEMINI.md"]` 并支持 `@file.md` 导入（[GEMINI.md 文档](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html)）；Claude Code 官方称「Claude can also read a repository's AGENTS.md files in place of CLAUDE.md」（[Memory 文档](https://code.claude.com/docs/en/memory)）；`copilot-instructions.md` 于 2025-08 GA（[GitHub Changelog](https://github.blog/changelog/2025-08-06-copilot-code-review-copilot-instruction-md-support-is-now-generally-available/)）。差异仅在默认文件名、分层与就近覆盖规则、导入语法、截断阈值（如 Codex 32 KiB）——机制差异与 t0–t3 强度分层留第 3 章。

**特权层级铺垫。** OpenAI 的 Instruction Hierarchy 提出显式优先级并训练模型「selectively ignore lower-privileged instructions」，其**预设前提**是「LLMs often consider system prompts … to be the same priority as text from untrusted users and third parties」，实验在 GPT-3.5 上验证鲁棒性提升且「minimal degradations on standard capabilities」（[arXiv:2404.13208, 2024-04](https://arxiv.org/abs/2404.13208)）；**边界**：这是提示注入防御的受控实验，非仓库指令文件实测，本章仅借其分层框架。仓库级指令的实际位置由官方文档给定：「CLAUDE.md content is delivered as a user message after the system prompt, not as part of the system prompt itself. Claude reads it and tries to follow it, but there's no guarantee of strict compliance」，并被定性为「context, not enforced configuration」（[Memory 文档](https://code.claude.com/docs/en/memory)）；[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices) 同样把 `CLAUDE.md` 归为 non-deterministic，与 harness 强制执行的 deterministic 机制相对。故它处于低特权、建议性层，冲突消解时不能压过系统／开发者级指令，也不承诺被严格遵循；零例外约束的分工留第 5 章。

**辩证审视与现状缺口。** 官网「60k+ open-source projects」、OpenAI 主仓 88 份 AGENTS.md（[官网](https://agents.md/)）与厂商自述采纳数字（[OpenAI](https://openai.com/index/agentic-ai-foundation/)）只是采纳率与规模，不构成有效性证据。恰恰相反，官网「How to use」仍建议包含「Project overview」，而对照实验已显示 repository overviews「not helpful」、MSR 2026 又显示真实仓库大量堆砌概览与架构描述——规范模板、主流写法与实验证据之间存在明确**现状缺口**。另须注明一条证据边界：模型团队官方仓库内的 AGENTS.md（[deepseek-harness 187 行](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md)、[qwen-code 368 行](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)、[kimi-code 133 行](https://github.com/MoonshotAI/kimi-code/blob/main/AGENTS.md)）只回答「工程团队实际怎么写」，不回答「是否有效、为何有效」，不作效果证据。

**反方与边界。** 最强版本的「既然模型能读代码，为何还需要文件」来自官方自身：「Claude starts every task by reading your code … Claude already knows most of what you might be tempted to write down for it」（[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)）。回应是：能读代码 ≠ 能推断从未写进仓库的约定、决策与验证路径，且「每次会话重新挖掘」本身是可消除的冷启动成本；[arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 的实证把质疑限定在概览类内容，并未否定非标准实践。另有观点主张指令将随模型进步而消亡：Thoughtworks 提出「reevaluate instructions against each new model and delete those it no longer needs」（[Agent instruction bloat, 2026](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat)），并援引 Anthropic 删除 80%+ 系统提示词；但该援引指向 [claude.dev 厂商博客](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/)，属**厂商自述且方法未公开**（见『待完善事项』第 1 条），同页将 [arXiv:2601.20404](https://arxiv.org/abs/2601.20404) 转述为「手写优于 LLM 生成」亦属引用错位（见『待完善事项』第 2 条），均不得采信为事实依据。Anthropic 官方表述仅为趋势判断——「As model capabilities improve, agentic design will trend towards letting intelligent models act intelligently, with progressively less human curation」（[Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)），并非「文件消亡」结论。

### 四、小结

本章把目标失败模式界定为：编码代理冷启动时，**模型无法从代码／目录结构推断的项目专属信息**与**可预测地找到这些信息的发现路径**同时缺失。持久化指令文件的正确目标函数是补齐前者的最小集合，而非尽可能完整地描述项目；它是每次会话被注入、位于低特权建议性层的上下文，不是强制配置。由此，「6 万+ 项目采用」只是现状，而规范模板偏爱的概览内容恰是实验显示为无增益的部分——这一现状缺口构成后续各章的问题起点：冗余信息为何反致性能衰减（第 2 章）、厂商证据强度与机制差异如何分层（第 3 章）、描述性与实验性结论的规范边界何在（第 4 章）、最小高信号指令集与验证闭环如何建立（第 5 章）。文件形态是否会随模型进步演化，本章仅记录为待检验观点，留第 5 章论证。

---

## 2. 上下文注意力预算、Context Rot 与指令遵循衰减的因果机制

**（论点）** 第 1 章确立 AGENTS.md 类文件是冷启动时确定注入的低特权上下文；本章追问「注入多少」的代价。中心论点：指令遵循衰减不是模型「不够努力」的工程瑕疵，而是自注意力架构、训练数据分布与概率采样叠加的**架构性**结果；上下文窗口扩容只改变管理手段，未取消约束。证据按强度分层使用：t0 模型团队一手工程论述与官方文档为机制骨架，受控实验为因果支点，学术论文交叉验证，厂商自述评测仅作旁证。

### 2.1 架构性根因：token 变多为何稀释有效信号

Transformer 使每个 token 可 attend 全体 token（[Vaswani et al., 2017，奠基架构论文，年份较早仅作架构依据](https://arxiv.org/abs/1706.03762)）。Anthropic 由此给出量化关系：n 个 token 产生 n² 对成对关系，上下文变长时模型捕捉这些关系的能力被「摊薄（stretched thin）」（[Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)）。机制上 softmax 注意力是竞争性归一化的：总权重质量守恒，候选 token 越多，任一「指令—证据」配对分到的注意力份额越小，有效信号被无关 token 稀释、信噪比下降——不是「记不住」，而是「分辨不出」。叠加训练分布偏差（短序列通常比长序列常见，模型对 context-wide dependencies 经验更少、专门参数更少；位置编码插值虽可外推，但以位置理解降级为代价，同前），形成「性能梯度而非硬断崖」：长上下文仍可用，但信息检索精度与长程推理精度下降。**为何随模型变大也不消失**：只要仍是 softmax 自注意力、仍从有限长度分布训练，n² 竞争与分布外推就内生存在；扩容窗口只把「耗尽」这一硬边界后移，不改变摊薄机制。该文明确预判「在可预见的未来，任何尺寸的上下文窗口都会受 context pollution 与信息相关性问题影响」（同前）。须区分事实与主张：以上是厂商基于架构推导、外部 benchmark 与内部观察作出的**解释性／规范性论证**，该文自述为工程立场文章（engineering/position piece）、非受控实验，不构成实测因果结论。

### 2.2 受控实验支点：因果解释力从何而来

[Chroma, Context Rot（2025-07-14）](https://www.trychroma.com/research/context-rot) 测 18 款模型（含 GPT-4.1、Claude 4、Gemini 2.5、Qwen3），自述「保持任务复杂度不变、仅改变输入长度，以直接测量输入长度本身的效应」，任务为 NIAH 扩展、LongMemEval 与重复词复现；结论是「所有实验中模型性能随输入长度增长持续下降」，语义相近但不答题的 distractor、较低的 needle–question 相似度、haystack 结构均放大退化。[Lost in the Middle（arXiv 2023 / TACL 2024）](https://arxiv.org/abs/2307.03172) 则在多文档问答与键值检索中只操纵**相关信息的位置**，得到 U 型曲线：首、尾表现最好，中部显著退化，「即便对显式长上下文模型亦然」。

「仅变一个变量」之所以赋予因果解释力，在于问题、答案、任务规则与模型固定后，性能差异无法归因于任务变难、数据不同或模型差异，只能归因于被操纵的长度或位置——这是从「相关」跃迁到「因果」的条件，也正是与第 4 章横截面仓库统计（诸多变量同时变动、只能描述现状）的分野（此处仅立边界，系统甄别留第 4 章）。

**边界须四层递进**：① 二者都是**通用长上下文受控实验，并非 AGENTS.md 专属实验**——Chroma 语料为 Paul Graham 随笔、arXiv 论文、聊天历史与重复词串，明确未测仓库指令文件；Lost in the Middle 为合成检索任务。② Chroma 并非全篇单变量：LongMemEval 的 full input 同时引入检索步骤，重复词任务的输出长度随输入增长，distractor 数量为子因素；needle–haystack 相似度仅测两个主题，作者自述不足以推广。③ 最关键：该报告明言「我们不解释这一性能退化背后的机制」——它确立的是「长度→退化」的**因果效应**，机制由 2.1 的架构论证补足，二者不可互证。④ Lost in the Middle 用的是 2023 年模型与两类检索任务，U 型曲线属任务相关现象。故两项实验只支持「注入 token 越多、位置越不利，信号越易被稀释」这一方向性机制，**不得外推为「AGENTS.md 一定有害」**。

### 2.3 指令层衰减、冲突消解与 advisory 属性

衰减的第二条路径在语义层。OpenAI 的 [GPT-5 prompting guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide) 指出，含矛盾或含糊指令的 prompt「消耗推理 token 去搜寻调和矛盾的方式，而非随机挑一条指令执行」，并建议把 prompt 视为多人持续更新的 living documents、定期审查去歧义——须标注：这是厂商工程指南、属观察性主张（「we've seen multiple early users…」），页面自带 archived 提示，非对照实验。机制上，文件越长、参与维护者越多，内部冲突与含糊的先验概率越高；冲突不是被忽略，而是**额外消耗推理预算去调和**，与被摊薄的注意力叠加，构成随长度衰减的可测代价。

冲突如何被消解，[The Instruction Hierarchy（arXiv:2404.13208, 2024）](https://arxiv.org/abs/2404.13208) 提供解释框架：system > developer > user > 第三方输出，通过数据生成训练模型「选择性忽略低特权指令」；其**预设前提是提示注入／越狱防御**、实验在 GPT-3.5 完成、未在仓库指令文件上实测，且层级是训练所得行为而非架构天然属性。可谨慎对接的实现事实：Claude Code 官方记忆文档明确 CLAUDE.md「作为 system prompt 之后的一条 user message 注入，而非 system prompt 本身」，且「不保证严格遵循，对含糊或冲突指令尤其如此」（[Claude Code Memory](https://code.claude.com/docs/en/memory)），这解释了指令文件在冲突中的低特权位；但二者来自不同模型、不同研究目的，**层级概念只提供解释框架，不构成相互验证**。

更根本的一点：LLM 是概率采样器而非规则引擎，指令只是抬高某行为条件概率的 token。故「遵循」天然是概率性的，模型有权判定某条 guideline 不适用——[Anthropic 现行版 Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices) 把控制二分为 “Non-deterministic (Claude decides)” / “Deterministic (Claude Code enforces)”，并直言 “CLAUDE.md is for guidelines, and Claude is allowed to decide a guideline doesn't apply”；记忆文档同样定性为 “context, not enforced configuration”“not a hard enforcement layer”。由此可推出一条不依赖经验的结论：对「零例外、每次必须发生」的要求，自然语言加权（IMPORTANT、全大写）只能提高被注意到的概率，无法提高执行效力——现行版即指出「继续加 IMPORTANT 大概修不好：要么文件已长到那一行被淹没，要么它属于不能有例外的规则、应放进 hook」。获得执行效力确定性的唯一机制，是把判定权移出模型、交给 harness 在固定生命周期事件上执行（hooks / deny rules / managed settings / CI）。术语澄清：此处「确定性执行」指**执行效力确定**，与第 1 章「注入时机确定」非同一维度，不可混淆。至于写什么、不写什么、多长、谁维护及验证闭环，属第 5 章；本章只确立「为何必须分流」。

### 2.4 机制推论与反方辨析

由上述三条因果链可推出注意力预算的分流策略，均为 [Anthropic 上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) 的**规范性主张而非实验结论**：寻找「最小可能的高信号 token 集」（且 minimal ≠ short）；just-in-time / progressive disclosure 优于全量预加载，代价是运行时探索更慢、依赖良好工具与启发式；compaction 与 sub-agent 把高 token 量、低信号的探索隔离到子上下文、只回传约 1,000–2,000 token 摘要，保护主上下文预算。

一条实证旁证来自 [Evaluating AGENTS.md（arXiv:2602.11988, ETH Zurich, v3 2026-09）](https://arxiv.org/abs/2602.11988)：这项带对照性质的评估报告指出，提供 context files「并不普遍提升任务成功率，同时平均增加超过 20% 的推理成本」，且「指令被较好遵循」「仓库概览无帮助」，价值收窄到「指定非标准编码实践」，并自限「任何性能改进尝试都应在部署前严格评估」。**必须注明边界**：该研究测的是任务成功率与推理成本，是「文件变长带来可测代价」的旁证，**并未测量 context rot 或注意力机制**；摘要页亦未披露仓库／任务／模型数量与基线构造细节。故它支持「注入有代价」，不构成对 2.1 机制的检验，更不能读作「AGENTS.md 有害」。

**反方质疑**：上下文窗口已达 1M token，且 Anthropic 现行版把 “The context window is your main constraint, manage it aggressively” 列入 “What you can stop doing”（此即厂商建议随模型代际自我反转之一例），是否说明注意力预算约束已被工程解决？

辨析四层。① **是管理手段变了，不是约束消失了**。读该条目原文理由可见：被否定的是「把窗口当容量、怕耗尽而手工激进管理」，理由是 1M 窗口与自动 compaction 使「耗尽已罕见」；但同一句紧接着保留 “`/clear` between tasks 仍值得做，因为聚焦的上下文带来更好答案且更省钱”——「聚焦得更好答案」恰恰**承认稀释机制仍在**，只是应对从手工压缩换为清空／聚焦 + 自动 compaction（[现行版规范](https://code.claude.com/docs/en/best-practices)）；同一文档仍要求「只写 Claude 读仓库找不到的信息」，记忆文档仍保留「超过 200 行的文件消耗更多上下文并可能降低遵循度」与 /doctor 裁剪建议。② 厂商最新一手文档把两者直接并置：[Anthropic 官方 Context windows 文档](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) 在说明多款模型 1M 为默认窗口、「无需 beta header」的**同一页**写明「更多上下文并不自动更好」「随 token 数增长，accuracy 与 recall 退化，即所谓 context rot」「筛选上下文内容与窗口有多大同样重要」。窗口扩容与 context rot 在厂商自己最新的一手文档中并存，是「约束未消失」证据强度最高的直接材料。③ 独立受控证据显示**标称窗口 ≠ 有效上下文**：[NoLiMa（ICML 2025）](https://arxiv.org/abs/2502.05167) 测 13 款宣称 ≥128K 的模型，在 needle 与 question 词汇重叠最小、无字面匹配捷径的条件下，32K 时 11 款跌破短基线 50%，GPT-4o 由 99.3% 降至 69.7%，作者归因于「缺少字面匹配时注意力机制在长上下文中的困难」；[RULER（COLM 2024，年份较早）](https://arxiv.org/abs/2404.06654) 以可配置序列长度与任务复杂度测 17 款模型，vanilla NIAH 近乎满分却随长度大幅下降，宣称 ≥32K 者仅一半能在 32K 维持满意性能。二者均为合成受控 benchmark、非编码代理任务，仅用于否定「容量等同可用性」，不外推到 AGENTS.md 效果。④ [claude.dev 博文（2026-07-24）](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/) 声称删去 80%+ system prompt 后「在我们的编码评测中无可测量损失」（Claude Opus 5 / Fable 5）；经访问核验，页面未披露评测方法、数据集、样本量、控制条件与结果数据，独立读者无法复现，属**厂商自述且方法未公开（见『待完善事项』第 1 条）**，仅反映厂商自身工程取向，不能作为「约束消失」的独立证据，措辞亦限定于 “our coding evaluations”；且删低信号 token 恰是最小高信号集原则的应用，与本章机制不冲突。

### 小结

n² 摊薄叠加训练分布偏差导致信号稀释、冲突调和消耗额外推理预算、概率采样使遵循天然非确定——三条因果链共同说明指令遵循衰减是架构性的，不随窗口或模型规模归零；扩容与自动 compaction 只把管理从「容量」层移到「信号」层。[arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 的 +20% 推理成本与 Chroma / Lost in the Middle 的方向性退化，从不同维度说明这段注入不是免费的。因此第 5 章的最小高信号规范不是经验偏好，而是对有限注意力预算的必然配置：文件形态会演化（skills、auto-memory、渐进披露），但「每枚常驻 token 都消耗预算、只有高信号 token 值得常驻」这一约束不消亡。

### 数据摘要

| 指标 | 数据 | 来源 |
|------|------|------|
| 自注意力成对关系 | n 个 token → n² 对；能力随长度「stretched thin」 | [Anthropic 上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| Chroma 受控实验 | 18 款模型，固定复杂度仅变输入长度，性能随长度一致下降 | [Context Rot](https://www.trychroma.com/research/context-rot) |
| Lost in the Middle | 只操纵相关信息位置得 U 型曲线（arXiv 2023 / TACL 2024） | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) |
| AGENTS.md 对照评估 | 成功率不普遍提升，推理成本平均 +20% 以上；未测 context rot | [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) |
| 1M 窗口与 context rot 并存 | 1M 为默认、无需 beta header；同页称 accuracy / recall 随 token 增长退化 | [Anthropic 官方 Context windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) |
| NoLiMa | 13 款 ≥128K 模型；32K 时 11 款 < 短基线 50%；GPT-4o 99.3%→69.7% | [arXiv:2502.05167](https://arxiv.org/abs/2502.05167) |
| RULER | 17 款模型；宣称 ≥32K 者仅一半在 32K 维持满意性能（COLM 2024） | [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) |
| sub-agent 回传量 | 子代理可耗数万 token，仅回传约 1,000–2,000 token 摘要 | [Anthropic 上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| 删 80%+ 系统提示 | 「no measurable loss on our coding evaluations」，方法／样本／控制条件未披露 | [claude.dev, 2026-07-24](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/) |

---

## 3. 模型团队一手证据的 t0–t3 强度分层与等价指令文件差异

### 3.1 论点：按「距模型行为的距离」分层，而非按厂商名录

分层判据不是厂商规模，而是**谁掌握「模型读到这段文本后概率上会怎样」的一手知识**。自研模型团队同时握有训练分布、系统提示、内部评测与 harness 实现，其方法论文档能回答「机制上为何」；第三方仓库里的 `AGENTS.md` 只是下游使用样本，只能回答「别人写了什么」；产品页能力宣称连「写了什么」都不涉及。据此本章把一手证据分四档：**规范性方法论与机制论述 > 官方 harness 发现/加载规范 > 官方仓库自身指令文件（工程实践）> 产品页宣称与检索空白**。

值得注意的是，格式创立者的公开动机本身落在「可预测发现路径」而非效果验证：OpenAI 在捐赠声明中写明「AGENTS.md began as a practical solution… Codex needed a predictable way to find project-specific instructions」，并于 2025-12-09 联合 Anthropic、Block 在 Linux 基金会下创立 AAIF 完成标准收口（[OpenAI 官方](https://openai.com/index/agentic-ai-foundation/)、[AAIF](https://aaif.io/)）。这决定了「规范文本」与「效果证据」从起点就是两类东西。

### 3.2 论据一：t0–t3 一手证据分层（补检索后）

| 层级 | 厂商 / 官方渠道 | 一手证据类型 | 具体来源 | 证据强度（指一手知识分量，不代表可支撑规范性结论） | 能否支撑规范性结论 |
|---|---|---|---|---|---|
| **t0** | **OpenAI**：规范＋标准治理＋模型行为观察；**Anthropic**：规范性方法论＋机制论述＋加载规范；**Google**：官方分层规范；**DeepSeek**：官方 harness 仓库实践 | 官方方法论 blog｜官方 harness 规范｜模型卡/系统提示｜仓库 AGENTS.md | [Codex 指南](https://developers.openai.com/codex/guides/agents-md)、[AAIF 捐赠](https://openai.com/index/agentic-ai-foundation/)、[GPT-5 prompting guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)（archived、观察性）；[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)、[Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)、[Memory 文档](https://code.claude.com/docs/en/memory)；[GEMINI.md 文档](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html)；[deepseek-harness/AGENTS.md](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md)（187 行） | **最强**：四类知识齐备 | **部分**——方法论可提出规范，但需对照实验校验（见 3.5） |
| **t1** | **Qwen / Kimi**：官方 harness AGENTS.md＋发现机制线索；**xAI Grok / MiniMax**：本轮新检出官方 coding agent harness；**智谱 GLM**：官方开发者文档；**字节 Seed**：官方产品页；**Meta Muse**：未检出官方渠道 | 仓库 AGENTS.md｜官方 issue/提交说明｜开发者文档｜仅产品页宣称 | [qwen-code/AGENTS.md](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)（368 行）、[kimi-code/AGENTS.md](https://github.com/MoonshotAI/kimi-code/blob/main/AGENTS.md)（133 行）、[kimi-cli #2401](https://github.com/MoonshotAI/kimi-cli/issues/2401)、[grok-build](https://github.com/xai-org/grok-build)、[grok-prompts](https://github.com/xai-org/grok-prompts)、[minimax-code](https://github.com/MiniMax-AI/minimax-code)、[智谱接入工具文档](https://docs.bigmodel.cn/cn/coding-plan/tool/others)、[Seed2.0 官方页](https://seed.bytedance.com/zh/seed2) | 中：以仓库实践与工具规范为主，方法论稀缺 | **否**（仓库文件）／**部分**（发现机制）／**否**（产品宣称） |
| **t2** | **小米 MiMo / 阶跃 Step**：官方 harness 仓库＋（小米）官方域名技术文章；**蚂蚁 Ling**：官方 inclusionAI 组织内 agent 平台仓库 | 仓库 AGENTS.md/CLAUDE.md｜去重与维护提交记录｜机制论述文章 | [MiMo-Code](https://github.com/XiaomiMiMo/MiMo-Code/)、[MiMo 长程任务文章](https://mimo.xiaomi.com/en/blog/mimo-code-long-horizon)、[Step-Code](https://github.com/stepfun-ai/Step-Code)、[inclusionAI/Avernet](https://github.com/inclusionAI/Avernet) | 中偏强：一手工程实践＋一家机制论述 | **否**——仍属「实际怎么写」 |
| **t3** | **美团 LongCat / 百度 ERNIE**：官方组织与仓库可核验；**小红书 dots**：组织路径未核验成功 | 官方开源仓库（模型/harness），未见指令文件论述 | [meituan-longcat/LongCat-DeepResearch](https://github.com/meituan-longcat/LongCat-DeepResearch)、[PaddlePaddle/ERNIE](https://github.com/PaddlePaddle/ERNIE) | 弱：仅能确认公开渠道覆盖范围 | **否** |

**t1 逐一核对结论。** Qwen、Kimi 除既有 AGENTS.md 外，本轮另检出发现机制一手线索：`MoonshotAI/kimi-cli` 的[官方仓库 issue #2401](https://github.com/MoonshotAI/kimi-cli/issues/2401) 明载「Currently, Kimi CLI only discovers `AGENTS.md` files」，并请求同时合并 `CLAUDE.md`，属**工具行为声明**而非效果主张。xAI 官方组织 `xai-org` 下有 [grok-build](https://github.com/xai-org/grok-build)（官方自述「coding agent harness and TUI」）与 [grok-prompts](https://github.com/xai-org/grok-prompts)（公开 Grok 系统提示，含 `grok-code-fast-1` 的 injected system prompt prefix）；grok-build 提交历史出现「Document git prefix, nested AGENTS.md, two-pass…」「Read project-instruction files whole in read_file」「Keep the newest Agent line under 24k with a [truncated] marker」，但**根目录未显示 AGENTS.md 文件，故只记为文档线索，不记为仓库实践**。MiniMax 官方 [minimax-code](https://github.com/MiniMax-AI/minimax-code) 根目录列有 AGENTS.md，README 写明「Use `mcode init .` to generate or update project guidance in `AGENTS.md`. Describe the expected result, allowed changes, and how to verify the task.」智谱官方 [GLM Coding Plan 接入工具文档](https://docs.bigmodel.cn/cn/coding-plan/tool/others) 仅列出适用工具、端点协议与 Context Window Size，检索后**未发现关于 agent 指令文件的官方论述**。字节 Seed 官方 [Seed2.0 页](https://seed.bytedance.com/zh/seed2) 只宣称「重点强化了指令遵循能力」并给出 Instruction Following 评测，属产品能力宣称，**不是工程方法论文档**。Meta Muse：检索了 ai.meta.com、github.com、about.fb.com 域及「Muse Code 官方系统提示 / agent instruction file」中英文关键词，返回结果为 CSDN、钛媒体、百家号等二手稿，未找到官方 GitHub 组织、官方工程 blog 或模型卡中的相关论述；此处仅记录检索空白（由「未找到」推不出「没有」），不以二手稿补位。

**t2 / t3 逐一核对结论（本轮主要修正）。** 阶跃星辰官方 [Step-Code](https://github.com/stepfun-ai/Step-Code) 根目录同时列出 `AGENTS.md` 与 `CLAUDE.md`，README 称「Run `/init` to generate an `AGENTS.md` project guide; an existing `CLAUDE.md` is used as-is.」小米官方 [MiMo-Code](https://github.com/XiaomiMiMo/MiMo-Code/) 根目录列有 `AGENTS.md`，提交历史含「remove the `CLAUDE.md → AGENTS.md` symlink」。蚂蚁官方 inclusionAI 组织下 [Avernet](https://github.com/inclusionAI/Avernet) 根目录列有 `AGENTS.md`、`CLAUDE.md`、`CONTEXT.md`、`CONTEXT-MAP.md`，PR 模板引用「Pull Request Conventions in AGENTS.md」。t3：美团官方 [LongCat-DeepResearch](https://github.com/meituan-longcat/LongCat-DeepResearch)（开源 deep research harness）与百度官方 [PaddlePaddle/ERNIE](https://github.com/PaddlePaddle/ERNIE) 的可见根目录与 README 均**未发现 AGENTS.md / CLAUDE.md 或指令文件论述**；小红书方面，检索了 `github.com/rednote-hilab` 与 `github.com/RedNote-HiLab` 两个组织路径（均返回 GitHub 404）及中文「小红书 hi lab / dots.llm1 / dots.vlm1」关键词（结果均为二手转述），**未能核验官方组织页，未发现关于 agent 指令文件的官方论述**。据此，以上均为渠道覆盖结论，不反推「没有实践」。

### 3.3 论据二：t0 提供的六类不同性质的一手知识

不作标题罗列，而按知识类型区分：**（a）发现/加载规范**——Codex 三层拼接、`AGENTS.override.md`、`project_doc_fallback_filenames` 与 32 KiB `project_doc_max_bytes`（[Codex 指南](https://developers.openai.com/codex/guides/agents-md)）；**（b）机制论述**——Anthropic 关于上下文工程与指令遵循的因果说明（[Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)）；**（c）模型行为观察**——GPT-5 prompting guide 的观察性主张，已 archived，证据等级低于规范性文档（[GPT-5 guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)）；**（d）规范性方法论**——Claude Code Best Practices 的「CLAUDE.md is advisory」「Non-deterministic / Deterministic 二分」「只写模型读不到的信息」（[现行版](https://code.claude.com/docs/en/best-practices)）；鉴于此类建议随模型代际自我反转，须以现行版为准。另有**（e）标准治理**（[AAIF 捐赠声明](https://openai.com/index/agentic-ai-foundation/)）与**（f）仓库实践**（[deepseek-harness](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md)）两类。

t2 出现了一个例外，值得单独标注：小米 MiMo 官方仓库 README 直接链接的官方域名技术文章《MiMo Code: Scaling Coding Agents to Long-Horizon Tasks》（2026-06-10，[链接](https://mimo.xiaomi.com/en/blog/mimo-code-long-horizon)）给出了一手机制论述——自述长输入会稀释「useful constraints and intentions」、把编排逻辑从自然语言 prompt 下沉为代码（「natural language is ambiguous, forgettable, and unverifiable」）、分层 prompt 注入约 65K token 预算、选文件而非向量库的理由是 reviewability。但**全文未出现 `AGENTS.md` / `CLAUDE.md` 字面**，即它回答「上下文该如何管理」，不回答「持久化指令文件该怎么写」。因此证据分层的准确表述是：**关于「AGENTS.md 该怎么写」的一手方法论知识，仍高度集中于 t0 与 harness 工具方**；中下层厂商提供的是仓库实践与相邻机制论述，不是该问题的规范答案。

### 3.4 论据三：等价文件的差异只在发现路径、覆盖规则、导入语法、截断阈值与文件名约定

| 维度 | [Codex](https://developers.openai.com/codex/guides/agents-md) | [Gemini CLI](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html) | [Claude Code](https://code.claude.com/docs/en/memory) | [GitHub Copilot](https://docs.github.com/en/copilot/reference/custom-instructions-support) / [Copilot CLI](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions) |
|---|---|---|---|---|
| 文件名 | `AGENTS.md` / `AGENTS.override.md` / 可配 fallback 列表 | 默认 `GEMINI.md`；`context.fileName` 可设为含 `AGENTS.md` 的列表 | `CLAUDE.md` / `CLAUDE.local.md`；缺省时可直读 `AGENTS.md` | 并列 `.github/copilot-instructions.md` / `AGENTS.md` / `CLAUDE.md` / `GEMINI.md`（code review 另含 `REVIEW.md`） |
| 发现路径 | 全局 `~/.codex` → 项目根 → 当前目录逐级，每目录至多取 1 个文件 | 全局 `~/.gemini/GEMINI.md` → 当前目录及父目录至 `.git` → 子目录（尊重 `.gitignore`/`.geminiignore`） | managed → user → project → local；工作目录及以上启动时加载，子目录按需 | repo root / cwd / 中间目录 / 目标文件路径下嵌套目录；另有 `$HOME/.copilot/` 与 `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` |
| 覆盖规则 | 拼接，**靠近 cwd 者因排在后面而覆盖**；32 KiB 截断、空文件跳过 | 全量拼接，无覆盖；`/memory show` 查看拼接结果 | 全量拼接、不覆盖；冲突时「Claude may pick one arbitrarily」；单文件 4 MiB 上限 | 合并去重，但明确「does not define a general precedence order… Avoid conflicting instructions」；另一份[仓库指令文档](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) 给出 personal > repository > organization |
| 导入语法 | 无内建导入，改用 fallback 文件名 | `@file.md` 相对/绝对路径 | `@path` 导入（外部路径需批准） | `@` + 相对路径；**`GEMINI.md` 与 `*.instructions.md` 不展开引用** |
| 同名不同义 | `AGENTS.override.md` 是覆盖机制 | — | **明确不读** `AGENTS.local.md`、`AGENTS.override.md` 及 `.agents/` 目录；可用 `claude-md-and-agents-md` 开关 | Copilot Chat 部分环境仅支持 `.github/copilot-instructions.md`，cloud agent 才支持三类 agent 文件 |

语义上它们同类：都是冷启动注入的、低特权的建议性项目文本——Claude Code 官方文档明写 `CLAUDE.md` 以 system prompt 之后的 user message 送达，且「context, not enforced configuration」（[Memory 文档](https://code.claude.com/docs/en/memory)）。差异全部落在**发现路径、覆盖规则、导入语法、截断阈值、文件名约定**五项工程参数上；AAIF 的标准化收口统一了格式名与治理归属（[捐赠声明](https://openai.com/index/agentic-ai-foundation/)），并未统一发现语义。尤须区分：以上规则保证的是**确定性注入**（哪些字节、按何种顺序进入上下文），**不保证确定性执行**（模型是否照做）——后者属第 2 章的架构性约束与第 5 章的执行分工。

### 3.5 分析：一手 ≠ 最佳实践，「空白」≠「没有实践」

如前述证据边界，[三家官方 harness 文件](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md)（187/368/133 行，索引＋hot-path rules＋验证脚本＋Avoid 清单＋维护节）以及本轮新检出的 Step-Code、MiMo-Code、minimax-code、Avernet 文件，**只回答「工程团队实际怎么写」，不回答「是否有效 / 为何有效」**，不可与 t0 的规范性方法论混为一谈，更不可当效果证据。更有说服力的是这些文件本身在被持续删减：小米提交记录写「keep only enforced conventions in the style guide」「Drop opencode-era stylistic preferences… from AGENTS.md」，阶跃写「Import AGENTS.md from CLAUDE.md instead of keeping a second rule set」「package files no longer restate the full `pnpm run check` step」（[MiMo-Code](https://github.com/XiaomiMiMo/MiMo-Code/)、[Step-Code](https://github.com/stepfun-ai/Step-Code)）。这说明即便模型团队自己的文件，也是特定 harness、特定任务、特定模型代际下可修订的产物；其「最佳实践」地位需要 [ETH Zurich 带对照性质的评估（arXiv:2602.11988）](https://arxiv.org/abs/2602.11988) 一类第三方实验检验，而非由作者身份授予。

由此又见一处「现状缺口」：[AGENTS.md 官网](https://agents.md/) 的 How to use 仍把 **Project overview** 列为「Cover what matters」首项，并称已有 60k+ 项目、OpenAI 主仓 88 份嵌套文件；而上述对照实验显示项目概览对成功率无增益。**厂商规范与第三方实验证据不一致**，与第 1 章指出的缺口同源。反方质疑亦应并陈：厂商自述同样带边界——小米那篇技术文章自承其 benchmark「still measure one-shot problem-solving ability on individual repository-level issues」，长程设计目标「mainly show their value in real development scenarios」，即一手自述亦非已验证结论。

最后，t1–t3 的空白更可能来自**公开发布习惯差异**（部分厂商以闭源商业产品或中文 docs 站点为主，GitHub 组织以模型权重/技术报告为主）、**渠道路径差异**与**内部实践未公开**，而非能力差异。本章补检索本身就是证据：初步调研阶段判为「无确切资料」的 xAI、MiniMax 与判为「无公开资料」的 t2 三家，经一轮官方组织定向检索即检出官方 harness 仓库与系统提示，MiniMax 另检出根目录 AGENTS.md 与生成/去重记录。故这一边界不仅是措辞纪律，也是方法论结论：**由「未找到」推不出「没有」**。

### 3.6 小结

按证据强度分层后，结论有三：其一，能回答「AGENTS.md 该怎么写且为何」的一手知识仍集中于 t0 模型团队与 harness 工具方，t1/t2 主要提供仓库实践样本，t3 仅能确认公开渠道覆盖范围；其二，`CLAUDE.md` / `GEMINI.md` / `.github/copilot-instructions.md` / `AGENTS.md` 在语义上同类、在发现机制上异构，标准化只收口了格式名；其三，官方身份不等于有效性，官网模板与对照实验的冲突、以及厂商自身文件的反复删减，共同构成「现状 ≠ 最佳实践」的又一组证据。

---

## 4. 描述性统计与对照实验的规范性边界及指令膨胀现状缺口

**本章论点**：用户已明确提示「现在的仓库大部分并没有很好的做好 AGENTS.md 的管理」，以及「对于论文的话，要尤其注意他们的预设前提……仅针对现有仓库进行统计分析的，就是只统计表象的，需要尤其注意甄别」。本章不生产写作规范（属第 5 章），而是先清理证据地基：把「描述性横截面统计」与「有对照／效果测量的实验性研究」严格分开，逐篇标定其预设前提与结论适用边界，并据此诊断现状缺口。核心判断是——**采纳规模与写法分布只回答「现在大家写什么」，不回答「应该写什么」；把前者升格为后者，是本章要系统排除的推论错误。**

### 4.1 方法论立论：为什么「统计现有仓库」推不出「最佳实践」

横截面仓库统计的样本是**当前实际存在的文件**，它内嵌了一个未经检验的前提：现存写法值得被描述、且具有代表性。研究设计上，它既无对照组，也不设置任何下游效果变量，因此「哪种写法更好」在其设计内不可判定。三重偏差进一步削弱其外部效度：**选择偏差**（只有「已经写了 AGENTS.md」的仓库才能入样）、**幸存者偏差**（管理失败、指令严重腐化的仓库可能根本没写或已删除文件，恰恰是最有诊断价值的样本被系统性排除）、以及**模型代际过时**（见 4.6，厂商规范本身在一年内自我反转，基于旧模型写作的样本迅速失去参照意义）。

「采纳率高 ≠ 有效」的一般性论证在此有直接数据：官方站称 AGENTS.md 已被 [60k+ 开源项目](https://agents.md/) 采用（同页另称「OpenAI 主仓在写作时有 88 份 AGENTS.md」）；一项 2026 年 10 月的普查在 22,854 个活跃仓库、13 种语言中检出 [26.3% 的仓库含指令文件、高星仓库达 59.6%](https://arxiv.org/abs/2610.31715)。但这两组数字都是**采纳计数**，不含任何效果测量。值得注意的是，该普查作者自述一个测量效度警示：模型标注之间的一致性「远高于」与人工审阅者主题分配的一致性（*a caution for automated annotation*）——即连「现在写了什么」这一层，也依赖标注器的可靠性。

因此，本报告确立一条证据规则：**描述性研究可作现状基线与假设来源，不得单独支撑规范性建议**；规范性结论必须来自第一性原理推导（第 1、2 章）叠加可复核的效果证据。

### 4.2 描述性典型：arXiv:2510.21413 逐篇拆解

[arXiv:2510.21413（MSR 2026）](https://arxiv.org/abs/2510.21413) 是一项自述为 *preliminary study* 的研究，样本为 466 个开源项目，分析开发者提供什么信息、如何呈现、以及文件随 commit 如何演化；发现「**尚无既定的内容结构**」，并把呈现方式归为 descriptive / prescriptive / prohibitive / explanatory / conditional 五类。

- **预设前提**：把「正在被写」视为「值得刻画」；把结构多样性当作结论而非待检验假设；以「文件存在于仓库」为入样条件。
- **可支持的结论**：实践多样性、采用趋势、维护演化路径、缺乏收敛结构。
- **不可支持的结论**：哪类呈现更有效、该写什么、写多长、写法与任务结果的关系。
- **决定性证据**：作者把「哪些结构或呈现上的改动能正向影响生成内容的质量」明确写为 *we see great potential in studying...* 的**未来工作**——即该研究设计内**根本不包含效果变量**，作者本人也未越界。

补搜显示这一范式在 2025–2026 年持续扩大，且恰好提供甄别样例：[Agent READMEs（arXiv:2511.12884）](https://arxiv.org/abs/2511.12884) 对 1,925 仓库 2,303 份文件做「首个大规模实证研究」，报告测试流程 75.9%、实现细节 70.8%、架构 68.1% 而安全仅 14.8%，并落脚于 *highlighting the need for improved tools and practices*；[Configuration Smells（arXiv:2606.15828, SCAM 2026）](https://arxiv.org/abs/2606.15828) 对 100 个仓库归纳 6 类「配置异味」，检出 Context Bloat 42%、Skill Leakage 35% 且与 Conflicting Instructions 高频共现。二者都**无对照组、无下游效果测量**，「smell / need」是把频次观察命名为规范判断，不是效果证据。

最锋利的甄别样例是 [Toward an Evidence-Based Template（arXiv:2611.02383, 2026-11-02 预印本）](https://arxiv.org/abs/2611.02383)：它从 1,000 仓库 3,387 份文件的**类别出现频次与类别新增比（CAR）**推导出含 10 个 Required、4 个 Gap、2 个 Recommended 的「循证模板」，其附带实验使结构类别覆盖率从 18.7% 升至 82.8%，并以 ROUGE-1 F1 评估生成文件。其**预设前提是「类别越普遍＝越该写」**，而因变量是**结构覆盖率与文本对齐度，不是下游任务成功率**——「模板更有效」在其设计内未被测量。该文另报告 40.4% 的文件仅有一次修改记录、人工检视的 386 对对应章节中 16.1% 指导互相分歧，这些是极有价值的**现状诊断数据**，但同样不是规范性证据。

### 4.3 实验性典型：arXiv:2602.11988 与 arXiv:2601.20404 严格并列

**[arXiv:2602.11988（v3, 2026-09）](https://arxiv.org/abs/2602.11988)** 的研究问题是「仓库级上下文文件对真实任务是否有效」。摘要显示其跨两个互补设置：SWE-bench 类任务配 LLM 生成文件、以及含开发者提交文件仓库的新建 issue 集；经全文核验，样本为 SWE-bench Lite 300 tasks / 11 仓库 + 自建 CTXbench 138 instances / 12 仓库，4 个 agent–model 配置（Claude Code+Sonnet-4.5、Codex+GPT-5.2、Codex+GPT-5.1 mini、Qwen Code+Qwen3-30b-coder），条件为 None / LLM / Dev。**因变量**是任务成功率（补丁使全部测试通过）、steps 与 LLM 推理成本（USD）。实验数据显示：文件**不普遍提升成功率**，成本平均增加逾 20%（两设置分别约 20% / 23%）；指令被很好遵循，但**仓库概览无用**；结论是「上下文文件适用于指定非标准编码实践，任何提升性能的尝试都应在部署前严格评估」。其**预设前提**是「测试通过率＝任务成功」「Python + 可自动判定的 issue/PR 类任务＝真实开发」；**边界**由作者自限给出：Limitations 明确仅评 Python、仅测 task resolution rate（代码效率与安全性列为未来工作）、未测 context rot 或长上下文退化，附录 B 更显示结论依赖「仓库已有文档」这一前提——删除文档后 LLM 生成文件反而平均提升 2.7%。摘要页未披露样本规模，须读全文方能取得。

**[arXiv:2601.20404（v2, 2026-03）](https://arxiv.org/abs/2601.20404)** 的研究问题是「AGENTS.md 对运行效率的影响」。样本为 10 仓库 / 124 PR（由 132 → 89 → 26 → 10 逐层筛选），单一 agent（Codex + gpt-5.2-codex），PR 限 ≤100 LoC、≤5 文件、已合并且晚于 AGENTS.md 引入。**自变量与对照**是 paired within-task：同一任务、同一仓库快照、同一 issue 输入、同一 agent，仅操纵 root AGENTS.md 存在／删除。**因变量**只有 wall-clock time 与 token 用量，且仅时间与 output tokens 达显著。实验数据显示 runtime 中位数 ↓28.64%、output token ↓16.58%，措辞始终为 *associated with*。全文核验确认三条关键边界：作者明写「对输出质量的全面评估（语义正确性、与已合并 PR 的功能等价）**超出本文范围**」；摘要中的 *comparable task completion behavior* **仅由随机抽样 50 个 PR 的人工 sanity check 支撑**（确认非空、非琐碎、非中断运行），不是成功率或正确性评估；机制解释使用 *we speculate*，并承认 agent stochasticity 与需跨 agent／模型复现。

**必须明确：两文测量维度不同、样本量不同，不可互相印证、也不可互相否定。** 前者测的是**任务成功率与成本**（跨 4 配置、2 数据集），后者测的是**效率**（单 agent、10 仓库），后者结构性地不测量正确性，前者结构性地不测量 wall-clock。任何把二者合并得出「AGENTS.md 有效／无效」的推论都是范畴错误。补搜的 [两 agent 消融研究（arXiv:2607.27250）](https://arxiv.org/abs/2607.27250) 从方法层面印证了这一点：其 17 任务 / 3 仓库 / 288 runs 的 gold-test 等价检验显示「context strategy 不显著改变 correctness（等价界 ≤10–15pp）」，并指出既往结论矛盾可能源于「单 agent 研究的任务落在各自不同的信息难度带」（borderline difficulty 的 agent 特异性 Spearman ρ=0.75）——**连实验性研究之间的分歧，都可能来自任务与 agent 组合，而非真实对立**。

#### 表 4-1 逐篇研究设计对照

| 研究 | 研究问题 | 样本 | 自变量与对照组 | 因变量（测了什么） | 预设前提 | 结论适用边界 | 能否支撑规范性建议 |
|---|---|---|---|---|---|---|---|
| [arXiv:2510.21413](https://arxiv.org/abs/2510.21413) 描述性 | 采用与内容／演化 | 466 项目 | 无；横截面 + commit 级 | 无（内容类别、演化） | 现存写法具代表性 | 只述现状；作者自列效果测量为 future work | **否** |
| [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 实验性 | 是否提升真实任务成功率 | 300+138 instances / 23 仓库 / 4 配置 | None / LLM / Dev | 成功率、steps、推理成本 | 测试通过＝成功；Python issue 类＝真实开发 | 基准＋issue 类任务；作者自限「部署前严格评估」；未测 context rot | 有限（「概览无用」「适合非标准实践」为条件性结论） |
| [arXiv:2601.20404](https://arxiv.org/abs/2601.20404) 实验性 | 是否影响运行效率 | 10 仓库 / 124 PR / 1 agent | 配对 with / without | wall-clock、token | 效率＝价值；输出未退化（仅 50 PR 抽查） | 仅效率维度；未测成功率／正确性；*associated with* | **否**（仅效率） |
| [arXiv:2611.02383](https://arxiv.org/abs/2611.02383) 描述性+生成实验 | 归纳「循证模板」 | 1,000 仓库 / 3,387 文件 | 有／无模板指引（生成实验） | 结构覆盖率、ROUGE-1 F1 | 类别频次＝应写 | 测结构对齐，非任务成功率 | **否**（前提未受效果检验） |
| [arXiv:2607.27250](https://arxiv.org/abs/2607.27250) 实验性 | 上下文注入是否改变正确性 | 17 任务 / 3 仓库 / 288 runs / 2 agents | 消融 + 等价检验 | gold-test correctness | 等价界 ≤10–15pp 可接受 | 「未测出差异」≠「无效果」；任务带 agent 特异 | 有限（提示任务选择偏差） |
| [arXiv:2610.30571](https://arxiv.org/abs/2610.30571) 实验性 | 过期指令是否压过代码、可否修复 | 40 Python 项目 / 320 runs | 用户文件有／无「以代码为准」一句 | 遵循来源占比、测试通过率 | 指令文件 vs 工具输出的优先级 | 构造项目、单句干预；预印本（投稿 SN CS） | 有限（可操作的治理假设） |

### 4.4 补搜范围、结果与分型（2025–2026）

检索执行：多轮 WebSearch（关键词含 `AGENTS.md empirical study`、`repository context file coding agent`、`CLAUDE.md evaluation`、`context file ablation`、`agent instruction file effectiveness`、MSR/ICSE/FSE 2026）+ arXiv 站内检索页与 Atom API（`all:"context file" AND all:"coding agent"`）。中文社区结果（CSDN／知乎／阿里云／腾讯云开发者社区等）绝大多数为二手转述或实践指南，**不作关键论据**。共 WebFetch 逐篇核验 8 篇预印本 abs 页，采用 7 篇；未发现除来源池外由自研模型团队官方发布的效果测量报告。按同一框架分型：

- **描述性／只统计表象**：[arXiv:2511.12884](https://arxiv.org/abs/2511.12884)（2,303 文件／1,925 仓库，无对照、无效果变量，预设前提「高频类别＝重要」）、[arXiv:2610.31715](https://arxiv.org/abs/2610.31715)（22,854 仓库普查，341,852 条陈述，74.0% 指令未给理由，预设前提「可标注即刻画」）、[arXiv:2606.15828](https://arxiv.org/abs/2606.15828)（100 仓库气味目录，预设前提「反模式可由频次定义」）、[arXiv:2611.02383](https://arxiv.org/abs/2611.02383)（混合型，见 4.2）。
- **实验性／有对照与效果测量**：[arXiv:2607.27250](https://arxiv.org/abs/2607.27250)、[arXiv:2611.04367](https://arxiv.org/abs/2611.04367)（86 个 benchmark tasks × 4 组 harness–model，把四种措辞 policy 插入 ACF，测 instruction compliance；发现 conditional 措辞被遵循更少，而 prescriptive 与 explanatory 何者更优**随组合而变**；IEICE 在审预印本）、[arXiv:2610.30571](https://arxiv.org/abs/2610.30571)。

其中 [arXiv:2611.04367](https://arxiv.org/abs/2611.04367) 与 arXiv:2510.21413 构成一组教科书式对照：后者只是**描述**出 prescriptive / explanatory / conditional 三类存在，前者把它们**作为自变量**测效果，且发现交互效应——恰好说明「描述性提出假设、实验性检验假设」的分工，也说明任何措辞类规范结论的边界都绑定具体 harness–model 组合。

### 4.5 引用错位完整核验：二手引用如何制造伪证据

[Thoughtworks Technology Radar「Agent instruction bloat」条目](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat) 的 Apr 2026 文本写道：*research suggests that hand-written versions are often more effective than LLM-generated ones*，其超链接指向的正是 arXiv:2601.20404。**并列核验结果**：[该文全文](https://arxiv.org/abs/2601.20404) 只有「仓库历史中实际存在的 root AGENTS.md」与「删除该文件」两个条件，**没有任何 LLM 生成 AGENTS.md 的条件**；全文出现 LLM 仅用于两处——以 gpt-oss-120b 分类／筛选文件内容、以及生成 GitHub-issue 风格的任务描述；文中 *human-written merged pull requests* 指人工编写的 **PR 代码变更**（用于 sanity check 参照），不是手写 AGENTS.md；*comparable task completion behavior* 是 50 个 PR 的人工抽查，与作者身份无关。**该转述在原文中无对应变量，属引用错位。**

进一步，[arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 也不能救场。其全文确有一句 *developer-committed files outperform LLM-generated ones by a significant margin of 7% on average*，但四重限定使其不等于「手写更优」：① 该差异是**相对 LLM 条件**，而 Dev 相对 None 基线的 +2.4% 不显著（p=0.21）；② 作者措辞是 *developer-committed / developer-provided*，并谨慎写为 *likely to be human-written or edited*，从未断言纯手写；③ 结论仅在 CTXbench 一个设置成立，附录 B 在移除文档后给出**反向**结果（LLM 生成文件反优于开发者文件）；④ 摘要的总结论是「两类文件都不普遍提升成功率」。另需说明：Thoughtworks 该条目页面同时存在 Nov 2026 更新文本，已不再包含手写／生成比较，改为「随每个新模型重评并删除不再需要的指令；progression testing, rather than regression testing」（[Thoughtworks 该条目](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat)）。

**对全报告的示范意义**：二手引用把「原文测了什么」悄悄换成「转述者想说什么」。本报告的纪律是——任何规范性断言必须回溯到原文的自变量与因变量；凡涉及手写 vs 生成，必须回原文核验，不采信任何转述。

### 4.6 厂商建议自我反转：「一次写定」为什么是错的

2025 年 7 月的 [Anthropic《Claude Code: Best practices for agentic coding》中文全文转载](https://cloud.tencent.com/developer/article/2545665) 与 [2025 版要点存档](https://raw.githubusercontent.com/PeterHiroshi/anthropic-blog-study/main/claude-code-best-practices/README_CN.md) 记录了当时的建议：用 `IMPORTANT` / `YOU MUST` 强调以提高遵循度、以 `think` < `think hard` < `think harder` < `ultrathink` 递增思考预算、在 CLAUDE.md 记录「项目核心文件和工具函数」及项目特定架构决策、把「上下文窗口迅速填满、随之性能下降」列为**核心约束**并激进管理。

而 [code.claude.com 现行版 Best Practices](https://code.claude.com/docs/en/best-practices) 的「What you can stop doing」表把这些逐条列入可停止做法：context window 不再是主要约束（1M 窗口＋自动 compaction）、`IMPORTANT`/全大写反而使模型 over-apply、在 CLAUDE.md 描述架构（*Claude reads the repo itself. Only write down what isn't in there.*）、`think hard` 类短语（effort 才是控制项）。该表自述：*Every one of these was good advice when it was written. Most were workarounds for something the model couldn't reliably do yet, and the models have since caught up.* ——**这是厂商自身承认规范随模型代际失效的一手证据**。按参数卡要求，须以现行版为准，早期文本仅作历史对照。

结论有双重含义：其一，「现状 ≠ 最佳实践」不止是社区问题，连厂商官方规范都会在一年内反转；其二，任何「一次写定、长期不改」的 AGENTS.md 治理方式在架构上就是错的。[Claude Code Memory 文档](https://code.claude.com/docs/en/memory) 的 `/doctor prompt-audit` 明确检索三类问题——*instructions written for older models*、指向已不存在文件／命令的引用、以及**互相矛盾的指令文件**；`/doctor` 的裁剪对象正是 directory layouts、dependency lists、architecture overviews 这类「可从代码推导」的内容。术语上须区分：现行版把不可弯曲的规则下沉为 hook / deny rule，属**确定性执行**；把规则写入上下文只是**确定性注入**，二者不可混同（机制见第 2 章）。

### 4.7 现状缺口的证据清单（辩证呈现）

把前述证据按五类缺口归集：

- **指令膨胀**：业内观点认为指令随时间累积而「long, contradictory and stale」（[Thoughtworks Radar](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat)）；实验／挖掘数据显示 [Context Bloat 达 42%、Skill Leakage 35%，且与 Conflicting Instructions 高频共现](https://arxiv.org/abs/2606.15828)。
- **过期、与代码冲突、无人维护、无验证**：[`/doctor prompt-audit`](https://code.claude.com/docs/en/memory) 的三类检索项即官方对这四种失效的确认；[40.4% 文件仅有一次修改记录、16.1% 对应章节指导分歧](https://arxiv.org/abs/2611.02383) 是无维护的量化证据。最强的是因果证据：[arXiv:2610.30571](https://arxiv.org/abs/2610.30571) 在 40 个构造项目中让指令文件与既有代码约定冲突，320 runs 显示无干预时 **90.1%** 的 agent 选择遵循过期文件；Claude Code 在 80 次运行中 79 次**指出冲突却仍遵循文件**，Codex CLI 在 20/80 次中重写既有代码、其中 17 次导致项目自身测试失败；仅加一句「以现有代码为准」即使遵循代码的比例由 4.2% 升至 91.4%、测试通过率由 89.4% 升至 100%。
- **官网模板与实验结论背离**：[agents.md](https://agents.md/) 的「How to use → Cover what matters」仍把 **Project overview** 列为首选项，而 [arXiv:2602.11988 实验](https://arxiv.org/abs/2602.11988) 显示仓库概览无助于定位文件、也不改善成功率（第 1 章已标为现状缺口）。
- **厂商自身文件持续删减**：[XiaomiMiMo/MiMo-Code](https://github.com/XiaomiMiMo/MiMo-Code/) 与 [stepfun-ai/Step-Code](https://github.com/stepfun-ai/Step-Code) 的删减提交（第 3 章已核验），说明连一手实践者也在反向收敛。
- **厂商自述，方法未公开（见『待完善事项』第 1 条）**：[claude.dev 2026-07-24 博文](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/) 称「为 Claude 5 代模型删除超过 80% 的系统提示词，在内部编码评测中无可测量损失」。这是**厂商自述且方法未公开**——页面未披露评测方法、数据集、样本量、控制条件与结果数据，仅出现 *on our coding evaluations* 一处概括，**不可作为「规则无用」的独立证据**，只作「厂商实践方向」的旁证。

**辩证结论**：上述证据共同支持「现状管理普遍不到位」这一判断，但**不推出「AGENTS.md 无用」**。第 1／2 章已论证它补的是冷启动信息缺口；[arXiv:2602.11988 作者自述](https://arxiv.org/abs/2602.11988) 亦明确保留「适用于指定非标准编码实践」的价值；而 [arXiv:2610.30571](https://arxiv.org/abs/2610.30571) 的单句修正实验恰恰证明**治理方式可显著改善结果**。问题在内容与治理，不在存在性——这正是第 5 章规范要处理的对象。

### 4.8 反方与边界：不写简单的好坏二分

须为描述性研究辩护：它虽不能给规范，却能揭示实践多样性、采用趋势与维护演化，是提出假设的必要前置。[arXiv:2510.21413](https://arxiv.org/abs/2510.21413) 的五类呈现方式直接被 [arXiv:2611.04367](https://arxiv.org/abs/2611.04367) 用作自变量；[arXiv:2610.31715](https://arxiv.org/abs/2610.31715) 的 26.3% 采纳率与 74% 无理由指令是有效的现状基线；[arXiv:2606.15828](https://arxiv.org/abs/2606.15828) 的气味目录为后续效果研究提供可操作的自变量。有趣的是，[arXiv:2601.20404](https://arxiv.org/abs/2601.20404) 与 [arXiv:2510.21413](https://arxiv.org/abs/2510.21413) 作者名单有四人重叠——同一研究团队从描述性走向实验性，本身就是「先刻画、后检验」的正常研究演进，也更说明不该把两类结论放在同一证据层级上使用。

同样须承认实验性研究自身的边界：**基准任务与真实开发的差距**（arXiv:2602.11988 仅 Python、以测试通过率代理成功、结论依赖「仓库已有文档」前提；[arXiv:2607.27250](https://arxiv.org/abs/2607.27250) 仅 3 仓库 17 任务，等价界宽 ≤10–15pp，只能说「未测出差异」）；**样本量**（arXiv:2601.20404 为 10 仓库 124 PR、单一 agent，且经 132→10 的严格筛选）；**指标选择**（前者测效率不测正确性、后者测成功率不测安全与代码效率、[arXiv:2611.02383](https://arxiv.org/abs/2611.02383) 测结构覆盖率与 ROUGE 而非任务结果）。因此描述性给「地图」、实验性给「局部因果」，二者都不足以单独产出「普遍最佳实践」。

### 小结

本章三条结论：**一**，横截面仓库统计内嵌「现存写法具代表性」这一未检验前提，无对照组、无效果变量，受选择／幸存者偏差与模型代际过时影响，只能回答「现在写什么」；采纳率高 ≠ 有效。**二**，arXiv:2510.21413 属描述性（作者自列效果测量为 future work），arXiv:2602.11988 与 arXiv:2601.20404 属实验性但测的是不同因变量，二者**不可互相印证、亦不可互相否定**，任何合并推论都是范畴错误；二手引用错位（见『待完善事项』第 2 条）是「伪证据」的典型生成机制，厂商规范自我反转（见 4.6 节）则证明「一次写定」治理在架构上失效。**三**，膨胀、过期、冲突、无验证、无人维护的证据链支持「现状缺口」诊断，但该诊断指向内容与治理，不否定 AGENTS.md 的存在价值。本章完成对第 5 章的清基。

---

## 5. 最小高信号指令集规范、确定性执行分工与渐进式验证闭环

### 5.1 论点：规范是前四章约束的解，不是业界建议的汇编

前四章已把问题收敛为两个不随工程手段消失的约束：冷启动时的项目专属信息缺口（第 1 章），以及有限注意力预算下的信号竞争（第 2 章）；第 4 章进一步清基——缺口在内容与治理，不在文件存在性，描述性统计无资格充当规范依据。故本章每条规范都写成「约束 → 推导 → 规则」，只保留可回溯到机制或对照实验的条目。一个前置辨析必须先行：**确定性注入 ≠ 确定性执行**。指令文件在会话启动时被确定地读入（注入时机确定），但其内容是否被遵循仍由概率采样决定（执行效力不确定）；混淆二者，正是「把文件当强制配置」这一误区的来源。

### 5.2 规范清单：写什么、不写什么、多长

**N1 只写模型从代码读不到的信息。**【依据：第 1 章「瓶颈是信息增量而非服从度」＋arXiv:2602.11988 概览无用】[Claude Code Best Practices 现行版](https://code.claude.com/docs/en/best-practices)把判据写成两问：「Could it find this by reading the code? Would it actually go and look?」两问答案皆为「是」即不写；应留的是团队决策、部署流程、环境变量、非标准编码实践。

**N2 不写架构、目录结构、依赖清单与项目概览。**【依据：第 2 章 n² 摊薄＋arXiv:2602.11988 成本 +20% 且概览无增益】[现行版](https://code.claude.com/docs/en/best-practices)把「Describe your architecture and file layout in CLAUDE.md」列入 *What you can stop doing* 表，[Memory 文档](https://code.claude.com/docs/en/memory)的 `/doctor` 亦明确清理 directory layouts / dependency lists / architecture overviews。此处须点明一处冲突：[AGENTS.md 官网](https://agents.md/)在「Cover what matters」中把 **Project overview** 与 **Code style guidelines** 列为 popular choices，示例模板还写入 `single quotes, no semicolons` 一类风格条目；但「流行/现存写法」内嵌第 4 章已拒斥的「具代表性」未检验前提，且与 [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 的对照结果相悖（该实验测的是文件整体成功率与成本，并未单独测风格条目；风格条目之所以不写，应由 N1 判据收口——引号／分号风格可由现有代码推断）。按证据优先级，规范应以机制与对照实验为准，不照抄模板——这是用户「现状≠最佳实践」约束的具体落点。

**N3 最有用的一行是能返回 pass/fail 的验证命令。**【依据：第 1 章「可预测发现路径」缺失】[官方原话](https://code.claude.com/docs/en/best-practices)：「If your repo already has one of these, the single most useful line in your `CLAUDE.md` is the command that runs it.」展开见 5.5。

**N4 设长度上限，但 minimal ≠ short。**【依据：第 2 章注意力预算＋arXiv:2602.11988 成本项】[Codex AGENTS.md 指南](https://developers.openai.com/codex/guides/agents-md)设 32 KiB 截断，memory guide 建议「staying under 200」；二者分别是硬截断与软建议，性质不同。[Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) 的「最小高信号 token 集」指信噪比而非字数：删掉概览省下的预算若换成十条等价规则重述，信号密度反降；一条必要的非标准部署流程即便占二十行也该留。

**N5 格式、lint、风格交给 CI，不写自然语言。**【依据：第 2 章概率采样使遵循非确定＋Codex 指南「格式/lint 留给 CI」】[现行版](https://code.claude.com/docs/en/best-practices)给出的 formatter 是 `PostToolUse` hook，理由极精确：「whether Claude remembers the rule no longer matters, because Claude isn't the one running it」。

**N6 措辞不给普适教条，须绑定 harness–model 验证。**【依据：第 3 章证据分层＋厂商建议随模型代际自我反转】[arXiv:2611.04367](https://arxiv.org/abs/2611.04367) 实验表明 conditional 措辞遵循更少，而 prescriptive / explanatory 的优劣随 harness–model 组合反转。故本章不给出「一律用祈使句」类规则，只给元规则：措辞变更必须在本地组合上测量后再固化。

**N7 用一条元规则显式化信息源优先级。**【依据：第 2 章冲突调和耗推理预算】[arXiv:2610.30571](https://arxiv.org/abs/2610.30571) 报告单句「以现有代码为准」使遵循代码比例由 4.2% 升至 91.4%、测试通过率由 89.4% 升至 100%。可操作含义是：与其堆叠概览让模型自行对齐，不如声明冲突时的裁决次序。**边界**：该结果为构造项目、单 agent 的预印本，非真实仓库长期维护证据，不可外推为普适解。

### 5.3 谁维护：把腐化纳入工程流程

【依据：第 4 章「无人维护/过期/冲突」三类缺口＋厂商建议随模型代际自我反转】[现行版](https://code.claude.com/docs/en/best-practices) `/doctor prompt-audit` 明确标记三类应删指令——为旧模型写的、指向已删除文件的、互相矛盾的，这给出可执行判据，宜设为固定维护节（Editing these instructions / Where to Update Instructions）并随模型代际升级触发。多格式并存会放大分歧：[arXiv:2611.02383](https://arxiv.org/abs/2611.02383) 记录到 16.1% 的分歧率，而等价文件发现机制本就异构（第 3 章）；[官网](https://agents.md/)给出的 `ln -s AGENTS.md AGENT.md` 与 [Memory 文档](https://code.claude.com/docs/en/memory)的软链 / `@import` 即「单一真源」的两种实现。最后把「指令文件随代码变更」纳入 review：现行版 *What you can stop doing* 表自述「Every one of these was good advice when it was written」，等于承认建议随代际自我反转——「一次写定」在治理上不成立（见 4.6 节）。

### 5.4 判定权归属：advisory 与确定性执行的分流（本章关键）

【依据：第 2 章架构性结论＋现行版二分＋hooks 官方文档】[Claude Code Best Practices 现行版](https://code.claude.com/docs/en/best-practices)把每条指令二分为 Non-deterministic（prompt / CLAUDE.md / skills，模型决定）与 Deterministic（hooks / deny rules / managed settings / Stop hook / CI，harness 执行，「no matter what the model decided to do」），并直言「`CLAUDE.md` is for guidelines, and Claude is allowed to decide a guideline doesn't apply」；移出后的效果被表述为「the outcome doesn't depend on what the model remembers or decides」。

由此给出**分流规则**：① 允许例外、需读代码才能判断 → 留作建议性、高信号、可判定的自然语言；② 零例外且可自动判定 → 判定权下沉至 [hooks](https://code.claude.com/docs/en/hooks) / deny rules / [managed settings](https://code.claude.com/docs/en/settings) / Stop hook / CI / pass-fail 检查 / evidence 要求。两条来自一手的精度修正：其一，现行版指出 `prompt` / `agent` hook 仍「asks another model to decide」，不是确定性，真强制须 `command` / `http` hook 并配 `onFailure: "block"`；其二，[hooks 参考](https://code.claude.com/docs/en/hooks)声明 `if` 过滤仅为 best-effort，硬 allow/deny 须交给 permission system，且通过退出码机制阻断须用 `exit 2`、`exit 0` 静默不等于批准（另有 `exit 0` + JSON 结构化决策亦可阻断）。（该页与 settings 页均未使用 "deterministic" 一词，此为本章基于架构的定性。）**连 hook 内的模式匹配都被官方判定为不足以保证，自然语言更无可能**——这印证第 2 章：LLM 是概率采样器，无法由自身保证「一定遵守」，故零例外规则的判定权外置是架构必然，而非经验偏好。反向边界同样来自官方：「keep the enforced set small」，若堆到四十条「most are probably preferences」——确定性侧也有误报与维护成本，不可滥用分流。

### 5.5 验证闭环：从「一行命令」到可判定证据链

【依据：第 1 章发现路径缺失＋第 2 章自述亦为概率输出＋arXiv:2602.11988 元规则】闭环四环：

1. **指令层给出可自动判定的检查**。[官方](https://code.claude.com/docs/en/best-practices)要求「Give Claude something it can run that comes back pass or fail」；[MiniMax 官方 harness](https://github.com/MiniMax-AI/minimax-code) 的 `mcode init .` 甚至要求任务描述即写明 expected result / allowed changes / how to verify（一手仓库实践只回答「实际怎么写」，非效果证据）。
2. **提交前跑验证并交 evidence，而非自述**。[官方原话](https://code.claude.com/docs/en/best-practices)：「ask Claude for *evidence*: the command it ran and its output, or the screenshot. `All tests pass` on its own tells you very little」。
3. **CI 作为确定性兜底**。[headless / programmatic 文档](https://code.claude.com/docs/en/headless)给出 `claude -p`、非 0 退出码供脚本分支、`--output-format json` + `--json-schema` 结构化输出、可据 `mcp_server_errors` 做 CI gate，以及 GitHub Actions / GitLab CI/CD 集成入口；尤其 `--bare` 在 CI 中主动「without loading ... `CLAUDE.md`」，文档给出的适用说明是「Bare mode is useful for CI and scripts where you need the same result on every machine」——官方在实现层面即把强制判定放在流水线而非会话内建议，与 5.4 互证。
4. **确定性检查要回灌证据，形成闭环而非门禁**。在可阻断事件上，[hooks 文档](https://code.claude.com/docs/en/hooks)的 `exit 2` 把 stderr 作为阻断原因交回模型；[现行版](https://code.claude.com/docs/en/best-practices) Stop hook 示例中编译器错误被重定向回 Claude，「it gets the compiler errors back and keeps going until `tsc` passes」。

**元规则**：[arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 要求任何性能改进主张部署前严格评估，其附录 B 还出现删文档后 LLM 生成文件反提升 2.7% 的情形——故「写后必须验证」且验证本身需对照，与第 4 章「描述性统计不作规范依据」闭合。

### 5.6 分发机制演化，以及「AGENTS.md 会不会消亡」

【依据：第 2 章信号层结论＋progressive disclosure＋厂商建议随模型代际自我反转＋厂商自述且方法未公开】[skills 官方文档](https://code.claude.com/docs/en/skills)把按需加载写成机制（页面未用 "progressive disclosure" 原词）：描述常驻、正文「loads only when it's used」、支持文件按需读取，并建议 SKILL.md 控制在 500 行内；[现行版](https://code.claude.com/docs/en/best-practices)另给 `.claude/rules/` 的 `paths:` 条件加载。现行版同时把「Use subagents to protect your context window」列入可停用旧建议，却保留 subagent 用于并行、噪声隔离与独立 review（`/code-review` 用未看过推理过程的 fresh subagent）——其价值不在省窗口容量，而在信号隔离与独立验证，与第 2 章「容量问题迁移为信号问题」一致。这些演化**不是取消文件**，而是把常驻 token 预算让给高信号内容，改变的是承载位置与注入时机。

**正面回答：功能不消亡，条目持续淘汰。** 主张消亡一侧，[Thoughtworks Radar](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat)（Nov 2026）提出「reevaluate instructions against each new model and delete those it no longer needs; **progression testing, rather than regression testing**」，并主张先把指令「convert ... into deterministic hooks or checks」（该页早期条目关于手写/生成文件优劣的二手引述存在错位，见『待完善事项』第 2 条，不予采信）。其援引的「删掉 80% 系统提示且无可测量损失」源自 [claude.dev 厂商博客](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/)，属**厂商自述、方法未公开（见『待完善事项』第 1 条）**，Thoughtworks 转述同源，故只能列为厂商主张/业内观点，不构成「规则无用」的独立证据。本章判断是：第 2 章已论证 context rot 源于 softmax 自注意力摊薄与训练分布偏差，不随窗口扩大或模型规模归零（现行版自己在停用「窗口是主约束」时仍保留「a focused context gets you better answers and costs less」）；只要①跨会话无状态带来的冷启动信息缺口与②注意力预算稀缺这两个约束仍在，就需要一个承载项目专属最小信息的机制。**文件形态会演化（skills / auto-memory / 条件加载），但功能不会消亡**；厂商建议随模型代际的自我反转否证的是「一次写定」，不是「需要写」。

### 5.7 最强反方、适用边界与一条可证伪准则

【依据：第 4 章因变量不可互证＋前述对照实验】最强反方是：既然 [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) 显示 context file 未普遍提升成功率且成本 +20%，[arXiv:2607.27250](https://arxiv.org/abs/2607.27250) 显示 context strategy 未显著改变 correctness，为何还要写？诚实回应四点：① 二者测的是特定 benchmark 的成功率、成本与 correctness，均非「项目专属非标准实践是否被遵循」这一因变量（第 4 章已证二者不可互证互否，且二者亦未以本章因变量为测度），且属构造/实验室任务而非长期仓库治理；② 它们否定的是概览与冗余内容——arXiv:2602.11988 自身结论即含「概览无用、适合非标准实践」，与第 1 章「信息增量」瓶颈同向；③ [arXiv:2610.30571](https://arxiv.org/abs/2610.30571) 说明治理方式本身是可显著改善结果的有效变量（边界见 N7）；④ 正确姿态因而是**默认不写、按需最小写、写后必须验证、随模型重估**，而非「一律不写」或「一律写满」。

据此给出**可证伪的实践准则**：对任一拟写入条目，先问「模型能否从代码或工具直接得到」，能则不写；不能则写入并同时配一条可返回 pass/fail 的检查；部署前以对照方式测量其边际收益（遵循率/成功率/成本至少一项显著改善且成本不显著恶化），无显著改善即删除；每次模型代际升级重测，未复现即删除。该准则预言「按此筛出的最小集在对照测量中优于照抄模板的全量集」，并预言「零例外规则留在自然语言中会出现可观测的漏遵循率」；若在某仓库反复测量显示二者无差异、或漏遵循率长期为零，则准则在该场景被证伪，应放宽而非强行推广。本章规范自身证据强度并不齐平：官方规范是规范性方法论（t0）而非效果证据（官方仓库文件亦只回答实际怎么写），单预印本须本地复现，描述性统计不作规范依据。

### 5.8 规范—依据—强度—边界收口表

| 规范 | 推导依据 | 证据强度 | 适用边界 |
|---|---|---|---|
| N1 只写读不到的信息 | 第 1 章信息增量＋arXiv:2602.11988 概览无用＋现行版两问 | 机制推导＋对照实验＋官方规范 | 通用；「读不到」随模型能力变化需重估 |
| N2 不写概览/架构/目录 | 第 2 章 n² 摊薄＋arXiv:2602.11988 成本 +20%＋N1 判据 | 机制推导＋对照实验 | 与官网模板冲突，以实验为准 |
| N3 验证命令 | 第 1 章发现路径缺失 | 官方规范＋机制推导 | 需仓库已有可判定检查 |
| N4 长度上限、minimal≠short | 第 2 章预算＋32 KiB 截断 | 官方规范（硬截断/软建议）＋方法论 | 硬截断随 harness 而异 |
| N5 格式/lint 交 CI | 第 2 章非确定遵循＋Codex 指南 | 官方规范＋机制推导 | 需可自动判定的检查 |
| N6 措辞绑定组合验证 | 第 3 章证据分层＋厂商建议随模型代际自我反转 | 单预印本（组合依赖） | 不可给普适措辞教条 |
| N7 元规则＋5.3 维护 | 第 2 章冲突调和＋第 4 章治理缺口＋厂商建议随模型代际自我反转 | 单预印本＋官方规范 | 构造项目/单 agent，需本地复现 |
| 5.4 判定权分流 | 第 2 章概率采样（架构性）＋官方二分＋hooks 参考 | 机制推导＋官方规范（规范性，非效果证据） | 确定性侧须保持小集合 |
| 5.5 验证闭环 | 第 1、2 章＋arXiv:2602.11988 元规则 | 官方规范＋对照实验元规则 | CI 需可自动判定 |
| 5.6 功能不消亡 | 第 1、2 章两个根本约束 | 机制推导（分析性结论） | 条目级持续淘汰 |

### 本章小结

五章推导链可收束为一句：冷启动的信息缺口决定了必须有一个最小信息基线，注意力预算决定了它必须高信号且可判定，证据分层决定了何种主张有资格成为规范，清基否证了「现状即最佳实践」，于是落到本章——**把建议性内容压到最小、把零例外判定移出模型、把每一次写入都关进可测量、可回退、随代际重估的验证闭环**。

---

## 结论

引言追问「现存写法能否自我证成」，本研究答曰不能：AGENTS.md 的最佳实践不是静态模板，而是由约束推导、由对照测量裁定的规程——**默认不写，只补模型从代码读不到且可验证的信息，零例外判定权外置给 harness，以 pass/fail 检查测量边际收益，无显著改善即删**。价值在信息增量与可维护性，而非存在性 ([Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))。

三条跨章发现支撑此判断。① 最有用的一行是可返回 pass/fail 的验证命令；「指令被遵循而概览无用、成本反增 20%」表明瓶颈是信息增量而非服从度 ([arXiv:2602.11988](https://arxiv.org/abs/2602.11988); [Best Practices](https://code.claude.com/docs/en/best-practices))。② 过期与冲突是可测失效源：90.1% 的运行遵循过期指令，一句「以现有代码为准」使遵循代码 4.2%→91.4%、测试通过率 89.4%→100% ([arXiv:2610.30571](https://arxiv.org/abs/2610.30571))。③ 措辞无跨组合最优解，优劣随 harness–model 组合反转 ([arXiv:2611.04367](https://arxiv.org/abs/2611.04367))，故硬约束须交权限系统与 CI 而非 prompt 或 hook 匹配 ([Hooks](https://code.claude.com/docs/en/hooks))。

边界须诚实：测成功率/成本与测 wall-clock/token（明言质量评估「超出本文范围」）的两组实验严格并列、不可互证互否，任务难度带差异（ρ=0.75）可解释部分矛盾 ([arXiv:2607.27250](https://arxiv.org/abs/2607.27250))；官方仓库文件只答「实际怎么写」，且 2025 早期强调写法已被现行版列入「What you can stop doing」 ([历史对照](https://cloud.tencent.com/developer/article/2545665))，故有限收益不可回避、「一次写定」治理在架构上失效。

消亡争议的答案是**功能不消亡、条目持续淘汰**：两条约束不随窗口扩容归零，形态却已演化为按需加载的 skills、条件规则乃至 headless `--bare` 主动不加载文件 ([Skills](https://code.claude.com/docs/en/skills); [Headless](https://code.claude.com/docs/en/headless))。未来应沿 progression testing ([Thoughtworks](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat))，以任务成功率为因变量、按 harness–model 组合重复对照，每代模型升级后重测——让规范可证伪，而非被信奉。

---

## 参考文献

- AGENTS.md 官方规范站点, n.d., AGENTS.md [来源链接](https://agents.md/)（规范模板；60k+ 项目采用仅为规模数据，非效果证据）
- OpenAI co-founds the Agentic AI Foundation, donates AGENTS.md, 2025-12-09, OpenAI [来源链接](https://openai.com/index/agentic-ai-foundation/)
- Agentic AI Foundation, n.d., AAIF [来源链接](https://aaif.io/)
- Custom instructions with AGENTS.md（OpenAI Codex 官方指南）, n.d., OpenAI [来源链接](https://developers.openai.com/codex/guides/agents-md)
- Provide context with GEMINI.md files, n.d., Google [来源链接](https://google-gemini.github.io/gemini-cli/docs/cli/gemini-md.html)
- GitHub Copilot copilot-instructions.md GA, 2025-08-06, GitHub [来源链接](https://github.blog/changelog/2025-08-06-copilot-code-review-copilot-instruction-md-support-is-now-generally-available/)
- Claude Code Best Practices（现行版）, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/best-practices)（现行版；与 2025 早期建议逐条反转）
- Claude Code Memory 官方文档, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/memory)
- Claude Code Hooks reference, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/hooks)
- Extend Claude with skills, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/skills)
- Settings files and precedence, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/settings)
- Run Claude Code programmatically, n.d., Anthropic [来源链接](https://code.claude.com/docs/en/headless)
- Context windows — Claude Platform Docs, n.d., Anthropic [来源链接](https://docs.anthropic.com/en/docs/build-with-claude/context-windows)
- Adding repository custom instructions for GitHub Copilot, n.d., GitHub [来源链接](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)（同一页并列四类等价指令文件）
- GitHub Docs: Support for different types of custom instructions, n.d., GitHub [来源链接](https://docs.github.com/en/copilot/reference/custom-instructions-support)
- GitHub Enterprise Cloud Docs: Copilot CLI custom instructions, n.d., GitHub [来源链接](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- 智谱 AI 开放文档：接入工具, n.d., 智谱 AI [来源链接](https://docs.bigmodel.cn/cn/coding-plan/tool/others)
- 字节跳动 Seed2.0 官方页, n.d., ByteDance Seed [来源链接](https://seed.bytedance.com/zh/seed2)
- Effective context engineering for AI agents, n.d., Anthropic [来源链接](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)（工程立场文章，非受控实验）
- Building effective agents, 2024-12-19, Anthropic [来源链接](https://www.anthropic.com/engineering/building-effective-agents)
- The new rules of context engineering for Claude 5 generation models, 2026-07-24, Anthropic [来源链接](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/)（厂商自述，方法未公开）
- GPT-5 prompting guide, 2025-08-07, OpenAI [来源链接](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)（archived；观察性）
- Context Rot: How Increasing Input Tokens Impacts LLM Performance, 2025-07-14, Chroma Research [来源链接](https://www.trychroma.com/research/context-rot)（确立因果效应，自述不解释退化机制）
- MiMo Code: Scaling Coding Agents to Long-Horizon Tasks, 2026-06-10, Xiaomi MiMo [来源链接](https://mimo.xiaomi.com/en/blog/mimo-code-long-horizon)（相邻机制论述，全文未提 AGENTS.md）
- Agent instruction bloat（Thoughtworks Technology Radar）, 2026, Thoughtworks [来源链接](https://www.thoughtworks.com/en-ca/radar/techniques/agent-instruction-bloat)
- Attention Is All You Need, 2017, Vaswani et al., arXiv:1706.03762 [来源链接](https://arxiv.org/abs/1706.03762)
- Lost in the Middle, 2023（arXiv）/ TACL 2024, Liu et al., arXiv:2307.03172 [来源链接](https://arxiv.org/abs/2307.03172)（非 AGENTS.md 专属实验）
- RULER: What's the Real Context Size of Your Long-Context Language Models?, 2024, COLM 2024, arXiv:2404.06654 [来源链接](https://arxiv.org/abs/2404.06654)
- The Instruction Hierarchy, 2024, Wallace et al., arXiv:2404.13208 [来源链接](https://arxiv.org/abs/2404.13208)（预设为提示注入防御，非仓库文件实测）
- NoLiMa: Long-Context Evaluation Beyond Literal Matching, 2025, ICML 2025, arXiv:2502.05167 [来源链接](https://arxiv.org/abs/2502.05167)
- Context Engineering for AI Agents in Open-Source Software, 2026, Mohsenimofidi, Galster, Treude & Baltes, MSR 2026, arXiv:2510.21413 [来源链接](https://arxiv.org/abs/2510.21413)（仅描述性统计，效果测量列为 future work）
- Agent READMEs: An Empirical Study of Context Files for Agentic Coding, 2025（v2 2026-08）, Chatlatanagulchai et al., arXiv:2511.12884 [来源链接](https://arxiv.org/abs/2511.12884)（仅描述性统计）
- On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents, 2026（v2 2026-03）, arXiv:2601.20404 [来源链接](https://arxiv.org/abs/2601.20404)（对照实验：wall-clock 与 token、单 agent；输出质量评估「超出本文范围」；Thoughtworks 误引为「手写优于 LLM 生成」，不予采信）
- Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?, 2026（v3 2026-09）, Gloaguen, Mündler-Sasahara, Müller, Raychev & Vechev, arXiv:2602.11988 [来源链接](https://arxiv.org/abs/2602.11988)（对照实验：成功率与成本，4 配置 2 数据集；与上条严格并列，不可互证互否）
- Configuration Smells in AGENTS.md Files, 2026, SCAM 2026, arXiv:2606.15828 [来源链接](https://arxiv.org/abs/2606.15828)（仅描述性统计）
- Do Context Files Help Coding Agents? A Two-Agent Ablation Study, 2026-07-28, Khatri, arXiv:2607.27250 [来源链接](https://arxiv.org/abs/2607.27250)（双 agent 消融；correctness 无显著变化，任务难度带 ρ=0.75）
- Stale Instruction Files Override the Code in Coding Agents: A One-Sentence Fix, 2026-10-22, Xu & Wu, arXiv:2610.30571 [来源链接](https://arxiv.org/abs/2610.30571)（构造项目、单 agent 预印本）
- A Census of AGENTS.md and CLAUDE.md, 2026-10, arXiv:2610.31715 [来源链接](https://arxiv.org/abs/2610.31715)（仅描述性统计：22,854 仓库普查）
- Toward an Evidence-Based Template for Agent Instruction Files, 2026-11-02, Shanto, Islam, Asaduzzaman & Ngom, arXiv:2611.02383 [来源链接](https://arxiv.org/abs/2611.02383)（预设前提「普遍＝应写」；仅测结构覆盖率与 ROUGE-1，未测任务成功率）
- Do Coding Agents Follow AGENTS.md? The Effect of Policy Wording, 2026-11-03, arXiv:2611.04367 [来源链接](https://arxiv.org/abs/2611.04367)（措辞作为自变量的对照实验；优劣随组合反转）
- deepseek-harness/AGENTS.md, n.d., DeepSeek（t0） [来源链接](https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md)（官方仓库自身指令文件，非效果证据）
- qwen-code/AGENTS.md, n.d., QwenLM（t1） [来源链接](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)（官方仓库自身指令文件，非效果证据）
- kimi-code/AGENTS.md, n.d., MoonshotAI（t1） [来源链接](https://github.com/MoonshotAI/kimi-code/blob/main/AGENTS.md)（官方仓库自身指令文件，非效果证据）
- MoonshotAI/kimi-cli Issue #2401, n.d., MoonshotAI [来源链接](https://github.com/MoonshotAI/kimi-cli/issues/2401)
- XiaomiMiMo/MiMo-Code, n.d., Xiaomi MiMo（t2） [来源链接](https://github.com/XiaomiMiMo/MiMo-Code/)
- xai-org/grok-build, n.d., xAI（t1） [来源链接](https://github.com/xai-org/grok-build)
- xai-org/grok-prompts, n.d., xAI（t1） [来源链接](https://github.com/xai-org/grok-prompts)
- stepfun-ai/Step-Code, n.d., StepFun（t2） [来源链接](https://github.com/stepfun-ai/Step-Code)
- MiniMax-AI/minimax-code, n.d., MiniMax（t1） [来源链接](https://github.com/MiniMax-AI/minimax-code)
- inclusionAI/Avernet, n.d., Ant Group inclusionAI（t2） [来源链接](https://github.com/inclusionAI/Avernet)
- meituan-longcat/LongCat-DeepResearch, n.d., Meituan LongCat（t3） [来源链接](https://github.com/meituan-longcat/LongCat-DeepResearch)
- PaddlePaddle/ERNIE, n.d., Baidu（t3） [来源链接](https://github.com/PaddlePaddle/ERNIE)
- Claude Code: Best practices for agentic coding（2025 早期中文转载，历史对照）, 2025-07-22, Anthropic / 转载 [来源链接](https://cloud.tencent.com/developer/article/2545665)（历史对照；与现行版逐条反转）
- Claude Code 最佳实践 2025 版要点存档, 2025, 第三方存档 [来源链接](https://raw.githubusercontent.com/PeterHiroshi/anthropic-blog-study/main/claude-code-best-practices/README_CN.md)（第三方存档）

---

## 待完善事项

**审稿状态**：5 章均经明鉴秋 1 轮审查通过（PASS），无触发第 3 轮强制通过规则的章节，**无遗留审稿警告**。

**证据使用提示（阅读本报告时请一并保留）**：

1. **厂商自述不构成独立证据**：claude.dev（2026-07-24）「删除 80%+ 系统提示词而编码评测无可测损失」未披露评测方法、数据集、样本量、控制条件与结果数据，仅作厂商实践方向旁证，不可用于论证「规则无用」或「约束消失」。
2. **一处已核实的引用错位**：Thoughtworks Technology Radar 早期条目把 arXiv:2601.20404 转述为「手写版本常优于 LLM 生成」，但该文全文无「LLM 生成 AGENTS.md」条件；arXiv:2602.11988 的 7% 组间差异亦不能支撑该转述。本报告不采信此二手结论。
3. **描述性统计不作规范依据**：arXiv:2510.21413、arXiv:2511.12884、arXiv:2610.31715、arXiv:2606.15828、arXiv:2611.02383 仅刻画现状或结构对齐，不含下游任务效果变量；其中「循证模板」的预设前提「类别越普遍＝越该写」未受效果检验。
4. **实验结论不可合并**：arXiv:2602.11988（成功率 / 成本）与 arXiv:2601.20404（wall-clock / token）因变量不同，不可互证互否；单预印本（如 arXiv:2610.30571、arXiv:2611.04367）须在本地 harness–model 组合复现。
5. **厂商空白不可反推**：Meta Muse、小红书 dots、t3 各厂商未找到关于 agent 指令文件的官方论述，属公开发布渠道覆盖范围结论，不代表其内部没有实践；检索空白随渠道覆盖而收缩（第 3 章已实证）。
6. **t1/t2/t3 分层以用户本次给定为准**：工作区 `llm厂商.md` 的旧分层（GEMINI 在 t1、写作 facebook muse）与本报告采用的分层（GEMINI 在 t0、meta muse 在 t1）不同，已按用户本次消息统一。

**通用提示**：本报告引用的 2026 年官方文档、厂商博客与预印本时效性较强，重要决策场景下请二次核验链接可达性、版本与结论是否仍然成立。

---

> 本报告由 AI 深度研究团队生成，重要决策请经专业人员核验。所有引用来源请用户在重要场景下二次核验时效性与真实性。
