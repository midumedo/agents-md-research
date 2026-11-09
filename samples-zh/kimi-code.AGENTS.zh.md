# 仓库级 Agent 指南

用与用户相同的语言回复。

这是一个为 agent 辅助开发而构建的 TypeScript monorepo。根 `AGENTS.md` 只放热路径规则：项目地图、硬约束、流程要求——每项任务都必须知道的东西。

## 工作原则

- 从第一性原理出发。以真实需求、代码事实和验证结果为准；目标不清楚时，先与用户讨论。
- 以代码而非文档为事实来源。除非用户明确要求，不要为了理解实现去读普通 Markdown。
- 改代码之前，先读相关代码和最近的约束，并遵循目录树上最近的 `AGENTS.md`。
- 改动保持聚焦。不要顺手夹带无关重构。
- 提交时不要加任何共同作者署名，也不要在提交信息、PR 描述或任何说明文字里暴露 agent 身份。

## 项目地图

- `apps/kimi-code`：CLI / TUI 应用。通过 `@moonshot-ai/kimi-code-sdk` 使用核心能力，不得直接依赖引擎包。编写或修改终端 UI 时，使用 `write-tui` 技能（`.agents/skills/write-tui/SKILL.md`）。
- 浏览器 Web UI：**源码已不在本仓库。** 它在 code-app 仓库（`apps/web`）里开发，以提交进来的预构建产物 `apps/kimi-code/dist-web`（被 gitignore、强制入库）形式发布，用 `KIMI_CODE_REPO=<本检出> pnpm run sync:web` 从 code-app 同步——只要 Web UI 需要以不同方式发布，就在同一次改动里同步并提交该产物。`apps/kimi-code/scripts/check-web-assets.mjs` 会在打包时检查产物是否缺失。若要对本仓库的服务端改 Web UI，在本仓库跑 `pnpm dev:server`，再用 `KIMI_SERVER_URL` 把 code-app 的 `pnpm dev:web` 指过来。
- `apps/vis`、`apps/vis/server`、`apps/vis/web`：会话与回放的视觉调试工具。
- `packages/agent-core-v2`：DI × Scope 的 agent 引擎（kap-server 背后的 v2 移植）。三层 `LifecycleScope`——`App` / `Session` / `Agent`（`app/scopes.ts`）——加上 L3 单元层（`Service`/`Fiber` 单元、集合贡献点、`src/features/` 里的 Feature 接缝）；`WorkspaceInstance`/`Program` 图在运行时经 `IWorkspaceInstanceManager` 实例化，App 作用域的 `ISessionManager` 通过 `Program` 为每个工作区创建会话控制器。
- `packages/node-sdk`：公开的 TypeScript SDK 与 harness。
- `packages/oauth`：Kimi OAuth 与托管鉴权工具。
- `packages/telemetry`：共享的客户端遥测基础设施。
- `packages/session-import`：独立的 Node.js 工具，把外部会话转换成一条用户消息，携带整段对话、完整的有效历史附件与图片引用。它不依赖引擎或 UI；调用方提供目标会话的附件目录与上下文预算。
- `packages/kap-server`：Kimi Code 服务端，基于 `@moonshot-ai/agent-core-v2`；以 REST + WebSocket（`/api/v1` + `/api/v1/ws`）暴露会话，另有 `/api/v1/debug/*` 反射 RPC 面（`--debug-endpoints`，绑定回环 + bearer 鉴权）。一个 agent 的对话（turns 与 items）是对其持久化线记录的折叠（`src/services/conversation/`），以 `turns` / `turn.*` / `item.*` 帧流式输出。
- `packages/remote-control`：Kimi Remote Control 隧道客户端——把本机注册到中继，并把 HTTP/WebSocket 流量转发给本地服务端，带全机单实例锁；由 kap-server（`/api/v1/remote-control` 开关）与 CLI（`kimi web --remote-control`）使用。
- `packages/agent-core-v2/src/remote`：引擎内的远程执行栈——本地 `ExecBridge`（ssh/docker/`command` 启动器）与轻量 `kimi exec-server` 执行器之间自包含的 NDJSON 行帧 RPC（codex exec-server 方言），加上把已声明目标注册为 `Environment` 的 provider（fs/process 的 RPC 桩是 `src/os/backends/remote` 下的 OS 后端，与 `node-local` 同级）；以 `@moonshot-ai/agent-core-v2/remote` 与 `/remote/server` 导出。执行器保持轻量——`src/remote/server` 只能 import node 内建、相对模块、`#/remote/protocol`、`#/os/interface`、`#/_base/execEnv`，以及它所适配的无 DI 的 node-local fs/process 后端（由该包的 `lint:imports` 强制）；CLI 只能通过 SDK 的轻量子路径 `@moonshot-ai/kimi-code-sdk/exec-server` 到达它，并在 `src/cli/exec-server.ts` 中动态加载。
- `packages/klient`：客户端 SDK——agent-core-v2 之上契约驱动的门面（`global.*` / `session(id).*` / `agent(id).*`，zod 校验）；传输经子路径入口（`@moonshot-ai/klient/ipc|memory`，两者返回同一个 `Klient`）；e2e 套件也放在这里。见 `packages/klient/AGENTS.md`。
- `packages/tree-sitter-bash`：纯 TypeScript 的 bash 解析器（无运行时依赖、无 wasm）；`parse(source, { timeoutMs, maxNodes })` 在确定性预算下运行，返回可判别的 `ParseResult`——调用方必须把 aborted/hasError 的树当作"无法分析"并降级。只做解析，不做安全判断；见该包 README 的"Known differences"。
- `packages/minidb`：kap-server 搜索索引背后的嵌入式 JSON 文档存储（`MiniDb`）——快照 + WAL 持久化、独占写锁、可超出内存的全文本层、持久化索引代次。见 `packages/minidb/AGENTS.md`。

## 环境要求

- **Node.js**：`>=24.15.0`（来自根 `package.json` 的 `engines`；`.nvmrc` 为 `24.15.0`，供 nvm / fnm / mise 选择最低推荐版本）。
- **pnpm**：`10.33.0`（来自根 `package.json` 的 `packageManager`）。
- Node 版本不满足时 `pnpm install` 会直接失败，因为 `.npmrc` 设了 `engine-strict=true`。

## Monorepo 工作区维护

- `pnpm-workspace.yaml` 是工作区成员的事实来源，但 `flake.nix` 里另有一份**硬编码**的 `workspacePaths` 与 `workspaceNames` 列表。
- **新增或删除工作区包时，必须同时更新 `pnpm-workspace.yaml` 与 `flake.nix`——每一个包都要，包括没有任何东西依赖的叶子包、测试包、e2e 包。**
  - `pnpm-workspace.yaml` 用 glob（`packages/*`、`apps/*`），多数包会自动落在里面；`flake.nix` 全手工，漏就漏在这里。
  - `flake.nix` 的 `workspacePaths` 漏一条，Nix 构建的 `src` 文件集就会静默丢文件。
  - `flake.nix` 的 `workspaceNames` 漏一个，`pnpmConfigHook` 就会坏，因为该工作区的依赖不会被拉取。
- 自动化的"Check flake.nix workspace sync"（`scripts/check-nix-workspace.mjs`）只校验 `@moonshot-ai/kimi-code` 的传递依赖**闭包**。闭包之外的叶子包（例如没人 import 的 e2e 包）即使漏在 `flake.nix` 外也能通过检查。所以检查变绿并不能证明 `flake.nix` 已同步——每次增删都手工维护，别指望这道检查兜底。

## 通用编码规则

- `packages/agent-core-v2` 与 `packages/kap-server` 是**无注释区**：任何注释都不许有——行注释、块注释、JSDoc（导出的符号也不例外）；唯一例外是承重的 lint 抑制指令（`oxlint-disable` / `eslint-disable`），其余工具指令（`@ts-expect-error` 等）仍然禁止。由 `scripts/check-no-comments.mjs` 对 `src/`/`test/`/`scripts/` 下的 `.ts`/`.tsx`/`.mts`/`.mjs` 强制，随 `pnpm lint` 一起跑。
- 可选对象属性，直接传 `undefined`，不要用条件展开。
  - 对：`{ user }`
  - 错：`{ ...(user ? { user } : undefined) }`
- 可选对象属性不必在类型上再额外允许 `undefined`。
  - 对：`interface Options { user?: User }`
  - 错：`interface Options { user?: User | undefined }`
- 只有一个参数的内部方法，不要为了风格统一硬改成 options 对象。
- 函数只按抽象层级拆分：每个函数读起来是一个层级的叙述（逐层下降原则）；不引入新抽象层的包装——尤其只有一个调用点的——直接内联，不要抽出来。
- 除包的 `index.ts` 外，其他 `index.ts` 优先写 `export * from './module';`。
- 不要新增太多测试文件。优先把测试加到对应组件或模块已有的测试文件里。
- 测试因用户改动而失败时，默认先修测试；不要为了让旧测试通过而改实现，除非实现确实有 bug。
- 不要为外部兼容牺牲代码质量，除非用户明确要求。破坏性变更走 changeset 与 `major` 版本号，并受下面那条规则约束。

## 实验性特性

- 尚未公开的特性，用实验开关挡住。开关由环境变量驱动，默认关闭：`KIMI_CODE_EXPERIMENTAL_<NAME>` 控制单个，`KIMI_CODE_EXPERIMENTAL_FLAG` 全开。优先级为：单项环境变量 > `[experimental]` 配置 > 全局环境变量 > 开关自身的 `default`。发布时把该项的 `default` 翻成 `true`。
  - `packages/agent-core-v2` 与 kap-server 的模块：没有中央目录——在所属领域 import 时用 `registerFlagDefinition` 声明，然后用 `IFlagService.enabled(id)` 检查。

## 该在哪里更新指令

- 影响几乎所有任务的硬规则：更新根 `AGENTS.md`。
- 只影响某个目录的规则：更新最近的子目录 `AGENTS.md`。
- 项目地图条目保持 1–2 句；深入的包文档放在该包自己的 `AGENTS.md` 里。
- 指令更新要聚焦，且有代码事实支撑。

## 发布流程

- `main` 累积 changeset；`release/*` 维护 `ci: release packages` 的 PR。合并对应的版本 PR，即授权在其合并提交的 CI 成功后发布。普通推送不会发布。
- npm 发布与原生产物沿用既有的 Changesets 流程。CDN 上传与路由仍是单独审批的操作。迁移与主线同步见 `.changeset/README.md`。

## 流程要求

- 读代码时优先用 `rg` / `rg --files`。
- 设计改动时，先沿用既有的边界与本地模式。
- 公开文本与测试数据中，用 `example.com`、`example.test`、`YOUR_API_KEY` 这类中性占位符替换真实的内部标识。开 PR 之前，请一个只读 agent 审一遍 diff 里的上下文相关内部标识。
- 创建 PR 时，用 `write-pr` 技能（`.agents/skills/write-pr/SKILL.md`）写 PR 描述。标题遵循 Conventional Commit 风格，例如 `chore: remove legacy format commands`。
- AI agent 创建或更新 PR 时，填好 `.github/pull_request_template.md`——链接相关 issue 或说明问题，再描述改了什么。不要留占位文字，也不要提交一份泛泛的 diff 摘要。
- 不要提交含糊的 AI 生成 PR 文本。人类作者必须理解到能讲清代码、边界情况和为什么这个方案适合本仓库。
- 完成任务、提交 PR 之前，必须跑 `gen-changesets` 技能（见 `.agents/skills/gen-changesets/SKILL.md`），按它的规则在 `.changeset/` 下生成 changeset。
- Changeset 必须严格遵守 `.agents/skills/gen-changesets/SKILL.md` 的规则：写一句面向用户的短句，只说改了什么；用户感知不到的变化跳过。
- 生成 changeset 时，**绝不**自行决定 `major` 升级——停下来，解释清楚，取得用户明确确认；默认 `minor`，退而用 `patch`。见 `.agents/skills/gen-changesets/SKILL.md`。
- 优先用 `import ... from '#/...'`，它与 `import ... from '@/...'` 等价。
- 不要提交一次性的临时文件或探索性文件。绝不入库：
  - Agent 工作笔记、交接/总结文档（例如 `HANDOVER-*.md`、`HANDOFF-*.md`、`handoff.md`）。
  - 一次性的 UI/UX 原型或设计稿（例如 `*-designs.html`、`*-mockup.html`、`*-demo(s).html`），无论在仓库根还是 `design/` 目录下。唯一该入库的 `.html` 是 Vite 的 `index.html` 入口。
  提交或开 PR 之前，跑 `git status` 与 `git diff --staged --stat`，删掉符合上述模式的东西。临时工作放到 `.tmp/`（已 gitignore），不要放仓库根或源码树。

## 代码评审规则

以下规则适用于每一次 PR 评审，无论人工还是自动。它们引用的用户群体与契约文件列在 `.agents/skills/review-pr/surfaces.md`。

### 评审范围：PR 的 diff，不是整个仓库

- 只评审本 PR 相对其 base 的 diff。PR 未触及的代码里已有的问题（base 上就存在），作为非阻塞备注提出，绝不作为阻塞问题项——唯一例外是 PR 的行为直接触发或放大了那段旧代码。
- 一类问题一次报完：当某个问题项可推广到 diff 里的多处，一次列全，不要分几轮陆续提。
- 不报纯风格或个人偏好问题。每条问题项都带严重度：P0 阻塞 / P1 应修 / P2 建议。
- 复审是增量的：只评审自上次评审以来的变化；不要复述未改动代码上的结论，也不要重提已解决的讨论串。把某个已解决的讨论串当作已了结之前，先核对当前代码：只有问题确实修好、或结论已不成立，才压下这条问题项。
- 领域专属规则各有其权威出处——changeset（`.agents/skills/gen-changesets/SKILL.md`）、无注释区（通用编码规则）、PR 文本（`write-pr` 技能）；按那些规则评审，不按临时口味。

### 枚举被改变的行为，而不只是找 bug

改动之前能用的任何输入——配置键、环境变量、CLI 参数、provider 响应形状、旧版本写入的会话、客户端请求、hook 载荷——改动之后必须表现一致，除非 PR 声明了该变化。对每一个被删除或收窄的分支、条件、默认值或提示词句子，问一句：以前谁会走到这里，他们现在去哪；对每一个新增条件，问：哪些既有输入现在会先命中它。PR 把某条路径称作"未变"、而它的分支条件其实移动过时，要标出来。

### 翻转默认值或删除行为，必须写明损失并留逃生口

所有处在旧默认值上的人都会受影响。要求 changeset 写明用户失去的是什么，而不只是新默认值；要求提供配置、环境变量或开关形式的逃生口，或维护者在 PR 中的明确签字；并要求一条测试，为继续使用旧行为的群体钉住旧行为。

### 提示词文本就是行为

编辑或删除 `packages/agent-core-v2/src/**/*.md` 下的句子（系统提示词、工具描述、提醒、覆盖层、内置技能），会改变每一个收到该提示词的用户的 agent 行为。"没有测试引用这句话"不是没有影响的证据。要求 PR 写明谁会收到这段文本（每个会话、计划模式、Tower 这类开关控制的特性）、这句话在强制什么、谁依赖它、现在由什么来强制。

### 契约文件是绊线

`packages/agent-core-v2/docs/` 下的清单（`config-manifest.toml`、`wire-manifest.d.ts`、`state-manifest.d.ts`）、`packages/kap-server/test/__snapshots__/apiSurface.snapshot.test.ts.snap`、`packages/node-sdk/src/index.ts`、`packages/agent-core-v2/src/features/externalHooks/`、`packages/acp-server/` 以及 `apps/kimi-code/src/cli/` 在本仓库之外被消费：code-app 的桌面端与 Web 端、VS Code 扩展、Zed 这类 ACP 客户端、SDK 使用者、hook 脚本、无头输出解析器。它们变化时，要求 PR 写明消费方，以及上一版本的数据与客户端如何继续工作。

### 移植与重构，要带上旧路径的功能清单

当改动替换或绕开一条既有路径时，要求列出旧路径做过的所有事——认过的环境变量、兜底、接受的输入——以及每一项在新路径里的位置。静默丢掉一项是回归，不是清理。

### 静默失败比崩溃更严重

让产品静默忽略配置、静默批准或跳过某个动作、静默丢数据的改动，是最严重的问题项：用户拿不到任何信号去报告它。
