# AGENTS.md

DeepSeek-Harness 是全插件式的 Cordis agent harness。改动 `packages/` 之前先读 [docs/architecture.md](docs/architecture.md)；文档工作遵循 [docs/AGENTS.md](docs/AGENTS.md)。

## 预发布 API 与已发布的 Session 数据

公开 API 处于预发布状态；每一处消费方都要同步更新。遵循[版本/状态](docs/session-format-status.md)与[类型确认](docs/cookbook/reviewing-persistence-type-changes.md)。[相邻迁移](.agents/notes/implemented/architecture/2026-08-31-released-session-format-migrations.md)可以新增一个以版本命名的后继，但绝不移动、覆盖或删除已提交的代次；前代既不意味着兜底，也不意味着支持降级。SQLite 使用单调递增的 `SCHEMA_VERSION`。

对[已声明的持久化类型变更](docs/cookbook/reviewing-persistence-type-changes.md)做确认。

每一处外部可感知的破坏性变更，立即记入[升级指南](.agents/skills/dsh-create-upgrade-guide/SKILL.md)。

**应用启动。** 只有 `dsh` profile 可以启动受支持的 Node 应用；包的 bin、demo，以及公开 SDK 的 argv 逃逸口都禁止（[规则](docs/architecture.md#application-launch)）。

## 仓库布局

```
vendor/      内置的 Cordis（vendor/README.md）
packages/    @deepseek-ai/dsh-<pkg> 工作区，位于 packages/<group>/<pkg>/
  core/                 agent/session API
  api/                  远程 BFF
  typert/               类型图
  llm/                  模型 provider
  shell/                命令执行
  subprocess/           子进程管理
  ssh/                  SSH 执行 provider
  terminal/             持久终端
  ptc-runtime/          PTC 执行
  sandbox/              进程隔离
  deliverables/         回合交付物
  fs/                   文件系统访问
  lsp/                  语言服务器
  skill/                技能加载
  web/                  搜索/抓取工具
  computer-use/         计算机交互
  browser-use/          浏览器交互
  compaction/           上下文压缩
  context/              请求上下文
  subagent/             委派 agent
  team/                 共享任务协调
  jobs/                 后台任务
  bundle/               profile 打包
  workflow/             工作流执行
  todo/                 todo_write 工具
  plan/                 带记录的规划
  goal/                 会话目标
  schedule/             定时跟进
  preset/               agent 组合
  guard/                循环/工具守卫
  extensions/           运行时自修改
  session/              持久会话
  session-query/        浏览/搜索/导出
  attachment/           二进制附件
  spill/                输出溢出
  storage/              非会话存储
  workspace/            工作区实体
  feedback/             人工反馈
  identity/             匿名身份
  settings/             用户设置
  credentials/          凭据/授权
  acp/                  仅供自动化的 ACP
  interaction/          人机交互
  boot/                 应用启动引导
  sdk/                  JSON-RPC SDK
  host/                 GUI 宿主
  client/               GUI 客户端
  mcp/                  外部工具
  experimental/         已评估、但不承诺产品支持的能力
  test-support/         测试基础设施
  util/                 零依赖工具
python/      Python SDK/运行时（python/README.md）
native/      @deepseek-ai/node-addon-system 源码（native/README.md）
benchmarks/  性能门禁
.agents/     Agent 工作流/笔记
docs/        文档（docs/AGENTS.md）
scripts/     门禁与生成器
website/     VitePress 文档投影
```

包分组见 [packages/README.md](packages/README.md)。[实验状态](packages/experimental/README.md#status)与"是否可选交付"无关；非实验包必须有产品使用场景，或明确的[归类](scripts/product-package-policy.ts)。

## 命令

```sh
pnpm install            # pnpm 工作区，node >=24.9
pnpm run clean           # 删除构建产物，并安全清理已删包留下的残留
pnpm run test           # 单元测试
pnpm run test:coverage  # CI 覆盖率门禁：packages/*/*/src 逐文件 100%
pnpm run test:e2e       # 真实 API 测试；没有 DEEPSEEK_API_KEY 时自动跳过
pnpm run test:expected  # 各 owner 本地的进程期望
pnpm run test:snapshot  # 无需 key，用录制会话重放已发布 profile；过滤：-t <name>
pnpm run test:snapshot:record  # 重新录制期望输出（需要 key）
pnpm run typecheck
pnpm run lint
pnpm run duplication    # 跨文件 TypeScript 克隆检测
pnpm run build          # build:artifacts + build:check，并发
pnpm run build:artifacts # 产物：tsc 输出 lib/types，tsdown 打包 runtime 与 Web
pnpm run build:check    # 用当前 build:artifacts 的输出，对 tests/scripts 做类型检查汇总
pnpm run hygiene        # publint + 工作区/包/依赖检查 + NodeNext 消费者检查
pnpm run check:windows-wine  # 只在诊断已知的 Windows 失败时用（需要 wine）；这个信号归 CI
pnpm run doc-sync       # 文档门禁（scripts/run-gates.ts）
pnpm run test:docs      # 快速文档检查（不构建；doc-quick 汇总）
pnpm run website:build  # VitePress 构建（同时充当死链检查）
pnpm dsh --profile headless "task"  # 从源码跑一个任务（需要 DEEPSEEK_API_KEY）
pnpm run demo:ptc -- "task"  # 无头 PTC 模式（需要 key）
pnpm run dev:web | dev:desktop  # 先 build:artifacts 再启动；Web 端还会在编辑时重建客户端产物。start:web | start:desktop 跳过构建
make web|dev-web|desktop|dev-desktop|build  # 同样的命令；ARGS='--no-open' 转发选项
```

### 宿主沙箱失败

如果必须的 `gh`、`pnpm`、构建、测试或生成器命令失败，原因是沙箱挡住了凭据、网络、IPC、文件监听或嵌套 `sandbox-exec`，那就原样重试，只申请最窄的宿主侧提权。必须拿出沙箱证据；绝不绕过测试失败，也绝不绕过产品沙箱。

### 在本地跑相关检查

推送之前，遵循 [dsh-pre-push-checks](.agents/skills/dsh-pre-push-checks/SKILL.md)；只报告实际跑过的命令。`gh stack sync` 之后立即验证；检查通过前不要合并。

- 证据要与改动面匹配：聚焦的行为测试、模型/用户输出快照、文档用 `doc-sync`、发布路径用构建产物冒烟、provider 用真实 API 的 e2e。
- 提交或推送时，绝不默认跑全量，也不要重跑已通过的检查。穷尽覆盖与平台矩阵归 CI；只有在明确要求、诊断 CI、或改动无法避免地波及全仓时，才在本地全跑一遍。
- CI 的覆盖率门禁是 `test:coverage`，不是 `test`（[原因](docs/testing.md)）。
- 开发时用 `build:artifacts`；宣布完成之前，在最新的 `build:artifacts` 之后跑 `build:check`，或直接跑 `build`（[布局](docs/development.md#typescript-project-layout)）。
- **浏览器自动化与 GIF 录制：** 用 `pnpm dsh web --patch apps/web/tests/pin-browse-picker.overlay.yml` 启动，以使用[页内目录选择器](apps/web/tests/pin-browse-picker.overlay.yml)；只有明确要测原生选择器行为时才去掉这个覆盖。

## 密钥 / .env

Windows 打包/签名：[必读](apps/desktop/windows-signing/README.md)。

真实 API 的测试/demo 读取 `DEEPSEEK_API_KEY`、可选的 `DEEPSEEK_BASE_URL`，以及根目录 `.env`。cordis.yml 在插件的 `config` 与入口的 `disabled` 下允许 `!!js`（永远不是 `!js`）；其他元数据保持字面量，所以条件组合也要用 overlay（[入门](docs/cordis-primer.md#loader-configuration)）。绝不提交凭据。没有 key 时 CI e2e 自动跳过；key 策略以 [testing.md](docs/testing.md) 为准。

## 约定

- 包名用 `@deepseek-ai/dsh-<name>`；vendor 已[改写作用域](docs/rescope.md)且为 `private: true`。Harness 包在 `peerDependencies`/`devDependencies` 里声明 `@deepseek-ai/cordis`。工作区依赖段：DSH 包用 `workspace:*`，vendor/native 用 `workspace:~`（[规则](.agents/notes/implemented/process/2026-08-10-npm-release-sequences.md)）。
- 全线 ESM（`"type": "module"`）。跨包用包名，本地相对导入用 `.ts`。配置子进程在纯 Node 下跑构建出的 `lib/`；源码回归用各自声明的启动方式（[测试策略](docs/testing.md#test-subprocess-launch-modes)）。`dsh` CLI 的源码启动走 tsx 的纯 ESM hook（`node --import tsx/esm`）；它触达的模块必须保持 ESM（不得只有 CJS 导出）（[源码启动契约](.agents/notes/implemented/architecture/2026-07-29-dsh-source-launch-tsx-esm.md)）。Raw/Web 的 `cordis.yml` 裸插件必须出现在其解析器 manifest 的 `dependencies` 里；`verify-cordis-config` 强制执行。
- **注册即副作用**：每一处贡献都走 `ctx.effect()` / `ctx.on()`；注册表的 `register()` 返回 disposer，直接返回或经其文档化的注册句柄返回。
- **类型化事件用声明合并**与可合并扩展的 map。事件的 JSDoc 需要 `@mode` 与载荷的 `@param`。公开服务方法要记录参数与非 void 返回值。`SessionEventMap` 的成员在读取时是必填的，除非信封带 `ignorable: true`。只有当既有判别式无法阻止不安全解读时，才升 `SESSION_FORMAT_VERSION`；同版本内的变更要求读取端行为安全（[评审](.agents/notes/implemented/process/2026-10-08-session-reader-compatibility-review.md)）。
- **按判别标签做 switch。** 封闭联合以 `assertNever` 收尾；可合并扩展的联合落到一个有文档的 default。
- **瀑布式监听器必须调用 `next()`** 才能委派；不调用就返回会截断整条链（[语义](docs/cordis-primer.md#cordis-waterfall-semantics)）。
- **模型可见 ⟺ 已记录**：任何进入模型请求的内容，都必须能从会话日志重建；新增一个模型可见输入，就要新增一个会话事件。
- **会话投影：** 创建、扩展或评审投影，或把 fold 辅助函数迁移成投影时，遵循 [dsh-session-projection](.agents/skills/dsh-session-projection/SKILL.md)。
- **用插件，不改循环**：新行为挂在文档化的扩展点上；改 `agent-loop` 必须同步更新 docs/architecture.md。
- **一条能力接缝由 Service Definition / Service Provider / Consumer 三种角色构成。** 它是完整的，绝不可能只有一种角色；只有当角色独立演化时才拆开（[术语表](docs/glossary.md#capability-seam)）。
- **优先用有人维护的依赖，而不是自己手写**——前提是它真的能删掉自己拥有的代码与测试（[政策](.agents/notes/implemented/process/2026-07-26-dependencies-over-hand-rolling.md)）。
- **包边界上显式优于隐式**：默认值由所属实现里一个显式的 `resolve(request): Spec` 步骤完成，绝不是在 `run()` 里藏一个 `?? default`（`dsh-shell` 的 request/spec 拆分就是模板）。
- **插件里不写死可调参数**：随部署而变的选项，是可在 cordis.yml 里改、且经过校验的 `Config` 字段；`DEFAULT_*` 常量或测试钩子不算可配置。协议常量、外部规范与安全不变量保持固定。
- **配置错误要响**：自包含的在加载时失败，否则在最早期可解析点失败；绝不静默跳过缺失的引用对象。
- **跨边界的不透明 id 要打品牌**（`dsh-brand` 的 `Branded<B>`），绝不用裸 `string`。
- **同一进程内的类型化边界，信任 TypeScript。** 不要仅仅因为静态接口已经要求的取值，就加运行时校验、兜底行为或恶意输入测试；该校验的地方是：解析器/配置、入队、模型/工具 JSON、持久化/文件、worker、进程与线协议边界。
- **不新增到 `unknown` 的断言**（`as unknown` 或 `<unknown>`）。保持或减少既有的精确基线；替换时用类型化取值或校验（[规则](.agents/notes/implemented/process/2026-09-19-no-unknown-casts.md)）。
- **源码面与产物面，绝不混用。** 静态门禁与测试经 tsconfig `paths` 把工作区导入解析到 `src`，在干净的树上通过；消费构建产物 `lib/` 的门禁必须声明这一依赖（[布局](docs/development.md#typescript-project-layout)）。
- **编译面要显式。** 同时有 Host 与 Client 程序的包，暴露各自面的叶子配置和一个仅作 solution 的根；全仓程序以某个面的配置为种子，绝不用根 solution（[布局](docs/development.md#typescript-project-layout)）。
- **错误处理：** 用类型化、结构化的失败，以及本地化、可操作的报告。明确处理归属、遏制范围与失败保证；停止依赖工作，并显式报告不可信状态或未知结果。遵循 [dsh-error-handling](.agents/skills/dsh-error-handling/SKILL.md)。
- **空的 `catch` 必须点名错误**并说明为什么；它的 `try` 只放一条语句。
- **注释就地下笔。** 不复述代码，不扩写无关注释，不在没有本地需要时解释远处行为（[理由](.agents/skills/dsh-prose-standard/SKILL.md)）。
- **禁用 `prove` + `nance` 这一拼接**（[规则](.agents/notes/implemented/process/2026-08-26-ban-ambiguous-origin-label.md)）。
- **并列取值优先对称**；无法解释的不对称，通常意味着漏了一次提取。
- **测试描述行为，不裁决正确性。** 过时的行为连同它的测试一起改；在 PR 里说明原因。
- **只有需要长期留存的决策理由才写 Agent Note；** 机械性或局部改动免写，包括局部 UI 改动（[范围](.agents/notes/README.md#when-to-write-one)）。归档笔记是冻结的：绝不编辑，也不当作当前权威（[归档政策](.agents/notes/README.md#archiving-and-deletion)）。
- **官方插件图标：** 正式与正式实验性插件的图标，用 [dsh-official-plugin-artwork](.agents/skills/dsh-official-plugin-artwork/SKILL.md)；新图形落地前要拿到设计师批准。
- **客户端 UI 文案归语言包所有。** 产品文案经类型化字典与 `t` 或本地化的原语属性输出；`verify-client-ui-i18n` 拒绝硬编码文案（[决策](.agents/notes/implemented/architecture/2026-08-23-locale-owned-client-ui-copy.md)）。英文文案（含插件标题）用句首大写（[规则](.agents/skills/dsh-prose-standard/SKILL.md#required-coverage-by-prose-location)）。
- **UI 改动：** 以 [dsh-client-ui-ux](.agents/skills/dsh-client-ui-ux/SKILL.md) 作为设计与评审入口；只在需要时加载它的参考材料。
- **测试政策**——[docs/testing.md](docs/testing.md)。每一处非平凡的、模型或产品用户可见的改动，都要更新一个无需 key 的录制会话快照；[快照归属](snapshots/AGENTS.md)规定顶层树留给会话驱动的用例，其他期望输出归各 owner 本地。夹具在 macOS/Linux 上重放；修夹具，不要修 normalizer。
- **每个工具的 UI 呈现要提前设计。** Host 侧的 presenter 保持纯函数；Web 卡片从原始事件与持久化的结果元数据派生（[cookbook](docs/cookbook/adding-a-tool.md)）。
- **为能力接缝、生命周期路径与转录输出规划单元、e2e、快照覆盖**；缺失的快照 harness 支持，在同一次改动里补上。
- **两个 SDK 都投影同一套循环。** agent-loop、会话生命周期与 `SessionEventMap` 的改动，要在同一个 PR 里更新 TypeScript 与 Python SDK 的期望输出；`pnpm run test` 两者都不覆盖（[覆盖面](docs/testing.md#when-a-snapshot-test-is-required)）。
- **刻意选择 PR 历史。** 独立改动拆开；在扩散之前先修引入问题的那个 PR。独立分支/栈式分支可以 merge-forward 或 rebase。改写用 `--force-with-lease`，远端有变动就中止，绝不用裸 `--force`；换到更新的 base 之前，保留进行中的 merge-forward 检查点（[理由](.agents/notes/implemented/process/2026-08-02-native-github-stacks-and-optional-rebases.md)）。
- **标签：** 每个 PR 一个 `kind/*`、所有实质性的 `area/*`，外加原生 Issue Type（[分类](.agents/notes/implemented/process/2026-08-08-unified-github-label-taxonomy.md)）。
- **PR 描述：** 每个 PR 描述都用 [dsh-pr-description](.agents/skills/dsh-pr-description/SKILL.md) 撰写与更新。
- TODO 标记：按紧急度用 `FIXME`/`TODO`/`XXX`（[语义](docs/development.md)）。
- 文件结尾恰好一个换行；`git diff --cached --check`（pre-commit）把关。

## 防御性模式

改动包表面、远程服务、客户端组件或状态、对话或 Session 行为，以及生命周期、并发、子进程、拆卸代码之前，先读 [docs/defensive-patterns/README.md](docs/defensive-patterns/README.md)。其中的[评审原则](docs/defensive-patterns/README.md#why-these-rules)按产物逐项适用。

## 类型安全与文档

全部代码在 `strict: true` + `noImplicitAny` 下编译；每一处残留的 `any` 都要说明为什么无法收窄。每个模块与导出都有简洁的 JSDoc，讲清不显然的契约；函数类导出含 `@param`/`@returns`，由 `verify-export-jsdoc` 强制。继承声明的成员、插件协议槽位与构造函数，文档留在声明它的 Service Definition、协议或类上。

注释与文档陈述完整的契约与上下文，不是推理过程的转录。用直接、具体的词。不要用比喻。写 `contract`、`boundary`、`shape` 之前，先问有没有更准确的词：写 `response fields`、`JSON validation`、`ESM exports`，而不是 `response shape`、`validation boundary`、`module shape`。`contract` 只留给前置条件、后置条件、不变量、兼容承诺，以及其他调用方、被调方、实现方、provider、产出方或消费方所依赖的义务。字面意义上的进程、线协议、安全、事务、生命周期边界，可以保留这个说法。不要叙述控制流或测试，不要保留评审历史，不要复述代码。保留行为、失败、时序、归属与安全使用方面的事实；理由用链接。决策时用 [dsh-prose-standard](.agents/skills/dsh-prose-standard/SKILL.md)。把可机械检查的不变量接进一个会执行的上层门禁，并证明每条被改动的验收路径确实会拒绝非法用例。用范围窄、有理由的例外，而不是全局关掉一条规则。

文档随每次代码改动一起走：受影响的 README 与 JSDoc 契约同时更新。常规双语工作遵循 [docs/AGENTS.md](docs/AGENTS.md)；只有用户明确调用才能跑 `dsh-translate-docs`。现状式行文、一段一物理行、一事一处、字数预算，都在那里规定。

## 编辑这份指令

`CLAUDE.md` 在根目录与 `packages/` 下都是 `AGENTS.md` 的符号链接；改真文件。每条规则自身要完整，同时链接到高层文档。清晰度不受损就压缩；内容确实需要更多空间时，上调 `verify-doc-budgets` 的上限。

## Vendor 政策

`vendor/` 下是钉死版本的源码副本（上游 SHA 清单在 [vendor/README.md](vendor/README.md)）。按那里的同步流程更新；重新应用或清退已记录的本地修改；重跑 `pnpm run test && pnpm run build`。
