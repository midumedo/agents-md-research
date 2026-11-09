# AX —— Agent 契约

本仓库所有编码 agent 的唯一契约。`CLAUDE.md` 与 `GEMINI.md` 都是指向本文件的符号链接。

本文件只保留每项任务都要用的内容：布局、CI 门禁、硬规则、技能索引。流程、背景与排障写在
`skills/<name>/SKILL.md` 里——动手改之前，先加载对应的那一份。

## 这是什么

面向沙箱化 agent 任务的声明式编排器，形态对标 Kubernetes，运行在
[Agent Substrate](https://github.com/agent-substrate/substrate) 上。三个原语——`Task`、`Workspace`、`Model`（`ax.io/v1alpha1`）——外加一个类 `kubectl` 的命令行。为什么这样设计见 `DESIGN.md`；动 reconciler 或 server 之前先读它。

## 布局

```
cmd/ax                 CLI（apply/get/describe/ssh/suspend/resume/delete/ctx/tunnel）
cmd/ax-server          控制面：gRPC API；直接创建、挂起、恢复、删除任务 actor
cmd/ax-task-runner     沙箱内的 PID 1；实例化 Workspace，运行 agent
internal/controller    TaskReconciler：任务挂起/恢复/删除在 Substrate 一侧的实现；reconciler_test.go 是测试模板
internal/server        gRPC 处理函数
internal/store/        内存 + postgres 两种后端（schema.sql 与 schema.go 必须同改）
internal/substrate     唯一与 Substrate Control API 通信的地方
internal/workspace     git clone / 内联文件 / skills 目录（MCP 仅存在于 spec；agent 经 metadata server 读取）
internal/{tunnel,guest,metadata}  k8s port-forward、沙箱内 guest 客户端、metadata server
pkg/apis/v1alpha1      ax.proto（事实来源）-> *.pb.go（生成物）、types.go、resources.go
test/e2e               基于 Kind；`go test ./...` 会编译它，但没有 AX_E2E=true 时自动跳过
hack/                  E2E 环境准备脚本
docs/                  manifests.md 是面向用户的 API 参考
skills/                开发者技能（SKILL.md 格式）。.agents/skills、.claude/skills、.gemini/skills 均符号链接至此。
```

## 构建 / 测试（已核对；CI 跑的就是这些）

```bash
export GOTOOLCHAIN=auto          # go.mod 要求 1.27.1；本地 Go 较旧时会自动下载
go mod tidy && git diff --exit-code go.mod go.sum
go build ./cmd/ax ./cmd/ax-server ./cmd/ax-task-runner
go test ./...                    # 约 20 秒，16 个包带测试；Postgres 与 E2E 层自动跳过
go vet ./... && gofmt -l .       # CI 不跑；HEAD 上两者都是干净的
```

可选层级：

- **Postgres store**：设置 `AX_TEST_POSTGRES_URL='postgres://ax:ax@localhost:5432/ax?sslmode=disable'`（CI 使用 `postgres:17-alpine`），再跑 `go test -count=1 ./internal/store/postgres/...`。
- **E2E**：`make test-e2e`（或 `make test-e2e E2E_ARGS='-test.run TestE2E_ModelCRUD'`；见 `test/e2e/README.md`）。

## 规则

以下每条都由代码、CI 或 `CONTRIBUTING.md` 强制执行；标注的位置就是该去查的地方。怎么做写在技能里。

**A. 正确性**
- `ax.proto` 是事实来源；`*.pb.go` 是生成物（文件头有 `DO NOT EDIT`）。没有 `make generate` 目标。
- `internal/store/postgres/schema.go` 加载 `schema.sql`；两者必须同改。

**B. Agent Substrate**（细节：`/substrate`）
- `internal/substrate/client.go` 是唯一构造 `ateapipb.NewControlClient` 的非测试文件；新增 Control API 调用都放这里。
- `api.ate-system.svc`、`atenet-router.ate-system.svc` 和命名空间 `ate-system` 硬编码在 10 个文件里（清单见 `/substrate`）。
- Substrate 拒绝仅带 tag 的镜像；`spec.image` 必须以 digest 固定（`docs/manifests.md`、`TemplateCreationFailed`）。
- 只有当 `spec.debug` 为 true 时才提供 guest 服务（`ax ssh`）（`internal/metadata/server.go`、`cmd/ax/main.go`）。
- 单元测试跑进程内 fake：`mockControlServer`（`internal/controller/reconciler_test.go`）与 `internal/substrate/substratetest`（`internal/server/server_test.go`、`cmd/ax/main_test.go`）。
- `go.mod` 不含 `replace` 指令；E2E 按 `go.mod` 里的版本安装 Substrate（`hack/setup-e2e-kind.sh`）。

**C. API 表面**
- `Task`/`Workspace`/`Model` 的字段在 `ax.proto` 定义，在 `types.go` 映射，在 `resources.go` 校验，在 `docs/manifests.md` 记录。

**D. 卫生**
- `go.mod`/`go.sum` 不整洁时 CI 失败（`.github/workflows/go.yml`）。
- 每个入库的 `.go`、`.sh`、`Makefile`、`Dockerfile` 都带 Apache-2.0 头（从 `Makefile` 复制）。

**E. 流程**（`CONTRIBUTING.md`）
- 合并需要一次批准评审 + `build-and-test` 通过；批准之后再推送，需要重新批准。认领 issue 或 PR 时把自己指派上去。

## 技能——动手前先加载

| 领域 | 技能 | 何时加载 |
|---|---|---|
| Substrate | `/substrate` | 触碰 `internal/substrate`、`internal/guest`、`internal/metadata`；升级 Substrate；任何 `ate-system` 地址；排查 E2E |
| 契约同步 | `/sync-contract` | 任何代码改动之后、宣布完成之前：核对 `AGENTS.md` 与 `skills/` 是否仍与代码一致 |

约定：一个技能一个目录，`skills/<name>/SKILL.md`，frontmatter 含 `name` + `description`（一句说是什么，一句说何时用——它是路由文本，不是文档；正文过长就拆到 `references/`）。agent 本地根目录（`.agents/`、`.claude/`、`.gemini/`）只符号链接到 `skills/`，绝不把技能源码放在那里。新增技能，就要在上表加一行。

## 完成定义

1. 上面四条 CI 命令在本地通过。
2. 若改动了 `internal/store/postgres` 或 E2E 相关路径，Postgres 层或 E2E 层通过。
3. 已针对你的 diff 跑过 `/sync-contract`，使 `AGENTS.md` 与 `skills/` 与代码保持一致。
