# AGENTS.md 厂商样本索引

采集口径：2026-11-07，GitHub code search `filename:AGENTS.md|CLAUDE.md|GEMINI.md org:<org>`，**只取仓库根目录**（子目录样本另标）。24 个 org → 219 个根目录命中 → 17 个 org 有命中 → 根目录样本 24 份 / 21 个仓库；另补**子目录样本 8 份**（第 6 节），合计 32 份。

命名：`{项目}.AGENTS.md`；子目录用 `{仓库}-{目录}.AGENTS.md`。原始文件叫 `CLAUDE.md`、`GEMINI.md` 的，本地保留原后缀。官方仓库的 3 份文本放在第 12 节附录，只作"规范滞后"的证据，不参与统计。

## 0. 两个评分怎么读

**置信度**——这份样本能不能代表 tier 表里点名的那个团队：

| 级别 | 含义 |
|---|---|
| **A** | 就是该厂商、该产品团队自己维护的 harness / agent 仓库，文件真实作用于它的日常开发 |
| **B** | 同厂商官方仓库，但不是点名的团队或产品（字节 deer-flow 非 Seed；Meta matrix 是 FAIR 研究基建；Anthropic 这份是插件市场而非 harness 本体） |
| **C** | 同厂商但非工程团队（DevRel / demo），只能作旁证 |

**价值**——作为写作素材的可借鉴程度，与置信度独立：●●● 值得整段引用 / ●● 有一两个可复用的点 / ● 只作反例或边角证据。

## 1. t0

| 文件 | 厂商 | 团队 / 主体 | 行 | 别名 | 置信度 | 价值 | 定位 |
|---|---|---|---|---|---|---|---|
| `claude-for-legal.CLAUDE.md` | Anthropic | Claude Code 插件市场（官方） | 130 | 仅 CLAUDE | B | ●● | 没有构建系统，规则 = 校验命令 + 不变量编号 I1–I11 + "这个文件看着放错了，别修" |
| `openai-agents-python.AGENTS.md` | OpenAI | Agents SDK | 106 | 软链 | A | ●●● | 通篇只有权威关系、技能路由、评审规则、安全清单；**没有项目简介，没有目录树** |
| `openai-python.AGENTS.md` | OpenAI | API SDK | 59 | 仅 AGENTS | A | ●● | 生成式仓库：先说"哪些文件是生成的、别手改"，剩下全是版本策略与验证命令 |
| `codex-code-mode-host.AGENTS.md` | OpenAI | Codex | 3 | 仅 AGENTS | A | ● | 全样本最短，一句话："测试里不要用 sleep 同步" |
| `codex-bottom-pane.AGENTS.md` | OpenAI | Codex | 12 | 仅 AGENTS | A | ●● | 目录级约束 + 自检动作，改完要核对文档只提到真实存在的 API |
| `dsh.AGENTS.md` | DeepSeek | Harness | 189 | 软链 | A | ●●● | 拿包清单当目录树，52 条外链，规则全部挂到 `docs/` 与 `.agents/notes/` |
| `gemini-cli.GEMINI.md` | Google | Gemini CLI | 96 | 仅 GEMINI | A | ● | 2025 传统式：概览 + 技术栈 + 目录 + 命令。**反例首选** |
| `ax.AGENTS.md` | Google | Agent Executor | 108 | 仅 AGENTS | A | ●●● | 六节：是什么 / 布局 / 构建测试 / 规则 / skills / DoD。薄地图的标准形态 |
| `adk-go.AGENTS.md` | Google | ADK | 431 | 1 行指针 | A | ●●● | 最长的一份；把多模块 Go 的静默失败写成可验证命令 |
| `adk-python.AGENTS.md` | Google | ADK | 30 | 仅 AGENTS | A | ●● | 30 行，一半在说"去读 `.agents/skills/<x>/SKILL.md`" |

## 2. t1

| 文件 | 厂商 | 团队 / 主体 | 行 | 别名 | 置信度 | 价值 | 定位 |
|---|---|---|---|---|---|---|---|
| `kimi-code.AGENTS.md` | Moonshot 月之暗面 | Kimi Code | 131 | 软链 | A | ●●● | 项目地图当架构说明；"Prompt text is behavior"把 `src/**/*.md` 列为契约文件 |
| `kimi-agent-rs.AGENTS.md` | Moonshot 月之暗面 | Kimi Agent（Rust） | 126 | 仅 AGENTS | A | ●● | 与 Python 版的同步契约：wire 标识、工具名、about 串必须一致 |
| `minimax-code.AGENTS.md` | MiniMax | MiniMax Code | 65 | 仅 AGENTS | A | ●● | 公开投影仓库；"单源真相表"（关注点 → 声明位置 → 消费方）7 行说清 |
| `zcode.AGENTS.md` | 智谱 Z.ai | ZCode | 85 | 仅 AGENTS | A | ●● | 中文；命令表 + "先改 spec 再改代码" + "不用 tool call 次数做硬停止" |
| `zcode-cli.AGENTS.md` | 智谱 Z.ai | ZCode CLI（子目录） | 103 | 子目录 | A | ●●● | 分层范本：开头声明"通用规则以根 AGENTS.md 为准，本文件只补 CLI 特有规则" |
| `qwen-code.AGENTS.md` | 阿里 | Qwen Code | 368 | 5 行指针 | A | ●●● | 把外部 PR 写成两级 triage：500 行以上硬拦，小改动要求"100% 确信否则升级" |
| `deer-flow.AGENTS.md` | 字节 | deer-flow（**非 Seed**） | 249 | 5 行指针 | B | ●● | 服务拓扑 + "根命令 vs 模块命令"，指向 `backend/`、`frontend/` 各自的 AGENTS.md |
| `matrix.CLAUDE.md` | Meta | FAIR 多智能体基建 | 60 | 仅 CLAUDE | B | ● | 传统概览式：概览 + extras 表 + 命令。Meta 系里唯一沾 agent 的样本 |

## 3. t2

| 文件 | 厂商 | 团队 / 主体 | 行 | 别名 | 置信度 | 价值 | 定位 |
|---|---|---|---|---|---|---|---|
| `mimo-code.AGENTS.md` | 小米 | MiMo | 80 | 仅 AGENTS | A | ●● | 约定式：只准 `bun ci`、已发布 migration 只增不改、文档测试一律用合成值 |
| `mimoagent.AGENTS.md` | 小米 | MiMo Agent | 118 | 软链 | A | ●● | 目录职责 + Python 工具链 + 依赖策略 + 运行时注意事项 |
| `avernet.AGENTS.md` | 蚂蚁 inclusionAI | Agent 基础设施 | 399 | **双份全文** | B | ●●● | 32 节；与 CLAUDE 版已分叉，是"双拷贝必然腐化"的实证 |
| `avernet.CLAUDE.md` | 蚂蚁 inclusionAI | 同上 | 361 | 同上 | B | ●●● | 23 节，结构完全不同；最后修改 10-19，AGENTS 已到 11-04 |
| `step-code.AGENTS.md` | 阶跃星辰 | Step-Code | 149 | 3 行 `@AGENTS.md` | A | ●● | 开篇即"禁止 AI 署名"，然后用分层依赖图约束改动方向 |

## 4. t3

| 文件 | 厂商 | 团队 / 主体 | 行 | 别名 | 置信度 | 价值 | 定位 |
|---|---|---|---|---|---|---|---|
| `customer-education-workshops.AGENTS.md` | xAI | 开发者教育（**非 Grok**） | 5 | 仅 AGENTS | C | ● | 5 行讲清作用域：每个 demo 自带 AGENTS.md，根 `.cursor/rules` 用 globs 限定 |

## 5. 结构骨架（前 6 个二级小节）

| 文件 | 骨架 | 代码块 | 外链 |
|---|---|---|---|
| `adk-go` | 概览 → skills → 命令 → DoD → 布局 → 约定 | 4 | 20 |
| `adk-python` | 概览 → 知识与架构 → 项目架构 → 开发环境 | 0 | 0 |
| `ax` | 是什么 → 布局 → 构建测试 → 规则 → skills → DoD | 2 | 1 |
| `avernet` | 真相源 → 概览 → 布局 → 模块职责 → 架构规则 → CI | 8 | 0 |
| `avernet.CLAUDE` | 仓库是什么 → 动架构前必读 → 总体架构 → 常用命令 → Git 流程 → 编辑纪律 | 12 | 0 |
| `claude-for-legal` | 布局 → 校验 → 约定 → cookbooks → 别动的地方 | 2 | 0 |
| `codex-code-mode-host` | （无小节，全文 3 行） | 0 | 0 |
| `codex-bottom-pane` | （无小节，全文 12 行） | 0 | 0 |
| `customer-education-workshops` | （无小节，全文 5 行） | 0 | 0 |
| `deer-flow` | 是什么 → 服务拓扑 → 仓库地图 → 根/模块命令 → 参考 → 横切约定 | 4 | 17 |
| `dsh` | 预发布 API → 布局 → 命令 → 密钥 → 约定 → 防御模式 | 2 | 52 |
| `gemini-cli` | 概览 → 构建运行 → 测试质量 → 开发约定 → 测试约定 → 文档 | 0 | 2 |
| `kimi-agent-rs` | 命令 → 目的与命名 → 同步契约 → 版本 → 重写约束 → 工作区 | 0 | 0 |
| `kimi-code` | 工作原则 → 项目地图 → 环境要求 → monorepo 维护 → 编码规则 → 实验特性 | 0 | 0 |
| `matrix` | 概览 → 开发 → extras → 命令 → 架构 → skills | 1 | 0 |
| `mimo-code` | 约定 → 风格 → 测试 | 2 | 0 |
| `mimoagent` | 是什么 → 架构 → 布局 → 工具链 → 依赖策略 → 开发流程 | 1 | 1 |
| `minimax-code` | 布局 → 生成文件 → 分支同步 → 单源真相 → 常见改动 → 验证 | 0 | 0 |
| `openai-agents-python` | 权威与流程 → 技能路由 → 范围兼容 → 评审规则 → 安全清单 → 文档 | 0 | 8 |
| `openai-python` | 生成 SDK → 安全 → 版本策略 → 改最低版本 → 自动化图 → 验证 | 1 | 6 |
| `qwen-code` | 工作原则 → 模块指令 → 命令 → 代码约定 → 开发准则 → 评审 | 8 | 2 |
| `step-code` | 署名 → 范围与阅读序 → 结构分层 → 概念归属 → 选目录 → 验证 | 1 | 20 |
| `zcode` | 核心原则 → 命令与结构 → 实现验证 → UI 边界 → 进程协议 → 日志 | 0 | 2 |
| `zcode-cli` | 工作规范 → 工具规范 → 跨平台 → 模块边界 → IO 边界 → 副作用契约 | 0 | 2 |

读法：**命令块少、外链多**的是"路由型"（dsh 52 链、step-code 20 链、adk-go 20 链）；**命令块多、外链零**的是"手册型"（avernet 8 块、qwen-code 8 块）。`gemini-cli` 最特殊：0 个代码块，命令全是行内代码——想被复制执行很难。

## 6. 子目录 AGENTS.md（8 份）

根文件管全局，子目录文件管增量。这 8 份的共同点：**与根文件的重复率接近 0**（逐行比对，唯一例外 4%）。它们不解释项目、不重复命令，只写"这一层独有的不变量"。

| 文件 | 所属根 | 层级 | 行 | 与根重复 | 父子关系怎么写的 | 看点 |
|---|---|---|---|---|---|---|
| `dsh-agent-notes.AGENTS.md` | dsh | 3 级 | 7 | 0% | 指向 `docs/AGENTS.md` 与 `README.md` | 7 行只讲一件事：每写一条新 Note 都要做 supersession 检查，同 PR 归档旧的 |
| `step-code-agent-core.AGENTS.md` | step-code | 2 级 | 26 | 0% | "Inherits the [root rules]" | 三段式：职责与入口 / 边界与不变量 / 验证；边界清一色是"不要引入什么" |
| `zcode-ui.AGENTS.md` | zcode | 2 级 | 23 | 0% | "本文件补充[根规则]" | 中文；平台操作必须走 `usePlatform()`，不许直接调 `window.zcode` |
| `kimi-code-minidb.AGENTS.md` | kimi-code | 3 级 | 27 | 0% | 不声明，直接进本地机制 | 4 段讲 4 个子系统，密度最高；一个字的项目介绍都没有 |
| `dsh-packages.AGENTS.md` | dsh | 2 级 | 29 | 0% | "supplement the repo-wide conventions" | 几乎每条规则都挂一个 `postmortem` 链接——事故驱动的物证 |
| `kimi-code-app.AGENTS.md` | kimi-code | 2 级 | 72 | 4% | "only contains rules local to …" | 入口链 + 目录职责表；明写"KimiTUI 只协调，不攒业务规则" |
| `dsh-docs.AGENTS.md` | dsh | 2 级 | 77 | 0% | 引用技能与归档 Note | 一张 14 行的"分层归属表"：每类内容各有其家，放错层就是错 |
| `deer-flow-routers.AGENTS.md` | deer-flow | **4 级** | 123 | 0% | 不声明，标题即契约 | 每段一个接口的不变量 + 对应测试文件名；叶子级写法的极致 |

另有三份已在前面的章节：`zcode-cli`（子目录）、`codex-bottom-pane`、`codex-code-mode-host`（叶子）。

三条观察：

- **层级越深，越只写"不变量 + 测试"。** `deer-flow-routers` 123 行里没有一句流程说明，全是"这个接口必须保持什么"和"测试在哪"。根文件讲方向，叶子文件讲不能碰的地方。
- **父子关系有四种写法**：不声明（靠目录位置，dsh/minidb、deer-flow）、一句 `Inherits`、一句 `supplement`、显式声明"本文件只含本地规则"。四种都能工作，但显式声明的两份（step-code、kimi-code-app）最不容易被误读成第二份全局规则。
- **0% 重复是纪律，不是巧合。** 子文件重复父文件，等于给 agent 两份会各自漂移的真相——`avernet` 的双份全文已经演示了结局。

最深的链条来自 deer-flow：根 → `backend`（335 行）→ `harness/deerflow` → `agents`（56 行）→ `agents/memory`、`middlewares`、`task_continuity` 各一份。每一级只加本层约束，不重复上一级。

## 7. 跨仓库重复出现的硬规则

同一类规则在多份文件里反复出现，说明是踩过坑的地方，不是模板套话。

| 规则 | 出现的样本 | 原文 |
|---|---|---|
| 禁止 AI 署名 | step-code、kimi-code | "Do not add `Co-authored-by` or `Co-Authored-By` trailers for Claude, Claude Code, Codex, ChatGPT, OpenAI, Anthropic, or another AI client." / "do not add any co-author attribution, and do not reveal the identity of the agent" |
| 凭据不入库 | dsh、minimax-code、zcode、openai-python | "Never commit credentials." / "Do not commit account data, sessions, logs, credentials, or real user content" / "不在日志、示例或提交中写入凭据、真实用户数据和内部服务地址" |
| 生成文件勿手改 | minimax-code、openai-python | minimax 用三列表（文件 → 重生成命令 → 校验命令）；openai 用一节讲 schema / generator / 手写的归属 |
| 文档随代码同步 | codex-bottom-pane、ax、zcode | "改这两个状态机时同步更新模块文档" / "skills 先读后改" / "新功能、行为变化先更新 spec" |
| 单一真相源 | minimax-code、kimi-agent-rs、ax | minimax 7 行表：关注点 → 声明位置 → 消费方 |
| 就近 / 分层 | kimi-code、step-code、xai 工作坊 | "follow the nearest `AGENTS.md`" / "read the AGENTS.md files in that directory and its ancestors" / 根 `.cursor/rules` 用 globs 匹配 |

两条观察：

- **"禁止 AI 署名"只出现在自己把 agent 产出当产品卖的仓库**（step-code、kimi-code），SDK 类仓库一次都没提。
- **敢把"生成文件"标出来的仓库，正文才短得下来**（minimax-code 65 行、openai-python 59 行）——最大的噪音源被一条规则挡掉了。

## 8. 别名机制

21 个仓库里，9 个同时提供两种文件名，12 个只有一种：

| 做法 | 数量 | 样本 | 代价 |
|---|---|---|---|
| 软链（git mode `120000`） | 4 | kimi-code、dsh、openai-agents-python、mimoagent | 单一真相；Windows 需 `core.symlinks`，否则检出成 9 字节文本 |
| 一行指针 | 4 | adk-go（1 行）、step-code（3 行 `@AGENTS.md`）、qwen-code（5 行）、deer-flow（5 行） | 多一次跳转；`@AGENTS.md` 是 Claude Code 的 import 语法，别的 harness 不认 |
| 双份全文 | 1 | avernet | **已经漂移**：399 行 / 32 节 vs 361 行 / 23 节，最后修改 11-04 vs 10-19 |
| 只有一种 | 12 | 其余 | 另一家的 harness 直接读不到 |

`ax` 是个例外：正文写着"CLAUDE.md 和 GEMINI.md 是本文件的软链"，但仓库 tree 里只有 `AGENTS.md`——声明了机制，没落地。

## 9. 文本的年龄：谁在维护，谁在立碑

如果"规范滞后"，那实践侧应该相反：频繁改。查了每份文件的提交历史（截至 2026-11-08）：

| 文件 | 首次提交 | 末次提交 | 提交数 | 状态 |
|---|---|---|---|---|
| `dsh` | 2026-08-11\* | 2026-11-03 | 100+ | 每两周在改 |
| `deer-flow` | 2026-06-25 | 2026-10-29 | 58 | 活跃 |
| `kimi-code` | 2026-05-22 | 2026-11-06 | 50 | 活跃 |
| `openai-agents-python` | 2025-05-18 | 2026-11-06 | 46 | 跨 18 个月一直改 |
| `qwen-code` | 2026-03-20 | 2026-10-21 | 32 | 活跃 |
| `gemini-cli` | 2025-05-15 | 2026-04-08 | 29 | **停在 4 月** |
| `adk-python` | 2025-06-28 | 2026-10-06 | 23 | 活跃 |
| `avernet` | 2026-07-06 | 2026-11-04 | 22 | 活跃 |
| `adk-go` | 2026-06-16 | 2026-11-06 | 14 | 活跃 |
| `mimo-code` | 2026-06-10 | 2026-10-12 | 13 | 活跃 |
| `openai-python` | 2026-07-24 | 2026-11-06 | 9 | 活跃 |
| `codex-bottom-pane` | 2026-01-13 | 2026-05-27 | 5 | 停更 5 个月 |
| `kimi-agent-rs` | 2026-02-04 | 2026-02-06 | 4 | 只在重写期改过 |
| `step-code` | 2026-09-23 | 2026-11-02 | 4 | 新仓库 |
| `zcode` | 2026-09-20 | 2026-10-30 | 3 | 新仓库 |
| `ax` | 2026-10-29 | 2026-10-30 | 2 | 十天前才出生 |
| `minimax-code` | 2026-09-18 | 2026-09-18 | 2 | 一次性 |
| `mimoagent` | 2026-09-21 | 2026-10-18 | 2 | 一次性 |
| `claude-for-legal` | 2026-05-15 | 2026-05-15 | 1 | 一次性 |
| `matrix` | 2026-05-08 | 2026-05-08 | 1 | 一次性 |
| `customer-education-workshops` | 2026-10-29 | 2026-10-29 | 1 | 一次性 |
| **官方 `agents.md`** | **2025-08-19** | **2025-08-19** | **1** | **出生即冻结** |

\* dsh 命中 100 次上限，首提时间只能当下界。

中位数 11 次；8 份 ≥20 次；14 份在最近 30 天内改过；4 份只有 1 次提交。

三条观察：

- **不是"规范落后于实践"，是"规范只在撞墙时更新"。** 高频的那几份（dsh、deer-flow、kimi-code、agents-python）背后都有按周发版的 harness；每一条新规则几乎都能在提交信息里找到对应的事故。低提交数的不是"稳定"，是"没人拿它干活"。
- **官方那份 1 次提交、15 个月不动，不是因为格式稳定了，是因为它不参与任何开发循环。** 同一时期 README 换了 logo、章程通过、站点在更新——只有规范文件没有维护者。
- **连"反例"都是活的，只是死得早。** `gemini-cli` 改了 29 次，停在 2026-04：团队曾经认真对待过，重心转移后就不再碰。判断一份 AGENTS.md 是否可信，看它最近三个月有没有被改过，比看它写了什么更有效。

## 10. 厂商覆盖与空白

| 厂商 | 组织 | 根目录命中 | 采用 | 最高置信度 | 说明 |
|---|---|---|---|---|---|
| Anthropic | anthropics | 25 | 1 | B | 官方插件市场多，harness 本体（claude-code）未开源 |
| OpenAI | openai | 31 | 4 | A | SDK 全线都有根文件；**codex 主仓库没有，只有叶子目录** |
| DeepSeek | deepseek-ai | 3 | 1 | A | harness 一份，子包各带一份 |
| Google | google / google-gemini / google-deepmind | 93 | 4 | A | 数量最多，但多为通用工程仓库；ADK 三语言各有独立文件 |
| Meta | meta-llama / facebookresearch | 0 / 12 | 1 | B | meta-llama 零命中；FAIR 有文件但非 harness |
| xAI | xai-org | 1 | 1 | C | grok-build（27.7k star）全仓库 0 个 |
| Moonshot | MoonshotAI | 3 | 2 | A | CLI 与 Rust agent server 各一份 |
| MiniMax | MiniMax-AI | 5 | 1 | A | code / cli / OpenAgentCore / MiniApps 都有根文件 |
| 智谱 | zai-org | 3 | 2 | A | ZCode 根 + 子目录，中文 |
| 阿里 | QwenLM | 8 | 1 | A | qwen-code 最厚，另有 Live-Harness、computer-use |
| 字节 | bytedance / ByteDance-Seed | 13 / 2 | 1 | B | Seed 只有训练框架 VeOmni；harness 在 deer-flow |
| 小米 | XiaomiMiMo | 1\* | 2 | A | 代码搜索漏了 mimoagent，靠直接列目录补上 |
| 蚂蚁 | inclusionAI | 16 | 2 | B | `ant-group` 零命中；Ling 模型团队无独立样本 |
| StepFun | stepfun-ai | 3 | 1 | A | Step-Code 一份 |
| 小红书 | FireRedTeam / rednote-machine-learning / RedNote | 0 | 0 | — | 唯一命中 `xiaohongshu-pub/open-notebook` 是 **fork**（17 个 CLAUDE.md 全由上游作者 lfnovo 提交），不算 |
| 美团 | meituan / meituan-longcat / Meituan-Dianping / Meituan-AutoML | 0 | 0 | — | 含 AI 项目 EvoCUA、LongCat-DeepResearch、LongCat-Next，均 0 |
| 百度 | baidu / PaddlePaddle / baidu-research / Baidu-AIP | 0 | 0 | — | 含 vLLM-ERNIE、Unlimited-OCR（30.9k★）、ERNIE、PaddleFormers，均 0 |

\* 代码搜索索引有滞后，命中数只能当下界看；判定"有没有"时以**递归 tree 查询**为准：

```bash
gh api "repos/{owner}/{repo}/git/trees/HEAD?recursive=1" \
  --jq '[.tree[].path | select(test("(^|/)(AGENTS|CLAUDE)\\.md$"))] | length'
```

t3 三家是按组织逐个仓库查的，不是只搜主组织：小红书查了 FireRedTeam、rednote-machine-learning（RedKnot 3.1k★）、RedNote；美团查了 meituan、meituan-longcat、Meituan-Dianping、Meituan-AutoML；百度查了 baidu、PaddlePaddle、baidu-research、Baidu-AIP。

**一个坑**：`xiaohongshu-pub/open-notebook` 有 17 个 `CLAUDE.md`，看着像小红书的实践——但它是 fork，提交者全是上游作者。**fork 里的文件不算这个组织的实践**，判定前先查 `fork` 字段。

**空白本身是结论**：t3 这三家不是"写得差"，是根本没有公开的指令文件。任何"某厂 AGENTS.md 长这样"的说法，仓库里查不到就是编的。

## 11. 来源

```
https://github.com/anthropics/claude-for-legal/blob/main/CLAUDE.md
https://github.com/openai/openai-agents-python/blob/main/AGENTS.md
https://github.com/openai/openai-python/blob/main/AGENTS.md
https://github.com/openai/codex/blob/main/codex-rs/code-mode-host/AGENTS.md
https://github.com/openai/codex/blob/main/codex-rs/tui/src/bottom_pane/AGENTS.md
https://github.com/deepseek-ai/deepseek-harness/blob/master/AGENTS.md
https://github.com/google-gemini/gemini-cli/blob/main/GEMINI.md
https://github.com/google/adk-go/blob/main/AGENTS.md
https://github.com/google/adk-python/blob/main/AGENTS.md
https://github.com/google/ax/blob/main/AGENTS.md
https://github.com/MoonshotAI/kimi-code/blob/main/AGENTS.md
https://github.com/MoonshotAI/kimi-agent-rs/blob/main/AGENTS.md
https://github.com/MiniMax-AI/minimax-code/blob/main/AGENTS.md
https://github.com/zai-org/ZCode/blob/main/AGENTS.md
https://github.com/zai-org/ZCode/blob/main/apps/zcode-cli/AGENTS.md
https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md
https://github.com/bytedance/deer-flow/blob/main/AGENTS.md
https://github.com/facebookresearch/matrix/blob/main/CLAUDE.md
https://github.com/XiaomiMiMo/MiMo-Code/blob/main/AGENTS.md
https://github.com/XiaomiMiMo/mimoagent/blob/mimo-oss/AGENTS.md
https://github.com/inclusionAI/Avernet/blob/dev/AGENTS.md
https://github.com/inclusionAI/Avernet/blob/dev/CLAUDE.md
https://github.com/stepfun-ai/Step-Code/blob/main/AGENTS.md
https://github.com/xai-org/customer-education-workshops/blob/main/AGENTS.md
https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/AGENTS.md
https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/AGENTS.md
https://github.com/deepseek-ai/deepseek-harness/blob/master/.agents/notes/AGENTS.md
https://github.com/MoonshotAI/kimi-code/blob/main/packages/minidb/AGENTS.md
https://github.com/MoonshotAI/kimi-code/blob/main/apps/kimi-code/AGENTS.md
https://github.com/bytedance/deer-flow/blob/main/backend/app/gateway/routers/AGENTS.md
https://github.com/stepfun-ai/Step-Code/blob/main/packages/agent-core/AGENTS.md
https://github.com/zai-org/ZCode/blob/main/packages/ui/AGENTS.md
```

## 12. 附录：官方文本（存档，不作论据）

`agentsmd/agents.md` 三份文件存本地，前缀 `agents.md-official.`：README 49 行、AGENTS.md 43 行、章程 8 节（纯治理）。

只记一条时间事实：`AGENTS.md` 自 2025-08-19 首次提交后再未修改。规范停在出生那天，实践没有。

## 13. 怎么用这批样本

- **先看第 9 节的年龄，再看内容。** 一份 AGENTS.md 的价值不在它写了什么，在它**多久没被改过**：三个月没动的，多半是立项时的宣言；每周在改的，才是这个团队真正在用的规则。
- **按"结构骨架"找参照，不按"厂商名气"找。** 要写"命令该怎么给"，看 avernet / qwen-code 的代码块；要写"规则该放哪"，看 ax / adk-python 的路由；要写"别名怎么处理"，看第 8 节。
- **把置信度当过滤器。** 引用前先看级别：B / C 的样本只能说"某厂商的某个仓库这么做"，不能说"某厂商这么做"。
- **负样本和正样本一样有用。** codex 无根文件、grok-build 零文件、t3 三家全空白——这三条能挡掉大量二手转述。
- **别学长度，学取舍。** 24 份中位数 107 行，10 份 ≤100 行；>300 行的 4 份（adk-go 431、avernet 399、qwen-code 368、deer-flow 249）全出自模型团队自己的仓库——它们厚，是因为要拦的坑具体且可验证。

活文档具体在改什么，见 [`分析-活文档在改什么.md`](分析-活文档在改什么.md)（基于 166 条提交信息）。重点样本的中译在 [`../samples-zh/`](../samples-zh/)，6 份，附术语表。
