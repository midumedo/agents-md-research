# samples-zh：重点样本中译

选译 6 份，标准不是"写得好"，而是**形态互不重复**：

| 译文 | 原文 | 行 | 形态 |
|---|---|---|---|
| `ax.AGENTS.zh.md` | [`ax.AGENTS.md`](../samples/ax.AGENTS.md) | 108 | 薄地图：只有布局、门禁、硬规则、技能索引、完成定义 |
| `openai-agents-python.AGENTS.zh.md` | [`openai-agents-python.AGENTS.md`](../samples/openai-agents-python.AGENTS.md) | 106 | 流程型：没有项目介绍，全是权限、路由、评审、安全 |
| `kimi-code.AGENTS.zh.md` | [`kimi-code.AGENTS.md`](../samples/kimi-code.AGENTS.md) | 131 | 地图 + 契约：项目地图当架构说明，"提示词即行为" |
| `dsh.AGENTS.zh.md` | [`dsh.AGENTS.md`](../samples/dsh.AGENTS.md) | 189 | 路由型：规则全部挂到 `docs/` 与 `.agents/notes/` |
| `codex-bottom-pane.AGENTS.zh.md` | [`codex-bottom-pane.AGENTS.md`](../samples/codex-bottom-pane.AGENTS.md) | 12 | 叶子级：一个目录、一条约束、一个自检动作 |
| `codex-code-mode-host.AGENTS.zh.md` | [`codex-code-mode-host.AGENTS.md`](../samples/codex-code-mode-host.AGENTS.md) | 3 | 极简：一句话 |

## 翻译原则

1. **信优先。** 原文的条件、范围、例外一个不省；宁可中文略拗，也不替作者"理顺"逻辑。
2. **保留原节奏。** 原文用短句并列的地方不合并成长句；原文用长句套条件的地方不拆成清单。
3. **代码原样，注释译出。** 代码、路径、命令、标识符、配置键、专有名词一律不动；代码块里只作说明用的注释译成中文（见 `ax` 与 `dsh` 的目录树）。表格与标题层级一一对应。
4. **术语固定**，见下表；同一条规则在不同文件里用同一个译法，方便横向对照。

## 术语表

| 原文 | 译文 | 说明 |
|---|---|---|
| harness | harness | 不译。译作"框架"会与 framework 混淆 |
| skill / `SKILL.md` | 技能 / `SKILL.md` | 目录名与文件名不译 |
| changeset | 变更集 | |
| reconciler | reconciler | Kubernetes 语境，不译 |
| Control plane | 控制面 | |
| primitive | 原语 | |
| schema | schema | 不译，与代码里的标识符一致 |
| migration | 迁移 | |
| lockfile | 锁文件 | |
| symlink | 符号链接 | |
| fixture | 测试夹具 | |
| gate | 门禁 | 指 CI 检查或必须通过的关卡 |
| Definition of done | 完成定义 | |
| finding | 问题项 | 评审语境，指一条被指出的问题 |
| wire protocol | 线协议 | |
| tripwire | 绊线 | 原文比喻，保留 |
| shim | 垫片 | |
| oracle | 判据来源 | 测试语境，指独立于实现的正确性来源 |

## 注意

- 译文对应的是 2026-11-07 抓取的版本，原文后续可能已改。
- `dsh.AGENTS.zh.md` 与 `kimi-code.AGENTS.zh.md` 里大量 `[文字](路径)` 链接保持原样，指向的是原仓库内的相对路径，在本仓库点不开。

## 未译的

| 样本 | 为什么没译 |
|---|---|
| `adk-go`（431 行） | 太长，且三分之一在讲 Go 多模块工作区的具体坑，翻译收益低于直接读 |
| `qwen-code`（368 行） | 大半是 PR triage 与命令清单，结构与 `openai-agents-python` 重叠 |
| `avernet`（399 / 361 行） | 双份且已漂移，译一份会误导；要看得两份对照 |
| `zcode`、`zcode-cli` | 原文就是中文 |
| `mimo-code`、`mimoagent`、`adk-python`、`minimax-code` | 与已译四份同型（约定式 / 路由式），读者可对照原文 |

