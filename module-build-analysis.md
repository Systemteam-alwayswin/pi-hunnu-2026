# pi-hunnu-2026 模块构建与依赖分析

> 分析依据：上传压缩包中的 `package.json`、各包 `README.md`、`tsconfig.build.json`、`src/` 源码及根目录构建脚本。路径均相对于仓库根目录 `pi-hunnu-2026/`。

## 一、模块分析表

| 子文件夹 | 代码主要实现什么功能 | 本模块 build 执行什么、生成什么 | 直接依赖哪些内部模块 | 依据文件或代码位置 |
|---|---|---|---|---|
| `telemetry` | 提供与厂商无关的遥测基础契约：TelemetryContext/Span、事件、Span 传播与链接、计数器/直方图等指标类型，并提供 NOOP 和内存参考实现。它定义接口，不绑定某个遥测后端。 | `tsc -p tsconfig.build.json`；按 `tsconfig.build.json` 将 `src/**/*.ts` 编译到 `dist/`，生成 JS、声明文件及 source map。 | 无 | `packages/telemetry/README.md`；`src/index.ts`、`src/memory.ts`、`src/metrics.ts`、`src/noop.ts`；`package.json` 的 `scripts/dependencies` |
| `schema` | JSON Schema 构建、类型推导、编译与校验；支持 Draft 2020-12/7，提供错误路径与格式扩展。 | `tsc -p tsconfig.build.json`；把 `src/**/*.ts` 编译到 `dist/`，包括格式扩展入口。 | 无 | `packages/schema/README.md`；`src/builders.ts`、`src/compiler.ts`、`src/checks.ts`、`src/formats/index.ts`；`tsconfig.build.json` |
| `chord` | 跨进程对象运行时：对象契约、Host/Facet、文档和事务、派生值、任务/Job、进程间链接、存储、插件热加载，以及 HTTP、Workers、Web/终端客户端等能力。 | `tsc -p tsconfig.build.json`；编译整个 `src/` 到 `dist/`，产出根入口及 `http`、`workers`、`plugins`、`web`、`tui`、`sqlite` 等多入口对应文件和声明。 | `schema`、`telemetry` | `packages/chord/README.md`；`src/contracts/schema.ts`、`src/context/index.ts`、`src/node/`、`src/store/`；`package.json` 的 `dependencies`、`exports` |
| `tui` | 终端 UI 框架：差异化渲染、主屏/备用屏渲染器、输入与焦点管理、组件、主题、Markdown 和终端图像等。 | 默认 `build` 仅执行 `tsc -p tsconfig.build.json`，把 `src/` 编译到 `dist/`。原生扩展另有 `build:native:darwin/linux/win32`，不属于默认 `build`。 | 无 | `packages/tui/README.md`；`src/tui.ts`、`src/components/`、`src/terminal.ts`；`package.json` 的 `scripts/dependencies`；`native/*/README.md` |
| `codemode` | 在 QuickJS（WASM）沙箱中执行模型生成的 JavaScript；脚本只能通过宿主注入的工具/函数与外界交互，提供 Node/Bun、Cloudflare 和 inline 等运行方式。 | `tsc -p tsconfig.build.json`；编译 `src/` 到 `dist/`，包含 runtime、worker、portable、Cloudflare 等入口。 | 无 | `packages/codemode/README.md`；`src/runtime/`、`src/wasm.ts`、`src/portable.ts`、`src/cloudflare.ts`；`package.json` |
| `mcp` | 独立的 Model Context Protocol 客户端：负责 MCP 初始化、请求关联、超时、取消、服务端请求和工具调用；提供 stdio、Streamable HTTP、Pipe 等传输。 | `tsc -p tsconfig.build.json`；编译 `src/` 到 `dist/`，包含主入口、OAuth 和 testing 等入口。 | 无 | `packages/mcp/README.md`；`src/client.ts`、`src/protocol/`、`src/transports/`、`src/oauth/`；`package.json` |
| `ai` | 统一多厂商 LLM API：模型目录与 Provider、鉴权、流式/非流式请求、工具调用、图像/分类能力、Token 与成本统计、遥测等。 | `npm run generate-models && npm run build:offline`。先运行 `catalog/generate.ts --strict` 刷新生成的模型数据；再检查模型数据、执行 TypeScript 编译，并清理/复制 `src/providers/data` 到 `dist/providers`。 | `schema`、`telemetry` | `packages/ai/README.md`；`src/models.ts`、`src/providers/`、`src/api/`、`src/telemetry.ts`、`catalog/generate.ts`；`package.json` 的 `scripts/dependencies` |
| `durable` | 持久化 Agent/会话运行时：把会话输入、模型轮次、工具调用、任务和文档状态提交到存储；支持进程崩溃后的恢复、任务调度、工具、压缩、图像和 Code Mode 集成。 | `tsc -p tsconfig.build.json`；编译 `src/` 到 `dist/`，产出主入口以及 `env`、`codemode`、`tools`、`storage/*`、`images/*` 等子入口。 | `chord`、`ai`、`codemode`、`schema`、`telemetry` | `packages/durable/README.md`；`src/harness/`、`src/storage/`、`src/codemode/`、`src/tools/`；`package.json` 的 `dependencies`、`exports` |
| `env` | 为 Durable 提供远程执行环境：通过 SSH 连接远端主机、部署并启动 Rust daemon，远程执行命令/文件操作；也支持进程管道、伪终端和文件列表。 | 默认 `build` 执行 `tsc -p tsconfig.build.json`，生成 TS 包的 `dist/`。Rust daemon 另由 `build:daemon`（`cargo build --manifest-path daemon/Cargo.toml --locked`）构建，不包含在默认 `build` 中。 | `chord`、`durable` | `packages/env/README.md`；`src/remote-env.ts`、`src/connection.ts`、`src/ssh.ts`、`src/process.ts`；`daemon/`；`package.json` |
| `agent` | 通用的状态化 Agent 核心：维护消息和 Agent 状态，调用模型、循环执行工具，并通过事件流报告消息、轮次和工具执行进度。 | `tsc -p tsconfig.build.json`；把 `src/` 编译到 `dist/`。 | `ai`、`schema` | `packages/agent/README.md`；`src/agent.ts`、`src/agent-loop.ts`、`src/stream-fn.ts`、`src/types.ts`；`package.json` |
| `coding-agent` | 面向开发工作的完整 Pi 编码 Agent/CLI：命令行启动、交互式终端 UI、会话与工具、扩展、MCP 服务、Code Mode、配置和 RPC 等。它是上述库的应用层集成者。 | `npm run build:unbundled && node ../../scripts/build-coding-agent-bundle.mjs`。前者运行 `tsc`、设置 CLI/RPC 入口可执行属性并复制主题、图片、HTML 等资源；后者用 esbuild 将 CLI、SDK/RPC、worker 等入口打包到 `dist/bundle/`。 | `agent`、`ai`、`codemode`、`mcp`、`tui` | `packages/coding-agent/README.md`；`src/cli/`、`src/core/`、`src/extensions/`、`src/rpc/`；`package.json`；`scripts/build-coding-agent-bundle.mjs` |

## 二、根目录与子模块的 build

### 根目录 `npm run build`

根目录 `package.json` 的 `scripts.build` 使用一串 `cd ... && npm run build`，按以下次序逐个进入包目录执行构建：

`telemetry → schema → chord → tui → codemode → mcp → ai → durable → env → agent → coding-agent`

因此根目录构建不是一次性对所有源码执行一个 TypeScript 命令，而是按仓库显式编排顺序调用每个子包自己的 `build` 脚本。`&&` 表示前一模块失败时后续模块不会继续。

根目录还提供 `build:offline`：整体顺序相同，但 `ai` 使用 `build:offline`，不重新生成/刷新模型目录，而是检查并使用仓库已有模型数据。

根目录 `build` 本身不先执行 clean，也不等同于 `test`、`check`、原生扩展构建或独立可执行文件构建。根目录 `prepublishOnly` 才是 `clean → build → check`。

### 子模块 `npm run build`

在某个子包目录中执行 `npm run build`，读取该目录的 `package.json`，执行该包自己的 `scripts.build`。大多数包的脚本是 `tsc -p tsconfig.build.json`；这个配置通常继承根目录 `tsconfig.base.json`，并把 `rootDir` 指向本包 `src`、`outDir` 指向本包 `dist`。

例外：
- `ai`：先生成模型目录，再运行离线构建步骤。
- `coding-agent`：先 TypeScript 编译和资源复制，再用 esbuild 生成 bundle。
- `tui`：默认 build 只编译 TypeScript；原生模块需显式调用 `build:native:*`。
- `env`：默认 build 只编译 TypeScript；Rust daemon 需显式调用 `build:daemon`。

## 三、构建顺序与依赖关系

构建顺序通常需要满足内部包的先行构建要求：例如 `durable` 的 TypeScript 配置将 `chord`、`ai` 等内部包的类型路径指向它们的 `dist/*.d.ts`，所以这些依赖包需要先产生 `dist` 类型声明，后续包才能按该配置完成编译。

但**相邻构建不等于直接依赖**。根目录顺序中 `schema` 后面是 `chord`，两者确实有依赖；而 `chord` 后面是 `tui`，`tui` 并不依赖 `chord`。`tui` 后面是 `codemode`，两者也没有内部直接依赖。顺序是满足整体构建流程的线性安排，不是依赖边列表。

### 直接内部运行依赖（使用方 → 被依赖方）

- `chord → schema、telemetry`
- `ai → schema、telemetry`
- `durable → chord、ai、codemode、schema、telemetry`
- `env → chord、durable`
- `agent → ai、schema`
- `coding-agent → agent、ai、codemode、mcp、tui`
- `telemetry`、`schema`、`tui`、`codemode`、`mcp` 在这 11 个包范围内没有直接内部运行依赖。

图中不画开发依赖、外部 npm 依赖和可选 peer dependency。特别是 `chord` 声明的 `esbuild`、`react`、`ws` 是 `peerDependencies`，且在 `peerDependenciesMeta` 中标记为 optional；它们不是上述 11 个 Pi 包之间的内部依赖，也不是 `chord` 的必需内部构建边。

源码文档中存在的协作示例也不应误判为包依赖：例如 `mcp` README 展示了如何把 `env.openProcess()` 接到 `PipeTransport`，但 `mcp/package.json` 并未声明 `pi-env` 依赖，二者通过结构化接口协作；同样，Chord 的文件系统接口注释提到 Durable 的接口形状，并不构成 package.json 依赖。

## 四、三模块及以上依赖路径示例

选择路径：

`coding-agent → agent → ai → schema`

1. `coding-agent` 是最终应用入口，负责把用户的 CLI/交互请求、工具和配置组织起来，并调用 Agent 核心。
2. `agent` 负责通用 Agent 运行循环、状态管理、消息转换和工具执行调度；它不直接实现具体厂商模型协议。
3. `ai` 统一封装不同 LLM Provider，提供模型选择、鉴权、请求/响应流式处理等能力。
4. `schema` 提供 JSON Schema 构建、类型和参数校验能力，供 `ai` 的模型/provider 定义及 Agent 类型/参数约束使用。

这条链体现了应用层、Agent 编排层、模型接入层、基础数据契约层的分工。注意 `coding-agent` 还直接依赖 `ai`，所以图中同时存在 `coding-agent → ai` 的跨层直接边；不能把图理解成只有单一调用链。

## 五、目录与版本差异说明

本次分析的 11 个模块实际位于 `packages/<模块名>/`，且根目录 build 脚本明确构建这些目录。仓库中另外存在 `apps/p2/plugins/agent`、`apps/p2/plugins/codemode`、`apps/p2/plugins/mcp`、`apps/p2/plugins/telemetry` 等同名目录；它们的 `package.json` 只有 `name/private/type` 等元信息，没有这里所分析的库构建脚本，不能与 `packages/` 下的正式包混为一谈。

本仓库根目录 `package.json` 的 `build` 列表中没有单独的 `schema/chord/...` 之外的其他模块；本报告严格按题目给出的 11 个包和该版本的实际构建脚本分析。
