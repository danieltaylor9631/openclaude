# OpenClaude 设计说明书

**文档编号：** OC-DESIGN-2026-09  
**对应产品版本：** `@gitlawb/openclaude` 0.30.0  
**文档性质：** 源代码级设计说明书（架构、数据模型、功能、函数、算法、测试）  
**编写依据：** 仓库当前 `main` 快照的 TypeScript / TSX 源码、测试用例、集成描述符、CLI 入口与文档站点数据源。  
**读者对象：** 维护者、贡献者、二次集成方、需要审计权限与工具行为的安全团队、以及需要把 OpenClaude 嵌入自有工作流的 Agent 平台工程师。

本说明书不是营销材料，也不是 README 的扩写。它按“系统如何被设计出来、每个部件为什么存在、每个数据形状如何流转、每个算法在何种约束下工作、每个测试在守护哪条不变量”的粒度展开。文中出现的模块路径、类型名、函数名、工具名、命令名、测试名均来自当前源码树，而不是事后猜测。

---

## 1. 文档约定与阅读路径

### 1.1 术语

| 术语 | 含义 |
| --- | --- |
| REPL | 交互式终端会话，由 Ink + React 渲染，入口在 `src/main.tsx` → `launchRepl`。 |
| Query | 一次用户回合到模型返回（含多轮工具调用）的主循环，核心在 `src/query.ts`。 |
| QueryEngine | SDK / 无头路径上的查询引擎封装，位于 `src/QueryEngine.ts`。 |
| Tool | 模型可调用的能力单元，类型定义在 `src/Tool.ts`，注册在 `src/tools.ts`。 |
| Slash Command | 用户以 `/` 触发的本地命令，注册在 `src/commands.ts`。 |
| Provider / Route | 一个可被选中的模型后端（Anthropic、OpenAI 兼容、Ollama、Bedrock 等），描述符位于 `src/integrations/`。 |
| PermissionContext | 当前回合的权限快照，含 mode、allow/deny/ask 规则与额外工作目录。 |
| Compact | 在上下文逼近窗口上限时，把历史折叠成摘要并插入 `compact_boundary`。 |
| MCP | Model Context Protocol，把外部服务器的 tools/resources/prompts 接入同一工具池。 |
| Skill | 可被斜杠命令或 Skill 工具加载的指令包，可来自内置、项目目录或插件。 |
| Transcript | 会话落盘记录，支持 resume、fork、replay、export。 |
| Bare mode | `--bare` 最小模式：跳过 hooks、LSP、插件同步、归因、自动记忆与 CLAUDE.md 自动发现。 |

### 1.2 设计原则（从代码中反推）

1. **一个 Agent 运行时，多条 Provider 运输层。** 工具、权限、会话、UI、压缩、MCP 对所有模型共用；差异被收口到 `src/integrations/` 的描述符与 `src/services/api/` 的运输层（`claude.ts`、`openaiShim.ts`、`codexShim.ts`、Gemini/Vertex 客户端）。
2. **权限先于执行。** 任何工具都要经过 `validateInput` → `checkPermissions` → `canUseTool` / hooks / 分类器，才能进入 `call()`。
3. **上下文是稀缺资源。** 自动压缩、微压缩、工具结果落盘、ToolSearch 延迟加载、repo map、token 预算、max-active-messages 都是同一问题的不同阀门。
4. **交互与无头同构。** REPL、`--print`、SDK `query()`、后台 `--bg` 都走 Query / QueryEngine，差别在 IO 适配与权限 UI 是否存在。
5. **特性开关做死代码消除。** `feature('...')` 配合 Bun 打包，使 PROACTIVE、COORDINATOR_MODE、VOICE_MODE 等在未开启时不进入产物。
6. **测试守护回归，而不是装饰。** 仓库有约 739 个测试文件、超过 23 万行测试代码，覆盖权限、压缩、provider 兼容、中断分类、定价、SDK 生命周期等。

### 1.3 建议阅读顺序

若目标是理解“一次回车之后发生了什么”，按第 4 章架构、第 8 章查询循环、第 9 章工具、第 7 章权限阅读。  
若目标是接入新模型，按第 11 章集成系统与 `docs/integrations/` 阅读。  
若目标是审计安全，按第 7 章、Bash 工具安全子模块、沙箱与 YOLO 分类器阅读。  
若目标是扩展命令或 Skill，按第 10 章与第 13 章阅读。

---

## 2. 产品定位与问题域

OpenClaude 是一个面向软件工程的编码 Agent CLI。它要同时满足三类用户：

- **终端里的开发者：** 在仓库根目录运行 `openclaude`，用自然语言改代码、跑测试、查文档、管理 git。
- **脚本与 CI：** `openclaude --print` / `--output-format stream-json` / `--json-schema`，把 Agent 嵌进管道。
- **嵌入方：** 通过 `@gitlawb/openclaude/sdk` 以编程方式驱动同一套引擎。

它要解决的核心问题不是“调用一次 LLM”，而是：

1. 把不稳定的模型输出约束成可执行的工具调用；
2. 在用户机器上安全地读文件、改文件、跑命令；
3. 在有限上下文窗口里维持跨小时甚至跨天的任务连续性；
4. 把上百个模型供应商的差异隐藏在同一套交互与权限模型后面；
5. 让本地模型、云端模型和订阅计划可以切换，而不重写工作流。

对应约束：

- 运行时必须是 Node.js `>=22`（安装与执行），开发与测试用 Bun。
- 默认不以守护进程形式常驻；后台会话是本地子进程，而不是网络服务。
- 不自动加载项目 `.env`，避免把密钥意外注入子进程；provider 配置走 `/provider` 或显式 `--provider-env-file`。
- 隐私路径可验证：`bun run verify:privacy` / `scripts/verify-no-phone-home.ts` 用于检查非必要外连。

---

## 3. 技术栈与工程约束

### 3.1 语言与模块系统

- **语言：** TypeScript 5.9，`strict` 模式，ESM（`package.json` 中 `"type": "module"`）。
- **UI：** React 19 + 自研 / 定制 Ink 渲染器（`src/ink/`），在终端里画框、列表、权限对话框、Markdown、diff。
- **校验：** Zod 3.25 作为工具输入 schema 与配置解析的主路径；部分 MCP 工具直接提供 JSON Schema。
- **CLI 解析：** `commander` 12 + `@commander-js/extra-typings`。
- **子进程：** `execa`、`cross-spawn`、`tree-kill`。
- **搜索：** `@vscode/ripgrep` 驱动 Grep；Glob 走文件系统与 ignore 规则。
- **索引：** `@orama/orama` 用于技能/知识等本地检索。
- **协议：** `@modelcontextprotocol/sdk`、`@grpc/grpc-js`（可选 gRPC 开发路径）、WebSocket。
- **树分析：** `web-tree-sitter` + `tree-sitter-wasms`。

运行时依赖刻意很少：`package.json` 的 `dependencies` 只有 Orama 与 ripgrep。其余大量库放在 `devDependencies`，由 Bun 打包进 `dist/cli.mjs` / `dist/sdk.mjs`。这是为了 npm 安装体积与“一个可执行产物”的分发模型。

### 3.2 构建

`scripts/build.ts` 使用 Bun 把 CLI 与 SDK 打成 ESM 包。构建期会：

- 根据 `feature()` 做条件编译；
- 生成集成产物（`src/integrations/generated/`）；
- 处理可选运行时（Anthropic SDK、MCP SDK、React 作为 peer/optional）；
- 注入隐私相关 stub（GrowthBook / telemetry 在开源路径上被限制）。

验证契约写在 `CONTRIBUTING.md`：`bun install`、`bun run build`、`bun run smoke`、`bun run check`、`bun run typecheck`、`bun run typecheck:type-tests`。CI 额外覆盖干净 runner 与 Node 矩阵。

### 3.3 目录分层（逻辑架构，而非仅仅文件夹）

| 层 | 目录 | 职责 |
| --- | --- | --- |
| 入口层 | `src/entrypoints/`、`src/main.tsx`、`bin/openclaude` | 解析 argv、初始化、启动 REPL / print / SDK / MCP host |
| 会话层 | `src/QueryEngine.ts`、`src/query.ts`、`src/state/` | 回合状态、中断、压缩、预算、消息规范化 |
| 工具层 | `src/tools/`、`src/Tool.ts`、`src/services/tools/` | 工具定义、编排、流式执行、结果预算 |
| 命令层 | `src/commands/`、`src/commands.ts` | 斜杠命令与 CLI 子命令 |
| 运输层 | `src/services/api/` | 各供应商 HTTP/SSE/OAuth |
| 集成层 | `src/integrations/` | 描述符：厂商、网关、模型、品牌、兼容性 |
| 权限层 | `src/utils/permissions/`、`src/types/permissions.ts` | 规则、模式、分类器、路径校验 |
| 表现层 | `src/components/`、`src/screens/`、`src/ink/` | 终端 UI |
| 记忆层 | `src/memdir/`、`src/services/SessionMemory/`、`src/services/compact/` | 记忆、压缩、wiki、knowledge |
| 扩展层 | `src/plugins/`、`src/skills/`、`src/services/mcp/` | 插件、技能、MCP |
| 任务层 | `src/tasks/`、`src/tools/Task*Tool/` | 本地/远程/后台任务 |
| 配套 | `vscode-extension/`、`web/`、`scripts/` | 编辑器扩展、文档站、构建与医生脚本 |

---

## 4. 整体架构设计

### 4.1 一次交互的端到端数据流

用户在 REPL 按下 Enter 之后，数据沿以下管道流动。这是整份设计的主轴。

1. **输入采集。** `src/hooks/useTextInput.ts`、Vim 模式（`src/vim/`）、粘贴处理（`usePasteHandler`）把击键变成字符串或图像附件。
2. **命令分发。** 若以 `/` 开头，`src/commands.ts` 的 `findCommand` / `getCommand` 把它当成本地命令；否则进入模型回合。
3. **用户消息构造。** `createUserMessage`（`src/utils/messages.ts`）生成 `UserMessage`，带 UUID、时间戳、可选 `origin`、`permissionMode`。
4. **Query 启动。** REPL 或 SDK 调用 `query()`。`buildQueryConfig` 汇总模型、工具池、系统提示、思考配置、预算。
5. **系统提示组装。** `fetchSystemPromptParts` / `getSystemContext` 注入日期、仓库、git 状态、CLAUDE.md、记忆、技能目录、MCP 资源摘要、repo map。
6. **运输层请求。** `src/services/api/client.ts` 按当前 route 分发到 Anthropic Messages、OpenAI Chat Completions / Responses、Gemini、Ollama 等。流式事件被归一成内部 `StreamEvent`。
7. **工具调用编排。** 模型产出 `tool_use` 块后，`StreamingToolExecutor` 与 `runTools`（`src/services/tools/toolOrchestration.ts`）按并发安全、权限、失败循环守卫执行。
8. **结果回注。** 每个工具通过 `mapToolResultToToolResultBlockParam` 变成 `tool_result`，超长结果由 `applyToolResultBudget` 落盘并替换为预览。
9. **循环或终止。** 直到模型不再调用工具、触发 stop hooks、命中步数/预算/中断、或用户插入新消息。
10. **持久化。** `recordTranscript` / `flushSessionStorage` 把消息写入会话文件，供 `--resume`、`--continue`、`--fork-session` 使用。

### 4.2 进程与隔离模型

- **前台 REPL：** 单 Node 进程，Ink 占用 stdout。
- **`--print`：** 无 UI，结果写 stdout；工作区信任对话框被跳过，因此文档明确要求只在可信目录使用。
- **`--bg`：** 拉起本地子进程，不启动 daemon；`openclaude ps/logs/kill` 管理这些子进程。
- **`--worktree`：** 在独立 git worktree 中跑会话，可选 `--tmux` 分屏。fork-session 只分叉对话，不分叉文件系统。
- **子 Agent：** `AgentTool` / `runAgent.ts` 在同一进程内开嵌套 query，使用 `createSubagentContext` 限制 `setAppState`，但 `setAppStateForTasks` 仍能把后台任务登记到根存储。
- **Swarm / Team：** `TeamCreateTool`、`SendMessageTool`、mailbox / UDS inbox 把多个 teammate 连成进程内协作。
- **Sandbox：** Bash 可按 `shouldUseSandbox` 进入沙箱运行时（`@anthropic-ai/sandbox-runtime` 在相关构建中可用）。

### 4.3 控制面与数据面分离

控制面包括：配置（`src/utils/config.ts`）、权限规则、provider profile（`.openclaude-profile.json`）、插件市场、MCP 服务器列表、settings.json 三层（user / project / local）。  
数据面包括：当前消息数组、文件状态缓存 `FileStateCache`、文件历史 `FileHistoryState`、token 用量、压缩边界、工具结果替换表。

控制面可以在回合之间热更新（settings change detector、skill change detector）；数据面在 compact 时被分区、裁剪、摘要。

### 4.4 依赖方向

严格的依赖方向是：

`entrypoints/main → QueryEngine/query → tools/commands → services/api + integrations → utils/types`

禁止工具实现直接 new 一个运输客户端；工具只通过 `ToolUseContext` 拿状态。`src/types/permissions.ts` 被刻意做成“无运行时依赖的纯类型文件”，以打破权限与工具之间的循环 import。`buildTool()` 把散落的方法收成满足 `Tool` 接口的对象，便于 Tree-shaking 与测试替换。

### 4.5 失败域

系统把失败分成可重试、可降级、必须停三类：

- **可重试：** 运输层 429/5xx、流中断、OAuth 刷新；`withRetry`、`fetchWithProxyRetry`、`providerFallbackChain` 处理。
- **可降级：** 某个 MCP 服务器挂了只移除其工具；某个图片缩放失败可按策略放行原图（相关修复见 image resize 路径）；LSP 未连接则从可用工具池过滤 `LSPTool`。
- **必须停：** 用户 Ctrl+C（user interruption）、query timeout、hard max、权限拒绝且无 fallback、预算耗尽、连续 autocompact 失败触发冷却。

`src/query.abortClassification.test.ts` 明确区分：timeout / hard-max / background abort **不得**被渲染成“用户打断”，以免污染 transcript 语义。

---

## 5. 入口、启动与配置加载顺序

`src/main.tsx` 在其它 import 之前执行三件启动侧效应，这是经过性能剖析的：

1. `profileCheckpoint('main_tsx_entry')` 标记启动剖析起点。
2. `startMdmRawRead()` 并行拉起 MDM（macOS `plutil` / Windows 注册表）读取，避免后面同步 spawn 阻塞。
3. `startKeychainPrefetch()` 并行预取钥匙串中的 OAuth 与遗留 API Key。

随后 `main()` 用 Commander 注册全局选项与子命令。选项分组与文档站 `web/src/data/cliFlags.ts` 保持同步，包括：

- 核心：`--version`、`--print`、`--bare`、`--debug`、`--verbose`
- IO：`--output-format`、`--input-format`、`--json-schema`、`--heartbeat`
- 模型：`--model`、`--provider`、`--effort`、`--fallback-model`、`--agent`
- 会话：`--continue`、`--resume`、`--fork-session`、`--worktree`、`--tmux`
- 权限：`--permission-mode`、`--allowed-tools`、`--disallowed-tools`、`--yolo`
- MCP：`--mcp-config`、`--strict-mcp-config`
- 配置：`--settings`、`--setting-sources`、`--plugin-dir`、`--provider-env-file`

配置加载优先级（高到低，细节以 `src/utils/config.ts` 与 settings 合并实现为准）：

1. CLI 标志与 `--settings` JSON
2. 托管 / 远程策略（policy / remote managed settings）
3. 项目本地 `.openclaude/settings.local.json`
4. 项目共享 `.openclaude/settings.json`
5. 用户 `~/.openclaude/settings.json`（可用 `OPENCLAUDE_CONFIG_DIR` 改根）
6. 内置默认值与 provider 描述符默认模型

`subscriptionType` 被特别保护：只允许用户级设置覆盖，忽略项目/仓库/local/flag/policy，防止仓库投毒把账号类型改成 free 或反之。

---

## 6. 核心数据模型

本章按“每个类型一份说明书”展开。类型源文件主要是 `src/types/message.ts`、`src/types/permissions.ts`、`src/types/hooks.ts`、`src/types/tools.ts`、`src/Tool.ts`、`src/state/AppState.tsx`、`src/entrypoints/agentSdkTypes.ts`。

### 6.1 消息包络（Message union）

消息是系统的第一公民。`Message` 是判别联合：

- `UserMessage`：`type: 'user'`
- `AssistantMessage`：`type: 'assistant'`
- `AttachmentMessage`：`type: 'attachment'`
- `ProgressMessage`：`type: 'progress'`
- `SystemMessage`：`type: 'system'`，再用 `subtype` 二次判别

设计选择：每个变体带 `[key: string]: any` 索引签名。原因写在文件头注释——上游 Anthropic 源码的完整 Message 定义未完全镜像到开源快照，索引签名避免未重建字段导致 TS2339，同时保留 `type` / `subtype` 以便 narrowing。构造函数（`createUserMessage` 等）才是必填字段的真相来源。

#### 6.1.1 UserMessage

字段与职责：

- `uuid` / `timestamp`：稳定身份，供 rewind、snip、compact 锚点引用。
- `message.role = 'user'`，`content` 为字符串或 `ContentBlockParam[]`（多模态）。
- `isMeta`：对用户隐藏、对模型或内部逻辑可见。
- `isVisibleInTranscriptOnly`：只出现在 transcript 视图。
- `isVirtual`：合成消息，非人类键入。
- `isCompactSummary` / `isCollapseSummary`：压缩或上下文折叠产生的摘要；后者不可被 snip，以免唯一替代物被删。
- `summarizeMetadata`：记录被摘要的消息数、方向（`up_to` / `from`）、用户自定义摘要指令。
- `toolUseResult`：工具结果的结构化副本，供 UI 与 SDK；子 Agent 默认可剥离，除非 `preserveToolUseResults`。
- `imagePermissionToolUseIds`：图片与工具权限的归属；`null` 表示独立用户粘贴图。
- `isAgentStepLimitToolResult`：Agent 步数上限合成的 tool_result。
- `mcpMeta`：MCP `_meta` 与 `structuredContent` 透传。
- `sourceToolAssistantUUID` / `sourceToolUseID`：把 tool_result 绑回对应 tool_use，防止乱序。
- `permissionMode`：发送时的权限模式，rewind 时要还原。
- `origin`：`human` | `coordinator` | `task-notification` | `channel`。

#### 6.1.2 AssistantMessage

- 内嵌 `AssistantMessageContent`：`role/content/id/model/usage`。
- `requestId`：运输层请求关联。
- `apiError` / `error` / `errorDetails` / `isApiErrorMessage`：把失败变成可展示、可恢复的消息，而不是扔异常。
- `advisorModel`：顾问模型路径。
- `usage` 使用 Anthropic BetaUsage 形状，便于跨供应商归一后再进 `cost-tracker`。

#### 6.1.3 AttachmentMessage

附件不塞进 user content 的主路径，而是并行包络，便于 `filterDuplicateMemoryAttachments`、记忆预取、以及 compact 时按附件 UUID 清理。

#### 6.1.4 ProgressMessage

流式工具进度。`toolUseID` + `parentToolUseID` 支持嵌套（Agent 内部再调工具）。`filterToolProgressMessages` 会丢掉 hook_progress，避免 UI 把 hook 进度画成工具进度。

#### 6.1.5 SystemMessage 家族

这是设计里最“产品化”的内部事件流，subtype 包括：

| subtype | 含义 | 对模型可见？ |
| --- | --- | --- |
| informational | 普通系统通知；若 `isCollapseSummary` 则在 normalize 时转成 user 摘要 | 条件可见 |
| permission_retry | 权限被拒后的重试提示与命令建议 | 通常不可见或受限 |
| bridge_status | 远程桥接 URL 与升级提示 | 否 |
| scheduled_task_fire | cron/调度任务触发 | 作为触发来源 |
| stop_hook_summary | Stop hook 汇总：数量、耗时、是否阻止继续 | 否（UI） |
| turn_duration | 回合耗时与预算统计 | 否 |
| away_summary | 用户离开后的摘要 | 否 |
| memory_saved | 持久记忆写入路径列表 | 否 |
| agents_killed | 子 Agent 被清理 | 否 |
| api_metrics | TTFT、OTPS、hook/tool/classifier 耗时 | 否 |
| local_command | 本地斜杠命令的 stdout/stderr 包装 | 经 XML 标签进入上下文 |
| compact_boundary | 压缩边界与 CompactMetadata | 作为历史截断锚点 |
| microcompact_boundary | 微压缩：去掉工具结果但保留结构 | 锚点 |
| api_error | 可重试 API 错误，含 retryInMs | 否 |
| file_snapshot | 远程会话的计划/待办快照 | 恢复用 |
| thinking | 思考占位，渲染为 null | 否 |
| snip_boundary | HISTORY_SNIP 删除消息后的边界 | 锚点 |

#### 6.1.6 CompactMetadata

- `trigger: 'manual' | 'auto'`
- `preTokens`、`messagesSummarized`、`userContext`
- `preservedSegment`：`headUuid/anchorUuid/tailUuid`，用于保留压缩后仍必须留下的尾部对话，避免把“正在做的事”摘要掉。

### 6.2 流事件（非 Message）

`StreamEvent`（`type: 'stream_event'`）与 `RequestStartEvent` 与 Message 并列 yield，供 UI 打字机效果与 TTFT 埋点。SDK 再映射为 `SDKMessage`。

### 6.3 权限模型

#### 6.3.1 PermissionMode

外部模式：`acceptEdits`、`bypassPermissions`、`default`、`dontAsk`、`fullAccess`、`plan`。  
内部还可有 `auto`、`bubble`。  
`INTERNAL_PERMISSION_MODES` 才是 settings.json / `--permission-mode` / 会话恢复允许的集合。


当前实现中各模式的设计意图如下：

**`default`（默认模式）。** 每次敏感工具调用都询问用户，适合日常交互与不可信仓库。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`acceptEdits`（接受编辑）。** 自动批准文件编辑类工具，仍询问 Bash、网络与破坏性操作。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`plan`（计划模式）。** 只允许只读探索与计划文件写入，禁止直接改代码，直到退出计划。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`bypassPermissions`（绕过权限）。** 跳过交互式询问，仅用于隔离沙箱或用户明确授权的无人值守场景。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`dontAsk`（不问）。** 尽量不弹窗，结合规则与分类器决策。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`fullAccess`（完全访问）。** 宽权限模式，仍受策略文件与危险模式熔断约束。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`auto`（自动分类）。** 在 TRANSCRIPT_CLASSIFIER 特性开启时，用分类器决定是否放行。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。

**`bubble`（气泡模式）。** 内部权限模式，用于特定宿主 UI。 该模式不是简单的布尔开关，而是与 `alwaysAllowRules` / `alwaysDenyRules` / `alwaysAskRules`、计划文件路径、Bash 分类器、YOLO 分类器、危险模式 killswitch 共同作用。例如 `plan` 模式会把写工具拒绝或重定向到计划文件；`acceptEdits` 只放行被判定为编辑且路径在工作区内的操作；`bypassPermissions` 仍可能被 `bypassPermissionsKillswitch` 或策略文件拦住。


#### 6.3.2 PermissionRule

一条规则由三部分组成：

- `source`：`userSettings` | `projectSettings` | `localSettings` | `flagSettings` | `policySettings` | `cliArg` | `command` | `session`
- `ruleBehavior`：`allow` | `deny` | `ask`
- `ruleValue`：`{ toolName, ruleContent? }`

`ruleContent` 是可选的内容匹配器。空内容表示“整把工具”的blanket 规则。Bash 使用 `Bash(git *)` 这种语法，由 `permissionRuleParser.ts` 解析、`shellRuleMatching.ts` 做通配。MCP 工具可用 `mcp__server` 前缀匹配该服务器下全部工具——`filterToolsByDenyRules` 在模型看到工具列表之前就把它们摘掉，而不是等到调用时再拒绝。这是一项明确的产品决策：拒绝应该发生在“模型规划阶段”，否则模型会反复尝试被禁工具，浪费 token。

#### 6.3.3 PermissionUpdate

运行时可以增量改权限，而不是重写整个文件。判别联合包含：`addRules`、`replaceRules`、`removeRules`、`setMode`、`addDirectories`、`removeDirectories`。每条更新带 `destination`，决定写到 user/project/local/session/cliArg。会话级更新只影响当前进程，适合“这次允许跑测试”。

#### 6.3.4 PermissionResult

`checkPermissions` 返回 allow / deny / ask。allow 可带 `updatedInput`（用户在对话框里改了命令）与 `userModified`。ask 会进入 Ink 权限对话框或 SDK 的 `canUseTool` 回调。deny 带原因字符串，映射回模型可见的 tool_result 错误，促使模型改方案而不是死循环。`denialTracking` 统计连续拒绝次数，超过阈值可回退为强制询问，防止后台 Agent 在无 UI 时静默失败到停机。

#### 6.3.5 AdditionalWorkingDirectory

`--add-dir` 与 `/add-dir` 把额外目录登记为 `{ path, source }`。文件系统权限（`filesystem.ts`）用它扩展可读可写根。路径校验包含 tilde 展开、符号链接解析、以及 Windows 路径规范化（`windowsPaths.ts`）。

### 6.4 Tool 接口（`src/Tool.ts`）

`Tool<Input, Output, P>` 是整个 Agent 能力的契约。必填与关键可选成员：

- `name` / `aliases`：主名与兼容旧名（Agent 曾名 Task）。
- `searchHint`：给 ToolSearch 用的 3–10 词能力短语。
- `inputSchema`：Zod；MCP 可另给 `inputJSONSchema`。
- `outputSchema`：可选输出校验。
- `call(...)`：真正执行，返回 `ToolResult<Output>`，可附 `newMessages`、`contextModifier`、`mcpMeta`。
- `description(...)`：动态一句话描述，随 input 与是否无头变化。
- `prompt(...)`：注入系统提示的长说明，含何时用、何时不用、危险示例。
- `isEnabled()`：环境开关。
- `isReadOnly` / `isConcurrencySafe` / `isDestructive` / `isOpenWorld`：编排与权限分类。
- `interruptBehavior`：`'cancel' | 'block'`，决定用户插入新消息时是否杀掉工具。
- `shouldDefer` / `alwaysLoad`：与 ToolSearch 延迟加载协议配合。
- `maxResultSizeChars`：超限则落盘。Read 设为 Infinity，避免 Read→文件→再 Read 的循环。
- `validateInput`：不通过则直接以错误返回模型，不弹权限窗。
- `checkPermissions`：工具私有权限逻辑，通用逻辑在 `permissions.ts`。
- `preparePermissionMatcher`：把 `Bash(git *)` 编译成闭包。
- `mapToolResultToToolResultBlockParam`：模型可见序列化。
- `renderToolUseMessage` / `renderToolResultMessage`：Ink UI。
- `extractSearchText`：transcript 搜索索引必须与屏幕可见文本一致，有专门保真测试。
- `toAutoClassifierInput`：自动模式安全分类器的紧凑表示。
- `backfillObservableInput`：在不破坏 prompt cache 的前提下给观察者补遗留字段。

`ToolUseContext` 是执行时的世界状态：options（命令、模型、工具、MCP、agent 定义、预算、自定义系统提示）、`abortController`、`readFileState`、`getAppState/setAppState`、通知、JSX 注入、QueryGuard 租约、文件历史、归因、子 Agent id、内容替换状态、冻结的父系统提示字节（fork 时共享 cache）等。设计上刻意让工具函数签名稳定，把易变状态塞进 context，从而让 `runTools` 可以并行调度只读工具。

`buildTool()` / `ToolDef` 把实现从“手写满足大接口的对象”变成“提供需要的字段即可”，减少每个工具文件的样板。

### 6.5 AppState

`src/state/AppState.tsx` 与 `AppStateStore.ts` 持有会话级可变状态：消息列表、MCP 连接与工具、权限上下文、后台任务、插件命令、teammate 视图、通知队列。`onChangeAppState.ts` 做派生与持久化副作用。选择器在 `selectors.ts`，避免组件直接深挖大对象。异步子 Agent 的 `setAppState` 是 no-op，防止子 Agent 污染主 UI；它们必须走 `setAppStateForTasks` 或 `localDenialTracking`。

### 6.6 AgentDefinition

`src/tools/AgentTool/loadAgentsDir.ts` 从目录加载自定义 Agent：名字、描述、工具白名单、模型、记忆、颜色、prompt。内置 Agent 在 `built-in/`：`exploreAgent`、`planAgent`、`generalPurposeAgent`、`verificationAgent`、`codeReviewerAgent`、`claudeCodeGuideAgent`、`statuslineSetup`。`ONE_SHOT_BUILTIN_AGENT_TYPES` 标明 Explore/Plan 只跑一轮返回报告，省略 agentId/SendMessage 尾部以节省 token。

### 6.7 配置与定价模型

`settings.json` 关键键：`model`、`effortLevel`、`agent`、`permissions`、`env`、`hooks`、`smartRouting`、`modelLimits`、`modelPricing`、`providerFallbackChain`、`agentModels`、`agentRouting`、`advisorModel`。

`modelPricing` 设计约束（来自配置文档与测试）：

- 只从 user/local/`--settings`/SDK/managed 读取，**忽略**共享项目 settings，防止仓库投毒改价格显示；
- 每条必须有 input/output/cacheRead/cacheWrite 的每百万 token 美元价；
- `webSearchRequests` 默认 $0.01，显式 0 合法；
- 上限 256 条、id 512 字符、单价上限；
- 精确 id 全局覆盖，大小写敏感。

`cost-tracker.ts` 把 usage 聚合成会话成本；`QueryEngine.customPricingBudget.test.ts` 保证 SDK 的 `maxBudgetUsd` 与自定义零价/正价一致。

### 6.8 Hook 模型

`src/types/hooks.ts` 定义 PreToolUse、PostToolUse、Stop、pre_compact、post_compact、session_start 等。Hook 可返回允许、拒绝、改写 input、阻止继续。`handleStopHooks` 在 query 结束时运行；失败走 `executeStopFailureHooks`。无头模式可用 `--include-hook-events` 把生命周期打进 stream-json。

### 6.9 Task 模型

`src/tasks/types.ts` 与 `Task.ts` 描述本地主会话、本地 shell、本地 agent、远程 agent、MCP monitor、in-process teammate、dream、workflow。工具侧有 TaskCreate/Get/Update/List/Stop/Output。TodoWrite 是轻量待办；当 `isTodoV2Enabled()` 时切换到完整 Task 工具族。

### 6.10 SDK 对外模型

`src/entrypoints/sdk/` 把内部 Message 映射为稳定 SDK 类型：`SDKMessage`、`SDKPermissionDenial`、`SDKStatus`、`SDKCompactBoundaryMessage`。生成文件 `coreTypes.generated.ts`、`settingsTypes.generated.ts` 由 `scripts/generate-sdk-types.ts` 维护。测试覆盖大小写转换、生命周期、并发、MCP 清理、上下文隔离、工具 schema 缓存。

### 6.11 Provider 描述符模型

集成系统把“这是谁”和“怎么发请求”拆开：

- **Vendor / Gateway / Model / Brand descriptors**（`src/integrations/`）
- **route metadata / runtime metadata / transportConfig.kind**
- **profile**（用户保存的连接：base_url、api_key、model）

`transportConfig.kind` 是路由契约，例如 `'openai-compatible'`。新增供应商应改描述符而不是在 `claude.ts` 里堆 if。生成物 `integrationManifest.generated.ts` 是文档站与 `/provider` UI 的数据源。

### 6.12 其它重要结构体

- `ThinkingConfig`：是否默认开启思考、effort 如何映射到预算。
- `FileStateCache`：已读文件的 mtime/内容哈希，Edit 用来检测“文件被外部修改”。
- `FileHistoryState`：rewind 所需快照。
- `ContentReplacementState`：工具结果预算的替换表，主线程不重置，子 Agent 可克隆以共享 cache。
- `QueryChainTracking`：`{ chainId, depth }` 防止无限嵌套。
- `QueryActivity`：向 QueryGuard 注册活动、获取租约、在等人时暂停空闲计时。
- `AutoCompactTrackingState`：是否已压缩、回合计数、连续失败次数、冷却。
- `ValidationResult`：`{ result: true } | { result: false, message, errorCode }`。

---

## 7. 权限、安全与沙箱

权限子系统是编码 Agent 与“能在用户机器上执行任意命令的程序”之间的边界。

### 7.1 决策流水线

一次工具调用的安全流水线大致为：

1. 工具是否存在于当前 pool（`getTools` + MCP + deny 过滤 + `isEnabled`）。
2. `validateInput`：模式错误、路径越界、参数缺失。
3. Hook `PreToolUse`：可短路为允许或拒绝。
4. 通用规则匹配：deny > ask > allow，来源有优先级。
5. 工具私有 `checkPermissions`（Bash 命令语义、Edit 路径、WebFetch 域名）。
6. 模式策略：plan / acceptEdits / YOLO。
7. 可选分类器：`bashClassifier`、`yoloClassifier`、transcript classifier。
8. 用户对话框或 SDK `canUseTool`。
9. `PostToolUse` hook。
10. 失败循环守卫 `toolFailureLoopGuard`：同一工具连续失败则停止，避免燃烧配额。

### 7.2 Bash 安全模块

`src/tools/BashTool/` 不是一个文件能说完的工具，而是一套命令分析器：

- `bashCommandAnalysis.ts`：拆分复合命令、管道、重定向。
- `commandSemantics.ts`：判断读/写/搜索/列表，供 UI 折叠与只读判定。
- `bashSecurity.ts`：危险模式、替换漏洞、敏感路径。
- `bashPermissions.ts`：规则匹配与会话授权。
- `pathValidation.ts`：命令中的路径是否落在允许根内。
- `readOnlyValidation.ts`：声称只读的命令是否真只读。
- `destructiveCommandWarning.ts`：破坏性命令的额外警告。
- `sedEditParser.ts` / `sedValidation.ts`：把 sed 就地编辑纳入 Edit 同等风险。
- `shouldUseSandbox.ts`：是否进沙箱。
- `modeValidation.ts`：当前权限模式是否允许此类命令。

PowerShell 工具有平行实现：`powershellSecurity.ts`、`powershellPermissions.ts`、`gitSafety.ts`、`destructiveCommandWarning.ts`。

### 7.3 文件系统权限

`filesystem.ts` 计算可访问根：cwd、`--add-dir`、scratchpad、`.openclaude` 特殊模式。Edit 的 `CLAUDE_FOLDER_PERMISSION_PATTERN` 允许授权项目 `.openclaude/**` 与 `~/.openclaude/**`。意外外部修改会抛出 `FILE_UNEXPECTEDLY_MODIFIED_ERROR`，强迫模型先 Read 再 Edit，这是防冲突的乐观锁。

### 7.4 网络工具

`WebFetchTool` 有 `domainCheck`、`preapproved` 域名、prompt fallback。`WebSearchTool` 可插拔 provider：Brave、DuckDuckGo、Exa、Tavily、Jina、Firecrawl、Bing、Mojeek、You、Linkup、custom，统一超时层。搜索是 open-world 工具，权限上通常需显式允许。

### 7.5 危险跳过

`--yolo` / `--dangerously-skip-permissions` 只建议在无外网沙箱使用。另有 `--allow-dangerously-skip-permissions` 只是把选项暴露出来，并不默认打开。`dangerousSkipFlags`、`dangerousPatterns`、`dangerousModePrompt` 构成多层确认。`bypassPermissionsKillswitch` 可在运行时关掉。

---

## 8. 查询循环与 QueryEngine

### 8.1 `query()` 的状态机

`src/query.ts` 导出 `QueryParams`、`QueryTurnBudget`、`createQueryTurnBudget`，主体是一个生成器式循环，产出 Message 与 StreamEvent。关键步骤：

1. 规范化历史：`normalizeMessagesForAPI`，去掉纯 UI 系统消息，折叠 compact 边界之前的内容。
2. 附件与记忆：`getAttachmentMessages`、`startRelevantMemoryPrefetch`、去重。
3. 自动压缩检查：`isAutoCompactEnabled`、`getAutoCompactThreshold`、`calculateTokenWarningState`。
4. 调用运输层，处理 prompt-too-long：升级到 `ESCALATED_MAX_TOKENS` 或触发 compact 后重试。
5. 流式消费：thinking、text、tool_use。
6. `runTools` 执行工具，应用 `applyToolResultBudget`。
7. `analyzeContinuationIntent` 决定是否继续。
8. Stop hooks、目标（goal）检查、预算续跑计数 `incrementBudgetContinuationCount`。
9. 产出回合结束消息（turn_duration、api_metrics）。

`query/transitions.ts` 用 `Terminal | Continue` 把循环出口类型化，避免“再请求一次”的布尔陷阱。

### 8.2 中断分类

`abortReasons.ts` 与 `interruptionTrace.ts` 把中止分成用户打断、超时、hard max、后台、缺失 tool_result 等。`QueryEngine` 在 tracing 开启时记录 query-root 中断、成功终态、失败终态。测试禁止把超时写成用户打断，这直接影响产品文案与分析指标。

### 8.3 QueryEngine

面向 SDK 的门面：持有 messages、tools、system prompt、permission mode、usage 累计、autoCompact 状态。提供 `submitMessage`、中断、compact、goal 状态映射 `toSDKGoalStatusMessage`。它复用 `query()` 而不是另写一套，保证 REPL 与 SDK 行为一致。`fileHistoryMakeSnapshot` 在回合边界打快照。无头分析用 `headlessProfilerCheckpoint`。

### 8.4 工具编排算法

`toolOrchestration.ts` 的 `runTools`：

- 只读且 `isConcurrencySafe` 的工具并行；
- 写工具串行，且可 `contextModifier` 修改后续 context（例如更新 readFileState）；
- 尊重 `abortController`；
- 与 `StreamingToolExecutor` 配合，边流边启动已完整的 tool_use，降低尾延迟。

`queryActivityLease` 防止 QueryGuard 在长工具运行时误判空闲。

### 8.5 Provider 回退

`providerFallbackChain` 在 rate-limit/quota 时切换 profile。`resolveNextFallbackProviderFromState` + `setActiveProviderProfile` 保证下一请求用新凭据。这与 `--fallback-model`（同供应商换模型）是不同层的降级。


## 9. 工具系统详细设计

工具注册的唯一事实来源是 `getAllBaseTools()`。`getTools(permissionContext)` 在此之上做模式过滤、REPL 隐藏、deny 过滤、`isEnabled` 过滤。`assembleToolPool` 再合并 MCP 工具，重名时内置优先。`--tools` 可限制集合；`CLAUDE_CODE_SIMPLE` 只留 Bash/Read/Edit。协调器模式额外保留 Agent、TaskStop、SendMessage。

下面按实现目录逐一说明职责、关键函数、算法要点与测试挂钩。每个工具目录通常包含：主实现（`.ts`/`.tsx`）、`prompt.ts`（给模型的说明书）、`constants.ts`（名字，打断循环依赖）、`UI.tsx`（Ink 渲染）、以及 `*.test.ts`。

### 9.1 AgentTool（对外名：Agent）

派生子 Agent。可指定 type（Explore/Plan/general-purpose/verification/code-reviewer 等）、prompt、工具子集、模型、resume。`runAgent.ts` 开嵌套 query；`forkSubagent.ts` 共享父 prompt cache；`resumeAgent.ts` 恢复 sidechain。路由测试覆盖 Copilot scheduling、teammate 模型、持久化。别名 Task 保持旧权限规则有效。

实现落点：`src/tools/AgentTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.2 AskUserQuestionTool（对外名：AskUserQuestion）

向用户提出结构化问题。`requiresUserInteraction` 为真，QueryGuard 应 `beginUserInteraction` 暂停空闲计时。

实现落点：`src/tools/AskUserQuestionTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.3 BashTool（对外名：Bash）

在仓库工作目录执行 shell 命令，是 Agent 的“手”。输入通常包含 command、timeout、可选 description。算法上先做命令语义分析，再走权限与沙箱，最后用 execa 拉起进程，stdout/stderr 受 `BASH_MAX_OUTPUT_LENGTH`（默认 30000，上限 150000）截断，超长落入可再次 Read 的文件。只读判定影响并发：`git status` 可与 Read 并行，`rm` 必须串行。中断行为一般为 block 或按配置 cancel。测试覆盖 sandbox 分析、错误输出、权限、只读校验、sed 解析、路径校验。

实现落点：`src/tools/BashTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.4 BriefTool（对外名：Brief）

生成或展示 brief，用于 KAIROS 风格的任务简报。

实现落点：`src/tools/BriefTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.5 CtxInspectTool（对外名：CtxInspect）

CONTEXT_COLLAPSE：检查上下文折叠状态。

实现落点：`src/tools/CtxInspectTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.6 DiscoverSkillsTool（对外名：DiscoverSkills）

发现可用技能目录，配合实验性技能搜索。

实现落点：`src/tools/DiscoverSkillsTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.7 EnterPlanModeTool（对外名：EnterPlanMode）

模型主动进入计划模式，保存 `prePlanMode` 以便退出恢复。

实现落点：`src/tools/EnterPlanModeTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.8 EnterWorktreeTool（对外名：EnterWorktree）

在 worktree 模式开启时进入隔离工作树，避免主分支被实验性改动弄脏。

实现落点：`src/tools/EnterWorktreeTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.9 ExitPlanModeTool（对外名：ExitPlanMode）

V2 退出计划并请求开始执行。与 `/plan` 用户命令互补。

实现落点：`src/tools/ExitPlanModeTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.10 ExitWorktreeTool（对外名：ExitWorktree）

离开隔离工作树并回到原工作区。

实现落点：`src/tools/ExitWorktreeTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.11 FileEditTool（对外名：Edit）

在已知文件上做精确替换。要求 old_string 在文件中唯一匹配，否则失败并让模型重读。若 mtime 与缓存不一致，返回 `FILE_UNEXPECTEDLY_MODIFIED_ERROR`。成功后更新文件历史，供 `/rewind` 还原。UI 渲染 diff（`FileEditToolDiff`）。这是 acceptEdits 模式自动放行的核心工具。

实现落点：`src/tools/FileEditTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.12 FileReadTool（对外名：Read）

读取文件、图像、PDF。图像路径会走 `imageResizer`：超 5MB 或超 8000px 会缩放或报错；缩放失败时有策略允许大图/无元数据截图通过（回归 #1964）。PDF 按页数与大小阈值抽取。目录读取会列出条目。`maxResultSizeChars = Infinity` 防止结果落盘死循环。读取后更新 `FileStateCache`，为后续 Edit 做乐观锁。还会触发条件技能发现 `activateConditionalSkillsForPaths`。

实现落点：`src/tools/FileReadTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.13 FileWriteTool（对外名：Write）

覆盖或创建文件。比 Edit 更具破坏性，`isDestructive` 为真。权限更严，常需明确允许。用于生成新文件或全文件重写。与 Edit 分工：能补丁则补丁，避免无谓整文件覆盖导致 diff 噪音与冲突。

实现落点：`src/tools/FileWriteTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.14 GlobTool（对外名：Glob）

按 glob 找文件。限制 `globLimits.maxResults`。只读。常与 Grep、RepoMap 一起构成“先定位后阅读”的探索路径。

实现落点：`src/tools/GlobTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.15 GrepTool（对外名：Grep）

基于 ripgrep 的内容搜索。输入含 pattern、路径、glob、类型、行上下文。结果相对化路径。尊重 ignore 与插件缓存排除。是只读、并发安全、可折叠为 search 的工具。无嵌入式搜索二进制时才注册；Ant 原生构建若嵌入 ugrep 则可能从工具池去掉 Grep/Glob。

实现落点：`src/tools/GrepTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.16 LSPTool（对外名：LSP）

语言服务器桥：跳转、诊断、补全。未连接服务器时从可用工具过滤（`tools.lsp.test.ts`）。`/lsp` 命令负责安装插件、推荐、重启。

实现落点：`src/tools/LSPTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.17 ListMcpResourcesTool（对外名：ListMcpResources）

列出 MCP 资源。特殊工具，不总出现在默认可见池，按连接动态加入。

实现落点：`src/tools/ListMcpResourcesTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.18 MCPTool（对外名：MCP 动态工具）

每个 MCP server tool 被包装成 `Tool`，name 通常 `mcp__server__tool`。可带 `_meta['anthropic/alwaysLoad']`。

实现落点：`src/tools/MCPTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.19 McpAuthTool（对外名：McpAuth）

MCP 服务器鉴权辅助。

实现落点：`src/tools/McpAuthTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.20 MonitorTool（对外名：Monitor）

MONITOR_TOOL：监控任务/会话。

实现落点：`src/tools/MonitorTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.21 NotebookEditTool（对外名：NotebookEdit）

编辑 Jupyter notebook 单元格。searchHint 含 jupyter，便于 ToolSearch 发现。

实现落点：`src/tools/NotebookEditTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.22 OverflowTestTool（对外名：OverflowTest）

OVERFLOW_TEST_TOOL：专门测试溢出与截断，不面向最终用户。

实现落点：`src/tools/OverflowTestTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.23 PowerShellTool（对外名：PowerShell）

Windows 首选 shell。独立权限与 git safety。由 `isPowerShellToolEnabled()` 决定是否注册。

实现落点：`src/tools/PowerShellTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.24 REPLTool（对外名：REPL）

把 Bash/Read/Edit 等包进 VM。开启后隐藏 primitive，模型只见 REPL。简单模式与 coordinator 模式有特殊组合。

实现落点：`src/tools/REPLTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.25 ReadMcpResourceTool（对外名：ReadMcpResource）

读取 MCP 资源内容。

实现落点：`src/tools/ReadMcpResourceTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.26 RemoteTriggerTool（对外名：RemoteTrigger）

AGENT_TRIGGERS_REMOTE：触发远程 Agent。

实现落点：`src/tools/RemoteTriggerTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.27 RepoMapTool（对外名：RepoMap）

仓库结构地图，给模型一张“地图”而不是整库原文。`/repomap` 命令配置它。构建有超时 `runRepoMapBuildWithTimeout`。测试 `context.repoMap.test.ts` 验证 feature flag 关闭时返回 null。

实现落点：`src/tools/RepoMapTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.28 ReviewArtifactTool（对外名：ReviewArtifact）

审阅产物，用于 review 工作流。

实现落点：`src/tools/ReviewArtifactTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.29 ScheduleCronTool（对外名：CronCreate/Delete/List）

本地调度。创建/删除/列出 cron。触发时产生 `scheduled_task_fire` 系统消息。

实现落点：`src/tools/ScheduleCronTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.30 SendMessageTool（对外名：SendMessage）

向 teammate 发消息。关机中断有专门 trace 测试，避免协调器死锁。

实现落点：`src/tools/SendMessageTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.31 SendUserFileTool（对外名：SendUserFile）

KAIROS：向用户侧发送文件。

实现落点：`src/tools/SendUserFileTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.32 SkillTool（对外名：Skill）

加载并执行 Skill。Skill 来自 bundled、项目、插件、MCP。测试覆盖 prompt 与调用。与斜杠命令技能共享 `getSlashCommandToolSkills`。

实现落点：`src/tools/SkillTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.33 SleepTool（对外名：Sleep）

PROACTIVE/KAIROS 特性：主动等待，用于轮询类工作流。

实现落点：`src/tools/SleepTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.34 SnipTool（对外名：Snip）

HISTORY_SNIP：从上下文删除一段消息并写入 snip_boundary。

实现落点：`src/tools/SnipTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.35 SuggestBackgroundPRTool（对外名：SuggestBackgroundPR）

建议把长任务转成后台 PR 工作流。当前 `tools.ts` 中常量为 null，属特性裁剪。

实现落点：`src/tools/SuggestBackgroundPRTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.36 SyntheticOutputTool（对外名：SyntheticOutput）

强制结构化输出，配合 `--json-schema` 与 hook 执行。

实现落点：`src/tools/SyntheticOutputTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.37 TaskCreateTool（对外名：TaskCreate）

Todo v2：创建任务对象，支持后台与通知。

实现落点：`src/tools/TaskCreateTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.38 TaskGetTool（对外名：TaskGet）

按 id 取任务详情与输出。

实现落点：`src/tools/TaskGetTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.39 TaskListTool（对外名：TaskList）

列出任务。与 `/tasks` UI 同源数据。

实现落点：`src/tools/TaskListTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.40 TaskOutputTool（对外名：TaskOutput）

拉取任务输出流。带 activity 测试。

实现落点：`src/tools/TaskOutputTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.41 TaskStopTool（对外名：TaskStop）

停止后台任务或子 Agent。协调器模式下几乎总是可用。

实现落点：`src/tools/TaskStopTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.42 TaskUpdateTool（对外名：TaskUpdate）

更新任务状态、说明、进度。

实现落点：`src/tools/TaskUpdateTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.43 TeamCreateTool（对外名：TeamCreate）

在 Agent Swarm 开启时创建 teammate 团队。

实现落点：`src/tools/TeamCreateTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.44 TeamDeleteTool（对外名：TeamDelete）

解散团队并清理 mailbox。

实现落点：`src/tools/TeamDeleteTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.45 TerminalCaptureTool（对外名：TerminalCapture）

TERMINAL_PANEL：捕获终端面板内容给模型。

实现落点：`src/tools/TerminalCaptureTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.46 TodoWriteTool（对外名：TodoWrite）

写入会话待办。结果主要反映在待办面板而非 transcript。是长任务规划的短时记忆。

实现落点：`src/tools/TodoWriteTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.47 ToolSearchTool（对外名：ToolSearch）

当工具太多时，主 prompt 只带核心工具，其余 `shouldDefer`。模型先搜工具再调用。`alwaysLoad` 可豁免。这是对抗 schema token 膨胀的关键算法。

实现落点：`src/tools/ToolSearchTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.48 TungstenTool（对外名：Tungsten）

实时监视工具，`TungstenLiveMonitor` 推送进度。注释标明其 outputSchema 曾为可选特例。

实现落点：`src/tools/TungstenTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.49 VerifyPlanExecutionTool（对外名：VerifyPlanExecution）

环境变量 `CLAUDE_CODE_VERIFY_PLAN=true` 时验证计划执行是否落地。

实现落点：`src/tools/VerifyPlanExecutionTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.50 WebBrowserTool（对外名：WebBrowser）

特性 WEB_BROWSER_TOOL 开启时提供浏览器面板，用于需要交互的页面，而不是一次性 fetch。

实现落点：`src/tools/WebBrowserTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.51 WebFetchTool（对外名：WebFetch）

抓取 URL 并转成模型可读文本（turndown 等）。域名检查、预批准列表、prompt 失败回退。open-world。

实现落点：`src/tools/WebFetchTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.52 WebSearchTool（对外名：WebSearch）

多供应商搜索。供应商适配器实现统一 types：查询、超时、结果条目。可配置自定义端点。UI 展示进度 `WebSearchProgress`。

实现落点：`src/tools/WebSearchTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。

### 9.53 WorkflowTool（对外名：Workflow）

WORKFLOW_SCRIPTS：运行打包工作流脚本，有独立权限请求 UI。

实现落点：`src/tools/WorkflowTool/`。典型函数集合包括 `call`（执行）、`checkPermissions`（授权）、`mapToolResultToToolResultBlockParam`（模型可见输出）、`renderToolUseMessage`（进行中 UI）、`renderToolResultMessage`（结果 UI）、`isReadOnly`/`isConcurrencySafe`（调度分类）、`prompt`（系统提示段落）。若该工具会碰文件系统，还应实现 `getPath` 以便权限与 transcript 显示绝对/相对路径。若该工具是搜索/读取类，应实现 `isSearchOrReadCommand` 以允许 UI 折叠。若该工具面向开放网络，应实现 `isOpenWorld` 以便默认询问。


### 9.x 工具结果预算算法

`applyToolResultBudget`（`src/utils/toolResultStorage.ts`）在一轮工具全部完成后，按字符预算把过大结果替换为磁盘预览。状态放在 `contentReplacementState`。主线程 REPL 只供应一次、不重置，过期 UUID 无害；子 Agent 默认克隆父状态以保持 cache 决策一致。`syncToolResultReplacements` 把替换镜像回活 transcript，释放堆上的原始大字符串。预览测试见 `toolResultStorage.preview.test.ts`。

### 9.y 工具 schema 缓存

`toolSchemaCache.ts` 缓存已序列化的 JSON Schema。工具被移除时 `invalidateRemovedToolSchemas`。SDK 测试 `tool-schema-cache.test.ts` 防止泄漏与错误命中。该缓存直接服务于 prompt cache：系统提示里的工具列表不稳定会导致整段 cache 失效，成本剧增。


## 10. 斜杠命令与 CLI 子命令详细设计

`src/commands.ts` 把命令模块装配成表，并处理：内部命令、可用性、memoization、MCP 技能命令、远程安全命令、桥接安全命令。`findCommand` / `hasCommand` / `getCommand` 是查询 API。`formatDescriptionWithSource` 在帮助里标注命令来自内置、插件还是 MCP。

命令实现通常导出默认对象，字段包括 name、description、argumentHint、isEnabled、本地执行函数或 JSX 面板。本地命令的 stdout/stderr 用 `LOCAL_COMMAND_STDOUT_TAG` / `STDERR_TAG` 包起来再进上下文。

### 10.1 `/add-dir`

增加工作目录。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.2 `/ads`

赞助提示换积分。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.3 `/advisor`

顾问模型。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.4 `/agents`

管理 Agent 定义，含创建向导（颜色、工具、模型、记忆、位置）。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.5 `/auto-fix`

编辑后自动跑 lint/test。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.6 `/autofix-pr`

PR 自动修复。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.7 `/backfill-sessions`

回填会话索引。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.8 `/branch`

在当前点分叉对话。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.9 `/break-cache`

故意打破 cache 以调试。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.10 `/bridge`

桥接模式。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.11 `/brief`

任务简报。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.12 `/btw`

旁路提问，不打断主对话。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.13 `/buddy`

像素伙伴：孵化、命名、静音。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.14 `/bughunter`

四阶段猎虫：map → hunt → skeptic → fix proposals。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.15 `/bughunter-perf`

性能向猎虫：热路径、同步 IO、泄漏、N+1。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.16 `/bughunter-security`

安全向猎虫，OWASP，置信度阈值。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.17 `/cache-probe`

探测 prompt cache。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.18 `/cache-stats`

跨供应商的 cache hit/miss。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.19 `/chrome`

Chrome 集成（Beta）。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.20 `/clear`

清空对话并释放上下文。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.21 `/clear-context-window`

清除覆盖。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.22 `/color`

提示条颜色。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.23 `/commit`

生成并执行提交。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.24 `/commit-message`

提交归因文案。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.25 `/commit-push-pr`

`/commit-push-pr` 对应 `src/commands/` 下同名模块。它是本地控制面操作：修改会话状态、打开 Ink 面板、或触发一次受控副作用（登录、诊断、写配置），而不是模型工具。是否在帮助中展示取决于特性开关、用户类型与远程/桥接安全名单。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.26 `/compact`

手动压缩历史，可附自定义摘要指令。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.27 `/config`

配置面板。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.28 `/context`

上下文占用。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.29 `/copy`

复制最近回复。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.30 `/cost`

本会话成本与时长。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.31 `/ctx`

token 分解可视化。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.32 `/desktop`

切到 Desktop。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.33 `/diagnostics`

已捕获 LSP 诊断。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.34 `/diff`

未提交变更与按回合 diff。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.35 `/doctor`

诊断安装、Node、ripgrep、配置路径。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.36 `/dream`

记忆巩固，把近场会话提炼成长期记忆。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.37 `/effort`

设置推理努力档位：low/medium/high/xhigh/max/ultracode/auto。受模型能力门控。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.38 `/exit`

退出 REPL。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.39 `/export`

导出对话到文件或剪贴板。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.40 `/extra-usage`

超限后的额外用量。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.41 `/fast`

快速模式。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.42 `/feedback`

反馈。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.43 `/files`

列出当前在上下文中的文件。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.44 `/goal`

会话完成条件。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.45 `/help`

命令帮助。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.46 `/hooks`

查看工具事件 hook 配置。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.47 `/ide`

IDE 集成。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.48 `/init`

生成项目指令文件。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.49 `/install-github-app`

GitHub Actions 集成。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.50 `/install-slack-app`

Slack 应用。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.51 `/insights`

会话分析报告。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.52 `/issue`

问题报告。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.53 `/keybindings`

打开或创建键位文件。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.54 `/knowledge`

原生知识图谱 enable/clear/status/list。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.55 `/login`

Anthropic 账户登录。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.56 `/logo`

启动 logo 配色。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.57 `/logout`

登出。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.58 `/lsp`

LSP 状态/推荐/安装/重启。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.59 `/mcp`

启用/禁用/诊断 MCP 服务器。CLI 子命令另有 add/remove/list/doctor。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.60 `/memory`

编辑持久记忆文件。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.61 `/mobile`

移动端下载二维码。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.62 `/model`

设置本会话模型，接受别名（sonnet/opus）或全名。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.63 `/onboard-github`

GitHub Models/Copilot 设备流登录，凭据进安全存储。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.64 `/output-style`

输出风格。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.65 `/passes`

passes 相关。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.66 `/peers`

UDS 对等体。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.67 `/permissions`

交互式编辑 allow/deny。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.68 `/plan`

进入或查看计划模式。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.69 `/plugin`

插件与市场：validate/list/install/enable/marketplace。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.70 `/pr-comments`

拉 GitHub PR 评论。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.71 `/privacy-settings`

隐私。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.72 `/provider`

打开提供商向导，写入 `.openclaude-profile.json`。这是官方推荐的凭据配置方式，而不是依赖项目 .env。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.73 `/rate-limit-options`

限流选项。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.74 `/release-notes`

发行说明。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.75 `/reload-plugins`

激活待定插件变更。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.76 `/remote-env`

远程环境。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.77 `/remote-setup`

远程/CCR 设置。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.78 `/rename`

重命名会话。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.79 `/replay`

重放工具时间线。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.80 `/repomap`

显示或配置仓库地图。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.81 `/request-size`

请求体贡献者排名。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.82 `/resume`

按 id 或搜索恢复会话。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.83 `/review`

审阅 PR。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.84 `/rewind`

把代码和/或对话恢复到先前点，依赖 file history。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.85 `/sandbox-toggle`

切换沙箱。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.86 `/security-review`

对当前分支未提交变更做安全审查。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.87 `/session`

远程会话 URL 与二维码。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.88 `/set-context-window`

会话级窗口覆盖。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.89 `/share`

分享会话。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.90 `/skills`

列出技能；CLI 另有 show/validate/install/remove。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.91 `/smartroute`

把简单回合路由到 simple 模型。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.92 `/stats`

使用统计。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.93 `/status`

版本、模型、账号、API、工具状态。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.94 `/statusline`

状态行。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.95 `/stickers`

贴纸。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.96 `/tag`

会话标签。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.97 `/tasks`

后台任务列表。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.98 `/teleport`

会话迁移到其它环境。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.99 `/terminal-setup`

安装 Shift+Enter 换行。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.100 `/theme`

主题。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.101 `/thinkback`

回顾思考轨迹。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.102 `/ultraplan`

加强版规划（特性）。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.103 `/update`

自更新。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.104 `/upgrade`

升级流程 UI。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.105 `/usage`

计划用量。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.106 `/version`

版本。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.107 `/vim`

切换 Vim 编辑模式。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.108 `/voice`

语音模式（特性开关）。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.109 `/wiki`

项目 wiki 的 init/status/scan/ingest。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。

### 10.110 `/workflows`

工作流脚本。

设计要点：命令必须是幂等或可取消的；需要用户确认的操作应使用 Dialog 组件而不是直接写磁盘；需要调用模型的命令（如 rename 的自动起名、review）应复用 QueryEngine 而不是自己 new 客户端；远程模式只能暴露 `REMOTE_SAFE_COMMANDS`，桥接模式只能暴露 `BRIDGE_SAFE_COMMANDS`。相关测试位于命令目录的 `*.test.ts` 或 `src/commands.test.ts`。


### 10.x CLI 子命令（非 REPL）

`openclaude mcp`、`openclaude auth`、`openclaude auth xai`、`openclaude skills`、`openclaude plugin` 以及后台 `ps/logs/kill`、`aimlapi topup` 等在 `src/cli/` 注册。`--print` 模式下这些子命令被跳过，以避免脚本路径分叉。`src/cli/bgRouting.ts` 用标记位识别后台子进程。


## 11. 关键算法说明书
### 11.1 自动压缩（autoCompact）
`src/services/compact/autoCompact.ts` 计算有效上下文：`contextWindow - min(maxOutputTokens, 20000)`，再与下限 `reserved + AUTOCOMPACT_FLOOR_BUFFER_TOKENS` 取 max，避免阈值变负导致每条消息都压缩（issue #635）。阈值函数 `getAutoCompactThreshold()` 使用更大的 `AUTOCOMPACT_BUFFER_TOKENS`。环境变量 `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 可人为缩小窗口以便测试。连续失败计入 `consecutiveFailures`，超过 `MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES` 进入冷却，防止 prompt_too_long 风暴。成功后 `markPostCompaction`、`notifyCompaction` 打断 prompt cache 的错误复用，并 `setLastSummarizedMessageId`。
压缩本体 `compactConversation` 把历史分区（`partitionContext`）、按相关性剪枝（`pruneByRelevance` / `normalizeCompactTailTurns`）、调用模型写摘要、插入 `compact_boundary`、跑 `runPostCompactCleanup`。可选 `trySessionMemoryCompaction` 把可持久内容写入 SessionMemory。手动 `/compact` 与自动路径共享实现，trigger 字段不同。SDK 测试要求手动 compact 清掉过期冷却。
### 11.2 微压缩（microCompact）
不写摘要，只把已完成工具的大结果清掉，保留 tool_use 骨架，写入 `microcompact_boundary`。适合“工具结果占满窗口但对话结构仍需要”的情况。比全量 compact 更便宜、更可逆。
### 11.3 上下文折叠（contextCollapse）
特性 CONTEXT_COLLAPSE：把一段历史替换为 `<collapsed>` 摘要用户消息，并打 `isCollapseSummary`，禁止 snip。与 compact 的区别是面向 UI/模型的“折叠”语义，而不是完整的摘要 API 往返。
### 11.4 Token 估计与预算
`tokenCountWithEstimation`、`finalContextTokensFromLastResponse`、`doesMostRecentAssistantMessageExceed200k` 结合真实 usage 与启发式。`getCurrentTurnTokenBudget` / `getTurnOutputTokens` 限制单回合输出。`tokenBudget.ts` 与 `tokens.ts` 测试覆盖边界。自定义 `modelLimits` 与环境变量 `CLAUDE_CODE_OPENAI_CONTEXT_WINDOWS` 覆盖目录中缺失的开源模型窗口。
### 11.5 权限规则匹配
解析器把 `Bash(git *)` 变成 toolName=Bash、content=`git *`。匹配时对命令做规范化（去掉无关注释、处理引号）。通配遵循 shell 语义而非任意正则，降低规则意外过宽。MCP 前缀匹配在列表过滤阶段就生效。被遮蔽的规则由 `shadowedRuleDetection.ts` 提示用户。
### 11.6 Smart routing
把回合分类为 simple/strong，路由到 `smartRouting.simpleModel` / `strongModel`。可用 `/smartroute` 或环境变量 `OPENCLAUDE_SMART_ROUTING*`。出错则回退 strong。这是成本算法，不是正确性算法：分类错误的代价是质量下降，因此默认关闭、需 opt-in。
### 11.7 Prompt cache 友好布局
系统提示被刻意拆成稳定前缀（工具 schema、静态指令）与不稳定后缀（git status、日期、动态记忆）。`tengu` 相关缓存配置要求 `getAllBaseTools` 的顺序与内容跨用户稳定。`cache-probe` 命令发送相同请求以观测命中。`break-cache` 用于调试。cache 指标在 `cacheMetrics.ts` / `cacheStatsTracker.ts`。
### 11.8 图像处理
检测格式、量尺寸、按 token 上限压缩、必要时 downsample。失败分类为 `ImageSizeError`、`ImageResizeError`、`ImageProcessorUnavailableError`。粘贴路径 `Ctrl+V`（Windows `Alt+V`）若超过 5MB/8000px，提示具体错误而不是“没有图片”。
### 11.9 会话命名与检索
`generateSessionName` 可调用模型起名。`/resume` 用 Fuse.js 模糊搜。tag 可检索。transcript 搜索必须与渲染保真一致。
### 11.10 Doom loop / 失败循环守卫
`doomLoop.ts` 与 `toolFailureLoopGuard` 检测模型在同一错误上原地打转。超过阈值插入系统提醒或停止，避免无限花销。
### 11.11 QueryGuard 空闲与硬超时
活动租约：工具运行、流式输出、用户对话框都会续命。`beginUserInteraction` 在等人时暂停空闲钟，但租约截止仍生效，防止对话框被挂起到永远。
### 11.12 Repo map 构建
扫描仓库结构、语言分布、重要入口文件，生成给模型的压缩地图。有超时，失败则退回无地图模式，不能阻塞启动。
### 11.13 技能检索
实验性 `skillSearch`：本地 Orama 索引、信号、预取、远程技能状态。`clearSkillIndexCache` 在相关命令里暴露。
### 11.14 OAuth 与凭据池
Anthropic、Codex、xAI、GitHub 各有 OAuth。xAI 校验 OAuth state 再 settle callback（#2228）。`credentialPool` 对 `OPENAI_API_KEYS` 轮换。钥匙串预取避免启动时同步 spawn。
### 11.15 工作树与多仓库
`worktree.ts` 创建隔离树；`agentBase` 计算 Agent 根；多仓库父目录有专门 fixture 测试。fork-session 明确不创建 worktree，文档与实现一致，防止用户误以为代码也被分叉。
### 11.16 流看门狗
`claude.streamWatchdog` 在 SSE 静默过久时中止，避免“假死”占用超时预算。first-token 日期与 TTFT 写入 api_metrics。
### 11.17 结构化输出强制
`registerStructuredOutputEnforcement` 与 SyntheticOutput 工具保证 `--json-schema` 在模型想结束时仍能被纠正。
### 11.18 相关性剪枝
压缩前对旧工具结果按与当前用户目标的相关性打分，丢掉高噪声低价值块，保留最近尾部回合。这是自动压缩质量的核心，而不是简单“删掉最老的 N 条”。
### 11.19 提供商发现
`discoveryService.ts` 对网关做公开模型发现并缓存。LLMTR 等用发现结果决定默认 tool-capable 模型。测试覆盖 cache 与各网关。
### 11.20 启动性能算法
并行 MDM、钥匙串、GrowthBook、Ollama 模型列表、MCP 官方 URL、AWS/GCP 凭证预取（仅在判定安全时）。`startupProfiler` 打点。`--bare` 跳过大多数预取。


## 12. Provider 集成架构

集成系统的铁律见 `docs/integrations/overview.md`：元数据、路由、运输分离。描述符不得发 HTTP；运输层不得散落品牌文案。`transportConfig.kind` 决定走哪家客户端。

下列是当前文档站与清单中的主要路由及其设计角色。每一条都对应 `src/integrations/vendors/` 或 `gateways/` 或 OAuth 特例。

### 12.1 `anthropic` — Anthropic Claude

类型：订阅/API Key。官方 Messages API、OAuth 登录与 API Key 直连。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.2 `codex-oauth` — Codex OAuth / ChatGPT

类型：订阅。浏览器登录 ChatGPT，走 Responses API，覆盖 GPT-5.6 家族。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.3 `xai-oauth` — xAI Grok

类型：OAuth/API Key。浏览器 OAuth 或设备码，适配远程主机。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.4 `github-models` — GitHub Models / Copilot

类型：Token/OAuth。/onboard-github 引导，支持 Copilot Enterprise。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.5 `kimi-code` — Moonshot Kimi Code

类型：订阅。Kimi K3 1M/256K 上下文变体。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.6 `zai` — Z.AI GLM Coding Plan

类型：订阅。默认 glm-5.2。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.7 `dashscope` — 阿里云百炼 Coding Plan

类型：API Key。国际站与中国站，默认 qwen3.6-plus。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.8 `xiaomi-mimo-token` — Xiaomi MiMo Token Plan

类型：订阅。默认 mimo-v2.5-pro。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.9 `opengateway` — Gitlawb Opengateway

类型：网关。智能路由，聚合 MiMo/MiniMax/Qwen/GLM/Gemini 与免费模型。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.10 `aimlapi` — AI/ML API

类型：网关。一千以上模型，CLI 内充值与发钥。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.11 `openrouter` — OpenRouter

类型：网关。OpenAI 兼容聚合。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.12 `llmtr` — LLMTR

类型：网关。默认 deepseek-v4-flash。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.13 `novita` — Novita AI

类型：网关。合作网关。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.14 `atlas-cloud` — Atlas Cloud

类型：网关。合作网关。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.15 `apismart` — ApiSmart

类型：网关。默认 DEEPSEEK_V4_FLASH。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.16 `concentrate` — Concentrate

类型：网关。合作网关。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.17 `openai` — OpenAI

类型：厂商 API。官方 OpenAI 与兼容端点。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.18 `gemini` — Google Gemini

类型：厂商 API。GEMINI_API_KEY，含 Vertex 路径。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.19 `deepseek` — DeepSeek

类型：厂商 API。DeepSeek Chat/Reasoner。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.20 `minimax` — MiniMax

类型：厂商 API。含 usage 解析。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.21 `xai` — xAI 直连

类型：厂商 API。XAI_API_KEY。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.22 `mistral` — Mistral

类型：厂商 API。Mistral 与相关网关。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.23 `fireworks` — Fireworks

类型：厂商 API。托管开源模型。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.24 `together` — Together

类型：网关。托管推理。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.25 `groq` — Groq

类型：网关。低延迟推理。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.26 `nvidia-nim` — NVIDIA NIM

类型：网关。NVIDIA_API_KEY。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.27 `cloudflare` — Cloudflare Workers AI

类型：网关。CLOUDFLARE_API_TOKEN。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.28 `bedrock` — Amazon Bedrock

类型：云路由。STS/凭证链。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.29 `vertex` — Google Vertex AI

类型：云路由。GCP 凭证。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.30 `azure-openai` — Azure OpenAI

类型：云路由。Azure Identity。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.31 `ollama` — Ollama

类型：本地。本机或局域网 OpenAI 兼容。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.32 `lmstudio` — LM Studio

类型：本地。本机兼容端点。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.33 `atomic-chat` — Atomic Chat

类型：网关。合作伙伴路由。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。

### 12.34 `custom` — 自定义端点

类型：自建。任意 OpenAI 兼容 /v1。接入时需要：描述符（显示名、默认模型、环境变量、setup 提示）、可选模型目录（上下文窗口、是否支持 reasoning effort、是否支持工具）、运输映射（OpenAI 兼容则走 `openaiShim`，Anthropic 走 `claude.ts`，Codex 走 `codexShim` + Responses API）。测试应覆盖：缺密钥时的错误信息、默认模型、与 `/provider` 向导字段一致、以及至少一条 chat+tool 的契约测试（见 `bun run test:provider`）。


### 12.x openaiShim 设计

`src/services/api/openaiShim.ts` 把内部 Anthropic 风格的消息/工具转成 Chat Completions 或 Responses。子模块：

- `clientDispatch.ts`：选客户端；
- `codexDispatch.ts`：Codex 特殊分发；
- schema sanitizer：去掉不兼容字段；
- Ollama 文本 tool call 解析（有模型把工具调用写在文本里）；
- 压缩与诊断测试。

这是兼容 200+ 模型的核心适配器，回归必须谨慎：一次 schema 误删会导致某家模型彻底不能调工具。

### 12.y 错误分类

`openaiErrorClassification.ts`、`errors.openaiCompatibility.test.ts`、`errors.opencodeGo.test.ts` 把各家 JSON 错误映射到可重试、额度、鉴权、上下文过长。UI 文案与 fallback 链都依赖这张表。


## 13. 终端 UI、VS Code 扩展与文档站
### 13.1 Ink 应用壳
`src/components/App.tsx` 是 REPL 根。设计系统在 `components/design-system/`：ThemedBox、Dialog、Tabs、FuzzyPicker、ProgressBar、ThemeProvider。权限对话框、MCP 批准、成本阈值、IDE 连接、导出、历史搜索都是独立 Dialog。Markdown 渲染支持表格与代码高亮。`StatusLine` 可配置。Buddy 在用户按 Enter 时射箭，是故意的产品个性，但不得影响查询正确性。
### 13.2 键位
默认键位在 `src/keybindings/defaultBindings.ts`，用户覆盖 `~/.openclaude/keybindings.json`。Ctrl+C 中断、Ctrl+D 退出、Ctrl+L 重绘、Ctrl+T 待办、Ctrl+O transcript、Ctrl+R 历史搜索、Shift+Tab 循环权限模式、Ctrl+V 粘贴图、Ctrl+S 暂存草稿、Ctrl+G 外部编辑器。Vim 模式有 motions/operators/textObjects/transitions。
### 13.3 VS Code 扩展
`vscode-extension/openclaude-vscode/` 用 JavaScript 实现：`extension.js` 激活、`chatProvider`、`sessionManager`、`processManager`、`protocol`、`diffController`、`permissionResponse`、`messageParser`、`chatRenderer`。它不重新实现 Agent，而是拉起 CLI 进程并用协议桥接聊天与 diff。测试覆盖 state、presentation、permissionResponse。主题 `OpenClaude-Terminal-Black.json`。
### 13.4 文档站 `web/`
静态站点，数据源与 CLI 同步：commands.ts、cliFlags.ts、providers.ts、configuration.ts、keybindings.ts、skills.ts。构建后 `verify-dist` 测试防止文档漂移。发行说明不在站点维护，而指向 GitHub Releases。
## 14. MCP、插件、技能、记忆
### 14.1 MCP
`src/services/mcp/` 管理连接、工具包装、资源、鉴权、官方注册表预取。`src/entrypoints/mcp.ts` 也可让 OpenClaude 自身作为 MCP 服务器被别人连。测试覆盖 cleanup、SDK MCP tools。Elicitation（-32042）在 print/SDK 走 structuredIO，在 REPL 走队列 UI。
### 14.2 插件
市场、安装、信任警告、校验、热加载。Windows 上 marketplace cache 拷贝 ENOENT 有专门修复（#2220）。`reload-plugins` 在会话内激活待定变更。
### 14.3 技能
`src/skills/loadSkillsDir.ts` 扫描技能目录；bundled 技能包括 batch、simplify、debug、pdf、update-config。条件技能可在读到某些路径时激活。`/skills` 与 CLI `openclaude skills` 是两套入口同一加载器。
### 14.4 记忆
memdir 提供自动记忆路径；SessionMemory 在 compact 时提炼；`/dream` 做跨会话巩固；`/knowledge` 管知识图谱；`/wiki` 管项目 wiki ingest。teamMemorySync 监视团队记忆并扫描密钥，防止把 secret 写进共享记忆。
## 15. SDK 设计
`src/entrypoints/sdk/index.ts` 导出 query、sessions、permissions、agentDefinitions、v2 API。设计目标：嵌入方不必启动 Ink。权限通过回调，输出通过异步迭代器。测试目录 `tests/sdk/` 覆盖 happy path、lifecycle、concurrency、context isolation、casing、factories、engine mutators、generated types、package consumer types。`stubLeakDetection` 防止测试桩泄漏进产物。


## 16. 核心函数清单与行为契约
以下函数是维护时最常碰到的契约点。修改前应阅读对应测试。
### 16.1 启动与配置
`main`：CLI 总入口。`startDeferredPrefetches`：启动后延迟预取。`init` / `initializeTelemetryAfterTrust`：信任对话框之后才初始化遥测。`getGlobalConfig` / `saveGlobalConfig`：全局配置。`applyConfigEnvironmentVariables` / `applySafeConfigEnvironmentVariables`：把 settings.env 注入进程，后者更保守。`checkHasTrustDialogAccepted`：工作区信任。
### 16.2 命令
`getCommands`、`clearCommandsCache`、`findCommand`、`hasCommand`、`getCommand`、`filterCommandsForRemoteMode`、`isBridgeSafeCommand`、`getSlashCommandToolSkills`、`meetsAvailabilityRequirement`。
### 16.3 工具池
`getAllBaseTools`、`getTools`、`assembleToolPool`、`getMergedTools`、`filterToolsByDenyRules`、`parseToolPreset`、`getToolsForDefaultPreset`、`toolMatchesName`、`findToolByName`、`buildTool`。
### 16.4 查询
`query`、`buildQueryConfig`、`createQueryTurnBudget`、`runTools`、`handleStopHooks`、`createToolFailureLoopGuardState`、`updateToolFailureLoopGuard`、`processUserInput`、`fetchSystemPromptParts`。
### 16.5 消息
`createUserMessage`、`createAssistantMessage`、`createSystemMessage`、`createUserInterruptionMessage`、`createAssistantAPIErrorMessage`、`createToolUseSummaryMessage`、`createMicrocompactBoundaryMessage`、`normalizeMessagesForAPI`、`getMessagesAfterCompactBoundary`、`isHumanTurn`。
### 16.6 压缩
`getEffectiveContextWindowSize`、`getAutoCompactThreshold`、`isAutoCompactEnabled`、`compactConversation`、`buildPostCompactMessages`、`runPostCompactCleanup`、`trySessionMemoryCompaction`、`partitionContext`、`pruneByRelevance`。
### 16.7 成本
`addToTotalSessionCost`、`formatTotalCost`、`getTotalCost`、`getModelUsage`、`accumulateUsage`、`updateUsage`、`resetCostState`、`restoreCostStateForSession`。
### 16.8 权限
`checkPermissions`（工具方法）、通用 `permissions.ts` 导出的规则匹配与模式切换、`getNextPermissionMode`、`getDenyRuleForTool`、`checkReadPermissionForTool`、`matchWildcardPattern`。
### 16.9 模型
`getMainLoopModel`、`parseUserSpecifiedModel`、`getProviderRequestModel`、`getRuntimeMainLoopModel`、`renderModelName`、`getContextWindowForModel`、`getMaxOutputTokensForModel`。
### 16.10 集成
`src/integrations/registry.ts` 的查找与枚举函数、`discoveryService` 刷新、`setActiveProviderProfile`、`getActiveProviderProfile`、`getPrimaryModel`、`resolveNextFallbackProviderFromState`。
这些函数的错误语义必须稳定：对模型可见的错误字符串是接口的一部分，随意改文案会破坏依赖提示语的 Agent 行为，也会破坏快照测试。


## 17. 测试设计与用例说明书

当前源码树中测试文件约 739 个，测试代码行数合计超过二十万行。测试运行器是 `bun test`，默认 `--feature=UNATTENDED_RETRY --max-concurrency=1`。`test:full` 额外开启 CONVERSATION_ARC / MULTI_TURN_CONTEXT。`test:provider` 聚焦运输层。`test:provider-recommendation` 聚焦推荐与 profile。

测试分层：

1. **契约测试：** 权限模式集合、工具池成员、SDK 类型生成、集成产物 `--check`。

2. **算法测试：** autocompact 冷却、自定义定价预算、token 格式化、规则匹配。

3. **回归测试：** 中断分类、OAuth state、Windows 插件缓存、图片缩放失败、prompt_too_long。

4. **UI 测试：** Ink 组件的渲染与交互（provider、model、agents wizard）。

5. **SDK 测试：** `tests/sdk/` 独立于 REPL。

6. **脚本测试：** doctor、privacy、externals、heap、compile cache。

下面按文件列出设计意图。每个测试文件守护一组不变量：当生产代码重构时，应先扩展这些用例再改实现。

### 17.1 `bin/import-specifier.test.mjs`

该文件位于 `bin`，约 13 行，覆盖主题「import-specifier.test.mjs」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.2 `scripts/externalsValidation.test.ts`

该文件位于 `scripts`，约 344 行，覆盖主题「externalsValidation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.3 `scripts/feature-flags-source-guard.test.ts`

该文件位于 `scripts`，约 48 行，覆盖主题「feature-flags-source-guard」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.4 `scripts/missing-module-stub.test.ts`

该文件位于 `scripts`，约 60 行，覆盖主题「missing-module-stub」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.5 `scripts/no-ant-employee-gates.test.ts`

该文件位于 `scripts`，约 137 行，覆盖主题「no-ant-employee-gates」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.6 `scripts/no-raw-abort-signal-timeout.test.ts`

该文件位于 `scripts`，约 118 行，覆盖主题「no-raw-abort-signal-timeout」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.7 `scripts/no-telemetry-growthbook-stub.test.ts`

该文件位于 `scripts`，约 171 行，覆盖主题「no-telemetry-growthbook-stub」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.8 `scripts/openclaude-bin-compile-cache.test.ts`

该文件位于 `scripts`，约 204 行，覆盖主题「openclaude-bin-compile-cache」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.9 `scripts/openclaude-bin-heap.test.ts`

该文件位于 `scripts`，约 318 行，覆盖主题「openclaude-bin-heap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.10 `scripts/optionalRuntimeSpecifiers.test.ts`

该文件位于 `scripts`，约 99 行，覆盖主题「optionalRuntimeSpecifiers」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.11 `scripts/pr-intent-scan.test.ts`

该文件位于 `scripts`，约 198 行，覆盖主题「pr-intent-scan」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.12 `scripts/provider-launch.test.ts`

该文件位于 `scripts`，约 94 行，覆盖主题「provider-launch」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.13 `scripts/provider-recommend.test.ts`

该文件位于 `scripts`，约 96 行，覆盖主题「provider-recommend」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.14 `scripts/reactJsxDevRuntimeProductionShim.test.ts`

该文件位于 `scripts`，约 43 行，覆盖主题「reactJsxDevRuntimeProductionShim」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.15 `scripts/stubMarkerGuard.test.ts`

该文件位于 `scripts`，约 87 行，覆盖主题「stubMarkerGuard」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.16 `scripts/system-check.test.ts`

该文件位于 `scripts`，约 1015 行，覆盖主题「system-check」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.17 `scripts/verify-clean-install.test.ts`

该文件位于 `scripts`，约 76 行，覆盖主题「verify-clean-install」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.18 `src/QueryEngine.autoCompactCooldown.test.ts`

该文件位于 `src`，约 41 行，覆盖主题「QueryEngine.autoCompactCooldown」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.19 `src/QueryEngine.customPricingBudget.test.ts`

该文件位于 `src`，约 37 行，覆盖主题「QueryEngine.customPricingBudget」。不变量：自定义价格精确匹配；缺 webSearch 用默认；项目 settings 不得覆盖。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.20 `src/QueryEngine.interruptionTrace.test.ts`

该文件位于 `src`，约 183 行，覆盖主题「QueryEngine.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.21 `src/__tests__/bugfixes.test.ts`

该文件位于 `src/__tests__`，约 715 行，覆盖主题「bugfixes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.22 `src/__tests__/doctorContextWarnings.test.ts`

该文件位于 `src/__tests__`，约 178 行，覆盖主题「doctorContextWarnings」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.23 `src/__tests__/process-title.test.ts`

该文件位于 `src/__tests__`，约 13 行，覆盖主题「process-title」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.24 `src/__tests__/providerCounts.test.ts`

该文件位于 `src/__tests__`，约 55 行，覆盖主题「providerCounts」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.25 `src/__tests__/security-hardening.test.ts`

该文件位于 `src/__tests__`，约 191 行，覆盖主题「security-hardening」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.26 `src/__tests__/statusNoticeLocalModel.test.ts`

该文件位于 `src/__tests__`，约 454 行，覆盖主题「statusNoticeLocalModel」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.27 `src/assistant/sessionHistory.test.ts`

该文件位于 `src/assistant`，约 249 行，覆盖主题「sessionHistory」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.28 `src/bootstrap/state.modelUsageProto.test.ts`

该文件位于 `src/bootstrap`，约 98 行，覆盖主题「state.modelUsageProto」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.29 `src/bridge/initReplBridge.titleTruncation.test.ts`

该文件位于 `src/bridge`，约 62 行，覆盖主题「initReplBridge.titleTruncation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.30 `src/bridge/replBridgeTransport.types.test.ts`

该文件位于 `src/bridge`，约 99 行，覆盖主题「replBridgeTransport.types」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.31 `src/bridge/sessionRunner.test.ts`

该文件位于 `src/bridge`，约 85 行，覆盖主题「sessionRunner」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.32 `src/bridge/workSecret.test.ts`

该文件位于 `src/bridge`，约 104 行，覆盖主题「workSecret」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.33 `src/buddy/CompanionActionFX.test.tsx`

该文件位于 `src/buddy`，约 84 行，覆盖主题「CompanionActionFXx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.34 `src/buddy/CompanionSprite.test.tsx`

该文件位于 `src/buddy`，约 166 行，覆盖主题「CompanionSpritex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.35 `src/buddy/actionEffects.test.ts`

该文件位于 `src/buddy`，约 105 行，覆盖主题「actionEffects」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.36 `src/buddy/companion.test.ts`

该文件位于 `src/buddy`，约 85 行，覆盖主题「companion」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.37 `src/buddy/pixelSprites.test.ts`

该文件位于 `src/buddy`，约 98 行，覆盖主题「pixelSprites」。不变量：未连接即过滤；连接后可用。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.38 `src/buddy/sprites.test.ts`

该文件位于 `src/buddy`，约 108 行，覆盖主题「sprites」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.39 `src/buddy/types.test.ts`

该文件位于 `src/buddy`，约 33 行，覆盖主题「types」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.40 `src/cli/aimlapiCommand.test.ts`

该文件位于 `src/cli`，约 166 行，覆盖主题「aimlapiCommand」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.41 `src/cli/bg.test.ts`

该文件位于 `src/cli`，约 1845 行，覆盖主题「bg」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.42 `src/cli/bgFinalizer.test.ts`

该文件位于 `src/cli`，约 621 行，覆盖主题「bgFinalizer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.43 `src/cli/bgRegistry.test.ts`

该文件位于 `src/cli`，约 1867 行，覆盖主题「bgRegistry」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.44 `src/cli/handlers/aimlapi.test.ts`

该文件位于 `src/cli/handlers`，约 85 行，覆盖主题「aimlapi」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.45 `src/cli/handlers/skills.test.ts`

该文件位于 `src/cli/handlers`，约 1345 行，覆盖主题「skills」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.46 `src/cli/handlers/skillsVerify.test.ts`

该文件位于 `src/cli/handlers`，约 496 行，覆盖主题「skillsVerify」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.47 `src/cli/handlers/xaiAuth.test.ts`

该文件位于 `src/cli/handlers`，约 190 行，覆盖主题「xaiAuth」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.48 `src/cli/headlessHeartbeat.test.ts`

该文件位于 `src/cli`，约 531 行，覆盖主题「headlessHeartbeat」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.49 `src/cli/print.interruptionTrace.test.ts`

该文件位于 `src/cli`，约 108 行，覆盖主题「print.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.50 `src/cli/print.sdkModelOptions.test.ts`

该文件位于 `src/cli`，约 76 行，覆盖主题「print.sdkModelOptions」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.51 `src/cli/printHeartbeat.test.ts`

该文件位于 `src/cli`，约 196 行，覆盖主题「printHeartbeat」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.52 `src/cli/printMaxTurns.test.ts`

该文件位于 `src/cli`，约 223 行，覆盖主题「printMaxTurns」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.53 `src/cli/transports/HybridTransport.test.ts`

该文件位于 `src/cli/transports`，约 207 行，覆盖主题「HybridTransport」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.54 `src/cli/transports/ccrClient.test.ts`

该文件位于 `src/cli/transports`，约 318 行，覆盖主题「ccrClient」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.55 `src/cli/update.test.ts`

该文件位于 `src/cli`，约 61 行，覆盖主题「update」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.56 `src/commands.test.ts`

该文件位于 `src`，约 862 行，覆盖主题「commands」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.57 `src/commands/ads.test.ts`

该文件位于 `src/commands`，约 90 行，覆盖主题「ads」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.58 `src/commands/auto-fix.test.ts`

该文件位于 `src/commands`，约 19 行，覆盖主题「auto-fix」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.59 `src/commands/branch/branch.test.ts`

该文件位于 `src/commands/branch`，约 993 行，覆盖主题「branch」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.60 `src/commands/cache-probe/cache-probe.test.ts`

该文件位于 `src/commands/cache-probe`，约 205 行，覆盖主题「cache-probe」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.61 `src/commands/cacheStats/cacheStats.test.ts`

该文件位于 `src/commands/cacheStats`，约 171 行，覆盖主题「cacheStats」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.62 `src/commands/clear/conversation.goal.test.ts`

该文件位于 `src/commands/clear`，约 49 行，覆盖主题「conversation.goal」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.63 `src/commands/commit-message/commit-message.test.ts`

该文件位于 `src/commands/commit-message`，约 90 行，覆盖主题「commit-message」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.64 `src/commands/ctx_viz/ctx_viz.test.ts`

该文件位于 `src/commands/ctx_viz`，约 253 行，覆盖主题「ctx_viz」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.65 `src/commands/diagnostics/diagnostics.test.ts`

该文件位于 `src/commands/diagnostics`，约 318 行，覆盖主题「diagnostics」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.66 `src/commands/diagnostics/index.test.ts`

该文件位于 `src/commands/diagnostics`，约 40 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.67 `src/commands/doctor/doctor.test.tsx`

该文件位于 `src/commands/doctor`，约 185 行，覆盖主题「doctorx」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.68 `src/commands/effort/effort.test.tsx`

该文件位于 `src/commands/effort`，约 236 行，覆盖主题「effortx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.69 `src/commands/export/export.test.ts`

该文件位于 `src/commands/export`，约 294 行，覆盖主题「export」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.70 `src/commands/fast/fast.customPricing.test.ts`

该文件位于 `src/commands/fast`，约 36 行，覆盖主题「fast.customPricing」。不变量：自定义价格精确匹配；缺 webSearch 用默认；项目 settings 不得覆盖。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.71 `src/commands/goal/goal.test.ts`

该文件位于 `src/commands/goal`，约 213 行，覆盖主题「goal」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.72 `src/commands/init.test.ts`

该文件位于 `src/commands`，约 55 行，覆盖主题「init」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.73 `src/commands/install-github-app/repoSlug.test.ts`

该文件位于 `src/commands/install-github-app`，约 48 行，覆盖主题「repoSlug」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.74 `src/commands/install-github-app/setupGitHubActions.test.ts`

该文件位于 `src/commands/install-github-app`，约 279 行，覆盖主题「setupGitHubActions」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.75 `src/commands/knowledge/knowledge.test.ts`

该文件位于 `src/commands/knowledge`，约 153 行，覆盖主题「knowledge」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.76 `src/commands/lsp/lsp.test.ts`

该文件位于 `src/commands/lsp`，约 690 行，覆盖主题「lsp」。不变量：未连接即过滤；连接后可用。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.77 `src/commands/mcp/doctorCommand.test.ts`

该文件位于 `src/commands/mcp`，约 19 行，覆盖主题「doctorCommand」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.78 `src/commands/model/model.fastModeSwitch.test.ts`

该文件位于 `src/commands/model`，约 94 行，覆盖主题「model.fastModeSwitch」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.79 `src/commands/model/model.test.tsx`

该文件位于 `src/commands/model`，约 3830 行，覆盖主题「modelx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.80 `src/commands/onboard-github/onboard-github.test.ts`

该文件位于 `src/commands/onboard-github`，约 211 行，覆盖主题「onboard-github」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.81 `src/commands/provider/provider.test.tsx`

该文件位于 `src/commands/provider`，约 811 行，覆盖主题「providerx」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.82 `src/commands/replay/replay.test.tsx`

该文件位于 `src/commands/replay`，约 378 行，覆盖主题「replayx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.83 `src/commands/repomap/repomap.test.ts`

该文件位于 `src/commands/repomap`，约 299 行，覆盖主题「repomap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.84 `src/commands/request-size/request-size.test.ts`

该文件位于 `src/commands/request-size`，约 263 行，覆盖主题「request-size」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.85 `src/commands/resume/resume.test.tsx`

该文件位于 `src/commands/resume`，约 556 行，覆盖主题「resumex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.86 `src/commands/sandbox-toggle/index.test.ts`

该文件位于 `src/commands/sandbox-toggle`，约 64 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.87 `src/commands/smartroute/index.test.ts`

该文件位于 `src/commands/smartroute`，约 182 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.88 `src/commands/update/update.test.ts`

该文件位于 `src/commands/update`，约 68 行，覆盖主题「update」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.89 `src/commands/usage/index.test.ts`

该文件位于 `src/commands/usage`，约 175 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.90 `src/components/AutoUpdater.test.ts`

该文件位于 `src/components`，约 48 行，覆盖主题「AutoUpdater」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.91 `src/components/BuiltinStatusLine.test.tsx`

该文件位于 `src/components`，约 214 行，覆盖主题「BuiltinStatusLinex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.92 `src/components/ConsoleOAuthFlow.test.tsx`

该文件位于 `src/components`，约 145 行，覆盖主题「ConsoleOAuthFlowx」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.93 `src/components/CostThresholdDialog.test.ts`

该文件位于 `src/components`，约 20 行，覆盖主题「CostThresholdDialog」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.94 `src/components/EffortCallout.modelGate.test.ts`

该文件位于 `src/components`，约 22 行，覆盖主题「EffortCallout.modelGate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.95 `src/components/ExportDialog.test.tsx`

该文件位于 `src/components`，约 697 行，覆盖主题「ExportDialogx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.96 `src/components/Feedback.test.ts`

该文件位于 `src/components`，约 44 行，覆盖主题「Feedback」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.97 `src/components/LogSelector.resumeBranches.test.ts`

该文件位于 `src/components`，约 425 行，覆盖主题「LogSelector.resumeBranches」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.98 `src/components/LogoV2/WordmarkRow.test.tsx`

该文件位于 `src/components/LogoV2`，约 55 行，覆盖主题「WordmarkRowx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.99 `src/components/ModelPicker.switchProfile.test.ts`

该文件位于 `src/components`，约 276 行，覆盖主题「ModelPicker.switchProfile」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.100 `src/components/ModelPicker.test.tsx`

该文件位于 `src/components`，约 338 行，覆盖主题「ModelPickerx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.101 `src/components/NativeAutoUpdater.test.ts`

该文件位于 `src/components`，约 32 行，覆盖主题「NativeAutoUpdater」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.102 `src/components/PackageManagerUpdateGuidance.test.tsx`

该文件位于 `src/components`，约 75 行，覆盖主题「PackageManagerUpdateGuidancex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.103 `src/components/PromptInput/HistorySearchInput.test.tsx`

该文件位于 `src/components/PromptInput`，约 107 行，覆盖主题「HistorySearchInputx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.104 `src/components/PromptInput/KeepMounted.test.tsx`

该文件位于 `src/components/PromptInput`，约 98 行，覆盖主题「KeepMountedx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.105 `src/components/PromptInput/Notifications.effort.test.tsx`

该文件位于 `src/components/PromptInput`，约 312 行，覆盖主题「Notifications.effortx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.106 `src/components/PromptInput/PromptInputFooter.test.tsx`

该文件位于 `src/components/PromptInput`，约 130 行，覆盖主题「PromptInputFooterx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.107 `src/components/PromptInput/PromptInputFooterLeftSide.test.ts`

该文件位于 `src/components/PromptInput`，约 118 行，覆盖主题「PromptInputFooterLeftSide」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.108 `src/components/PromptInput/PromptInputFooterSuggestions.test.tsx`

该文件位于 `src/components/PromptInput`，约 35 行，覆盖主题「PromptInputFooterSuggestionsx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.109 `src/components/PromptInput/PromptInputQueuedCommands.test.tsx`

该文件位于 `src/components/PromptInput`，约 47 行，覆盖主题「PromptInputQueuedCommandsx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.110 `src/components/PromptInput/inputModes.test.ts`

该文件位于 `src/components/PromptInput`，约 104 行，覆盖主题「inputModes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.111 `src/components/PromptInput/utils.test.ts`

该文件位于 `src/components/PromptInput`，约 92 行，覆盖主题「utils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.112 `src/components/ProviderManager.test.tsx`

该文件位于 `src/components`，约 6304 行，覆盖主题「ProviderManagerx」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.113 `src/components/Spinner/SpinnerAnimationRow.test.tsx`

该文件位于 `src/components/Spinner`，约 1093 行，覆盖主题「SpinnerAnimationRowx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.114 `src/components/StartupScreen.palettes.test.ts`

该文件位于 `src/components`，约 31 行，覆盖主题「StartupScreen.palettes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.115 `src/components/StartupScreen.test.ts`

该文件位于 `src/components`，约 453 行，覆盖主题「StartupScreen」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.116 `src/components/StatusLine.active.test.tsx`

该文件位于 `src/components`，约 255 行，覆盖主题「StatusLine.activex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.117 `src/components/StatusLine.test.ts`

该文件位于 `src/components`，约 131 行，覆盖主题「StatusLine」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.118 `src/components/TextInput.test.tsx`

该文件位于 `src/components`，约 1405 行，覆盖主题「TextInputx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.119 `src/components/ThemePicker.test.tsx`

该文件位于 `src/components`，约 171 行，覆盖主题「ThemePickerx」。不变量：主题名解析到完整色板。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.120 `src/components/TrustDialog/utils.test.ts`

该文件位于 `src/components/TrustDialog`，约 292 行，覆盖主题「utils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.121 `src/components/agents/AgentDetail.test.tsx`

该文件位于 `src/components/agents`，约 139 行，覆盖主题「AgentDetailx」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.122 `src/components/agents/AgentDetailDialog.test.tsx`

该文件位于 `src/components/agents`，约 141 行，覆盖主题「AgentDetailDialogx」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.123 `src/components/agents/AgentRouteSelector.test.tsx`

该文件位于 `src/components/agents`，约 298 行，覆盖主题「AgentRouteSelectorx」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.124 `src/components/agents/AgentsList.test.tsx`

该文件位于 `src/components/agents`，约 192 行，覆盖主题「AgentsListx」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.125 `src/components/agents/AgentsMenu.test.tsx`

该文件位于 `src/components/agents`，约 496 行，覆盖主题「AgentsMenux」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.126 `src/components/agents/agentFileUtils.test.ts`

该文件位于 `src/components/agents`，约 46 行，覆盖主题「agentFileUtils」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.127 `src/components/agents/new-agent-creation/wizard-steps/wizardSteps.test.tsx`

该文件位于 `src/components/agents/new-agent-creation/wizard-steps`，约 254 行，覆盖主题「wizardStepsx」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.128 `src/components/design-system/ThemeProvider.test.tsx`

该文件位于 `src/components/design-system`，约 229 行，覆盖主题「ThemeProviderx」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.129 `src/components/memory/memoryFileSelectorPaths.test.ts`

该文件位于 `src/components/memory`，约 72 行，覆盖主题「memoryFileSelectorPaths」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.130 `src/components/messages/SnipBoundaryMessage.test.tsx`

该文件位于 `src/components/messages`，约 38 行，覆盖主题「SnipBoundaryMessagex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.131 `src/components/messages/SystemAPIErrorMessage.test.tsx`

该文件位于 `src/components/messages`，约 107 行，覆盖主题「SystemAPIErrorMessagex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.132 `src/components/messages/UserForkBoilerplateMessage.test.tsx`

该文件位于 `src/components/messages`，约 48 行，覆盖主题「UserForkBoilerplateMessagex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.133 `src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.render.test.tsx`

该文件位于 `src/components/permissions/ExitPlanModePermissionRequest`，约 229 行，覆盖主题「ExitPlanModePermissionRequest.renderx」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.134 `src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.test.ts`

该文件位于 `src/components/permissions/ExitPlanModePermissionRequest`，约 143 行，覆盖主题「ExitPlanModePermissionRequest」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.135 `src/components/permissions/MonitorPermissionRequest/MonitorPermissionRequest.test.tsx`

该文件位于 `src/components/permissions/MonitorPermissionRequest`，约 537 行，覆盖主题「MonitorPermissionRequestx」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.136 `src/components/permissions/rules/permissionModeOptions.test.ts`

该文件位于 `src/components/permissions/rules`，约 37 行，覆盖主题「permissionModeOptions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.137 `src/components/permissions/useDangerousModeConfirmation.test.tsx`

该文件位于 `src/components/permissions`，约 217 行，覆盖主题「useDangerousModeConfirmationx」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.138 `src/components/tasks/taskStatusUtils.test.tsx`

该文件位于 `src/components/tasks`，约 61 行，覆盖主题「taskStatusUtilsx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.139 `src/components/useCodexOAuthFlow.test.tsx`

该文件位于 `src/components`，约 486 行，覆盖主题「useCodexOAuthFlowx」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.140 `src/components/useXaiOAuthFlow.test.tsx`

该文件位于 `src/components`，约 166 行，覆盖主题「useXaiOAuthFlowx」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.141 `src/constants/brand.test.ts`

该文件位于 `src/constants`，约 44 行，覆盖主题「brand」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.142 `src/constants/outputStyles.protoName.test.ts`

该文件位于 `src/constants`，约 48 行，覆盖主题「outputStyles.protoName」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.143 `src/constants/product.test.ts`

该文件位于 `src/constants`，约 118 行，覆盖主题「product」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.144 `src/constants/promptIdentity.test.ts`

该文件位于 `src/constants`，约 209 行，覆盖主题「promptIdentity」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.145 `src/constants/prompts.doingTasks.test.ts`

该文件位于 `src/constants`，约 35 行，覆盖主题「prompts.doingTasks」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.146 `src/context.repoMap.test.ts`

该文件位于 `src`，约 236 行，覆盖主题「context.repoMap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.147 `src/context/repoMap/gitFiles.test.ts`

该文件位于 `src/context/repoMap`，约 42 行，覆盖主题「gitFiles」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.148 `src/context/repoMap/queries.test.ts`

该文件位于 `src/context/repoMap`，约 34 行，覆盖主题「queries」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.149 `src/context/repoMap/repoMap.test.ts`

该文件位于 `src/context/repoMap`，约 866 行，覆盖主题「repoMap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.150 `src/cost-tracker.cacheIntegration.test.ts`

该文件位于 `src`，约 141 行，覆盖主题「cost-tracker.cacheIntegration」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.151 `src/cost-tracker.customPricing.test.ts`

该文件位于 `src`，约 143 行，覆盖主题「cost-tracker.customPricing」。不变量：自定义价格精确匹配；缺 webSearch 用默认；项目 settings 不得覆盖。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.152 `src/cost-tracker.format.test.ts`

该文件位于 `src`，约 209 行，覆盖主题「cost-tracker.format」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.153 `src/entrypoints/cli.skills.test.ts`

该文件位于 `src/entrypoints`，约 306 行，覆盖主题「cli.skills」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.154 `src/entrypoints/cli.test.ts`

该文件位于 `src/entrypoints`，约 1259 行，覆盖主题「cli」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.155 `src/entrypoints/mcp.test.ts`

该文件位于 `src/entrypoints`，约 103 行，覆盖主题「mcp」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.156 `src/entrypoints/sdk/agentDefinitions.test.ts`

该文件位于 `src/entrypoints/sdk`，约 196 行，覆盖主题「agentDefinitions」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.157 `src/grpc/server.interruptionTrace.test.ts`

该文件位于 `src/grpc`，约 89 行，覆盖主题「server.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.158 `src/hooks/fileSuggestions.test.ts`

该文件位于 `src/hooks`，约 430 行，覆盖主题「fileSuggestions」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.159 `src/hooks/notifs/useNpmDeprecationNotification.test.ts`

该文件位于 `src/hooks/notifs`，约 60 行，覆盖主题「useNpmDeprecationNotification」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.160 `src/hooks/toolPermission/handlers/interactiveHandler.test.ts`

该文件位于 `src/hooks/toolPermission/handlers`，约 320 行，覆盖主题「interactiveHandler」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.161 `src/hooks/useApiKeyVerification.test.tsx`

该文件位于 `src/hooks`，约 141 行，覆盖主题「useApiKeyVerificationx」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.162 `src/hooks/useBackgroundTaskNavigation.interruptionTrace.test.tsx`

该文件位于 `src/hooks`，约 223 行，覆盖主题「useBackgroundTaskNavigation.interruptionTracex」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.163 `src/hooks/useCancelRequest.test.tsx`

该文件位于 `src/hooks`，约 304 行，覆盖主题「useCancelRequestx」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.164 `src/hooks/usePasteHandler.image-path-error.test.ts`

该文件位于 `src/hooks`，约 150 行，覆盖主题「usePasteHandler.image-path-error」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.165 `src/hooks/usePasteHandler.test.ts`

该文件位于 `src/hooks`，约 64 行，覆盖主题「usePasteHandler」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.166 `src/hooks/usePromptsFromClaudeInChrome.test.ts`

该文件位于 `src/hooks`，约 19 行，覆盖主题「usePromptsFromClaudeInChrome」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.167 `src/hooks/useTextInput.test.ts`

该文件位于 `src/hooks`，约 439 行，覆盖主题「useTextInput」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.168 `src/ink/components/App.test.tsx`

该文件位于 `src/ink/components`，约 221 行，覆盖主题「Appx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.169 `src/ink/hooks/use-input.test.ts`

该文件位于 `src/ink/hooks`，约 166 行，覆盖主题「use-input」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.170 `src/ink/log-update.test.ts`

该文件位于 `src/ink`，约 126 行，覆盖主题「log-update」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.171 `src/ink/output.test.ts`

该文件位于 `src/ink`，约 258 行，覆盖主题「output」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.172 `src/ink/parse-keypress.test.ts`

该文件位于 `src/ink`，约 151 行，覆盖主题「parse-keypress」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.173 `src/ink/reconciler.test.ts`

该文件位于 `src/ink`，约 369 行，覆盖主题「reconciler」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.174 `src/ink/renderer.test.ts`

该文件位于 `src/ink`，约 151 行，覆盖主题「renderer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.175 `src/ink/stringWidth.test.ts`

该文件位于 `src/ink`，约 56 行，覆盖主题「stringWidth」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.176 `src/ink/termio/osc.test.ts`

该文件位于 `src/ink/termio`，约 172 行，覆盖主题「osc」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.177 `src/integrations/aimlapi/client.test.ts`

该文件位于 `src/integrations/aimlapi`，约 649 行，覆盖主题「client」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.178 `src/integrations/aimlapi/config.test.ts`

该文件位于 `src/integrations/aimlapi`，约 266 行，覆盖主题「config」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.179 `src/integrations/aimlapi/onboarding.test.ts`

该文件位于 `src/integrations/aimlapi`，约 632 行，覆盖主题「onboarding」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.180 `src/integrations/aimlapi/topup.test.ts`

该文件位于 `src/integrations/aimlapi`，约 2026 行，覆盖主题「topup」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.181 `src/integrations/aimlapi/topupState.test.ts`

该文件位于 `src/integrations/aimlapi`，约 1463 行，覆盖主题「topupState」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.182 `src/integrations/aimlapi/transport.test.ts`

该文件位于 `src/integrations/aimlapi`，约 30 行，覆盖主题「transport」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.183 `src/integrations/artifactGenerator.test.ts`

该文件位于 `src/integrations`，约 277 行，覆盖主题「artifactGenerator」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.184 `src/integrations/compatibility.test.ts`

该文件位于 `src/integrations`，约 144 行，覆盖主题「compatibility」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.185 `src/integrations/discoveryCache.test.ts`

该文件位于 `src/integrations`，约 244 行，覆盖主题「discoveryCache」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.186 `src/integrations/discoveryService.test.ts`

该文件位于 `src/integrations`，约 1394 行，覆盖主题「discoveryService」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.187 `src/integrations/gateways/apismart.test.ts`

该文件位于 `src/integrations/gateways`，约 44 行，覆盖主题「apismart」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.188 `src/integrations/gateways/commandcode.test.ts`

该文件位于 `src/integrations/gateways`，约 263 行，覆盖主题「commandcode」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.189 `src/integrations/gateways/concentrate.test.ts`

该文件位于 `src/integrations/gateways`，约 79 行，覆盖主题「concentrate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.190 `src/integrations/gateways/custom.test.ts`

该文件位于 `src/integrations/gateways`，约 186 行，覆盖主题「custom」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.191 `src/integrations/gateways/gitlawb-opengateway.test.ts`

该文件位于 `src/integrations/gateways`，约 76 行，覆盖主题「gitlawb-opengateway」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.192 `src/integrations/gateways/llmtr.test.ts`

该文件位于 `src/integrations/gateways`，约 187 行，覆盖主题「llmtr」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.193 `src/integrations/gateways/nvidia-nim.test.ts`

该文件位于 `src/integrations/gateways`，约 149 行，覆盖主题「nvidia-nim」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.194 `src/integrations/gateways/opencode.test.ts`

该文件位于 `src/integrations/gateways`，约 532 行，覆盖主题「opencode」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.195 `src/integrations/gateways/openrouter.test.ts`

该文件位于 `src/integrations/gateways`，约 127 行，覆盖主题「openrouter」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.196 `src/integrations/gateways/xiaomi-mimo-token.test.ts`

该文件位于 `src/integrations/gateways`，约 56 行，覆盖主题「xiaomi-mimo-token」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.197 `src/integrations/gpt56Catalog.test.ts`

该文件位于 `src/integrations`，约 32 行，覆盖主题「gpt56Catalog」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.198 `src/integrations/index.test.ts`

该文件位于 `src/integrations`，约 123 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.199 `src/integrations/ling-tiny.test.ts`

该文件位于 `src/integrations`，约 77 行，覆盖主题「ling-tiny」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.200 `src/integrations/macaron.test.ts`

该文件位于 `src/integrations`，约 47 行，覆盖主题「macaron」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.201 `src/integrations/models/kimi.test.ts`

该文件位于 `src/integrations/models`，约 8 行，覆盖主题「kimi」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.202 `src/integrations/nearai.test.ts`

该文件位于 `src/integrations`，约 62 行，覆盖主题「nearai」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.203 `src/integrations/registry.test.ts`

该文件位于 `src/integrations`，约 533 行，覆盖主题「registry」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.204 `src/integrations/routeMetadata.test.ts`

该文件位于 `src/integrations`，约 1263 行，覆盖主题「routeMetadata」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.205 `src/integrations/runtimeMetadata.modelLimits.test.ts`

该文件位于 `src/integrations`，约 237 行，覆盖主题「runtimeMetadata.modelLimits」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.206 `src/integrations/runtimeMetadata.test.ts`

该文件位于 `src/integrations`，约 1525 行，覆盖主题「runtimeMetadata」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.207 `src/integrations/tencent.test.ts`

该文件位于 `src/integrations`，约 41 行，覆盖主题「tencent」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.208 `src/integrations/vendors/longcat.test.ts`

该文件位于 `src/integrations/vendors`，约 88 行，覆盖主题「longcat」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.209 `src/integrations/vendors/xai.test.ts`

该文件位于 `src/integrations/vendors`，约 110 行，覆盖主题「xai」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.210 `src/memdir/autoExtractFacts.test.ts`

该文件位于 `src/memdir`，约 343 行，覆盖主题「autoExtractFacts」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.211 `src/memdir/memdir.entrypointBytes.test.ts`

该文件位于 `src/memdir`，约 80 行，覆盖主题「memdir.entrypointBytes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.212 `src/memdir/memoryScan.test.ts`

该文件位于 `src/memdir`，约 507 行，覆盖主题「memoryScan」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.213 `src/memdir/paths.test.ts`

该文件位于 `src/memdir`，约 151 行，覆盖主题「paths」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.214 `src/memdir/vectorIndex.test.ts`

该文件位于 `src/memdir`，约 416 行，覆盖主题「vectorIndex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.215 `src/native-ts/color-diff/detectLanguage.test.ts`

该文件位于 `src/native-ts/color-diff`，约 45 行，覆盖主题「detectLanguage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.216 `src/optionalModules.types.test.ts`

该文件位于 `src`，约 30 行，覆盖主题「optionalModules.types」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.217 `src/plugins/bundled/karpathyGuidelines.test.ts`

该文件位于 `src/plugins/bundled`，约 41 行，覆盖主题「karpathyGuidelines」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.218 `src/proactive/index.test.ts`

该文件位于 `src/proactive`，约 58 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.219 `src/projectOnboardingState.test.ts`

该文件位于 `src`，约 62 行，覆盖主题「projectOnboardingState」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.220 `src/query.abortClassification.test.ts`

该文件位于 `src`，约 285 行，覆盖主题「query.abortClassification」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.221 `src/query.conversationArc.test.ts`

该文件位于 `src`，约 169 行，覆盖主题「query.conversationArc」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.222 `src/query/agentStepLimit.test.ts`

该文件位于 `src/query`，约 1124 行，覆盖主题「agentStepLimit」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.223 `src/query/autoCompactCooldown.test.ts`

该文件位于 `src/query`，约 985 行，覆盖主题「autoCompactCooldown」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.224 `src/query/goalContinuation.test.ts`

该文件位于 `src/query`，约 245 行，覆盖主题「goalContinuation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.225 `src/query/providerMaxTokensCapRetry.test.ts`

该文件位于 `src/query`，约 445 行，覆盖主题「providerMaxTokensCapRetry」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.226 `src/query/requestOnlyMessages.test.ts`

该文件位于 `src/query`，约 468 行，覆盖主题「requestOnlyMessages」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.227 `src/query/stopHooks.goal.test.ts`

该文件位于 `src/query`，约 382 行，覆盖主题「stopHooks.goal」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.228 `src/query/stopHooks.test.ts`

该文件位于 `src/query`，约 24 行，覆盖主题「stopHooks」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.229 `src/query/toolFailureLoopGuard.test.ts`

该文件位于 `src/query`，约 1386 行，覆盖主题「toolFailureLoopGuard」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.230 `src/queryEngine.goal.test.ts`

该文件位于 `src`，约 83 行，覆盖主题「queryEngine.goal」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.231 `src/screens/REPL.queryLifecycle.test.ts`

该文件位于 `src/screens`，约 182 行，覆盖主题「REPL.queryLifecycle」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.232 `src/screens/ResumeConversation.test.ts`

该文件位于 `src/screens`，约 65 行，覆盖主题「ResumeConversation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.233 `src/screens/doctorDiagnosticLoad.test.ts`

该文件位于 `src/screens`，约 59 行，覆盖主题「doctorDiagnosticLoad」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.234 `src/screens/doctorDistTags.test.ts`

该文件位于 `src/screens`，约 88 行，覆盖主题「doctorDistTags」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.235 `src/screens/replActiveAgentModel.test.ts`

该文件位于 `src/screens`，约 64 行，覆盖主题「replActiveAgentModel」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.236 `src/screens/replFallbackModelProp.test.ts`

该文件位于 `src/screens`，约 231 行，覆盖主题「replFallbackModelProp」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.237 `src/screens/replFocusedInputDialog.test.ts`

该文件位于 `src/screens`，约 63 行，覆盖主题「replFocusedInputDialog」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.238 `src/screens/replInputSuppression.test.ts`

该文件位于 `src/screens`，约 18 行，覆盖主题「replInputSuppression」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.239 `src/screens/replMaxTurnsProp.test.ts`

该文件位于 `src/screens`，约 481 行，覆盖主题「replMaxTurnsProp」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.240 `src/screens/replStartupGates.test.ts`

该文件位于 `src/screens`，约 53 行，覆盖主题「replStartupGates」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.241 `src/screens/replStreamingTextClear.test.ts`

该文件位于 `src/screens`，约 33 行，覆盖主题「replStreamingTextClear」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.242 `src/screens/streamingTextPublish.test.ts`

该文件位于 `src/screens`，约 102 行，覆盖主题「streamingTextPublish」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.243 `src/services/AgentSummary/agentSummary.test.ts`

该文件位于 `src/services/AgentSummary`，约 98 行，覆盖主题「agentSummary」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.244 `src/services/PromptSuggestion/speculation.test.ts`

该文件位于 `src/services/PromptSuggestion`，约 144 行，覆盖主题「speculation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.245 `src/services/ads.test.ts`

该文件位于 `src/services`，约 127 行，覆盖主题「ads」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.246 `src/services/api/agentRouteSettings.test.ts`

该文件位于 `src/services/api`，约 477 行，覆盖主题「agentRouteSettings」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.247 `src/services/api/agentRouting.test.ts`

该文件位于 `src/services/api`，约 875 行，覆盖主题「agentRouting」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.248 `src/services/api/authRouting.attribution.test.ts`

该文件位于 `src/services/api`，约 218 行，覆盖主题「authRouting.attribution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.249 `src/services/api/bootstrap.test.ts`

该文件位于 `src/services/api`，约 301 行，覆盖主题「bootstrap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.250 `src/services/api/cacheMetrics.test.ts`

该文件位于 `src/services/api`，约 810 行，覆盖主题「cacheMetrics」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.251 `src/services/api/cacheMetricsIntegration.test.ts`

该文件位于 `src/services/api`，约 339 行，覆盖主题「cacheMetricsIntegration」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.252 `src/services/api/cacheStatsTracker.test.ts`

该文件位于 `src/services/api`，约 224 行，覆盖主题「cacheStatsTracker」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.253 `src/services/api/claude.abortClassification.test.ts`

该文件位于 `src/services/api`，约 131 行，覆盖主题「claude.abortClassification」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.254 `src/services/api/claude.lifecycle.test.ts`

该文件位于 `src/services/api`，约 1392 行，覆盖主题「claude.lifecycle」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.255 `src/services/api/claude.streamWatchdog.test.ts`

该文件位于 `src/services/api`，约 663 行，覆盖主题「claude.streamWatchdog」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.256 `src/services/api/claude.toolHistoryRouting.test.ts`

该文件位于 `src/services/api`，约 87 行，覆盖主题「claude.toolHistoryRouting」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.257 `src/services/api/client.oauthRouting.test.ts`

该文件位于 `src/services/api`，约 79 行，覆盖主题「client.oauthRouting」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.258 `src/services/api/client.optionalRuntime.test.ts`

该文件位于 `src/services/api`，约 148 行，覆盖主题「client.optionalRuntime」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.259 `src/services/api/client.test.ts`

该文件位于 `src/services/api`，约 3092 行，覆盖主题「client」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.260 `src/services/api/clinepassUsage.test.ts`

该文件位于 `src/services/api`，约 190 行，覆盖主题「clinepassUsage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.261 `src/services/api/codexOAuth.test.ts`

该文件位于 `src/services/api`，约 434 行，覆盖主题「codexOAuth」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.262 `src/services/api/codexOAuthShared.test.ts`

该文件位于 `src/services/api`，约 38 行，覆盖主题「codexOAuthShared」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.263 `src/services/api/codexShim.interruption.test.ts`

该文件位于 `src/services/api`，约 841 行，覆盖主题「codexShim.interruption」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.264 `src/services/api/codexShim.test.ts`

该文件位于 `src/services/api`，约 1484 行，覆盖主题「codexShim」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.265 `src/services/api/codexUsage.test.ts`

该文件位于 `src/services/api`，约 204 行，覆盖主题「codexUsage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.266 `src/services/api/compressToolHistory.test.ts`

该文件位于 `src/services/api`，约 925 行，覆盖主题「compressToolHistory」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.267 `src/services/api/credentialPool.test.ts`

该文件位于 `src/services/api`，约 94 行，覆盖主题「credentialPool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.268 `src/services/api/errors.openaiCompatibility.test.ts`

该文件位于 `src/services/api`，约 148 行，覆盖主题「errors.openaiCompatibility」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.269 `src/services/api/errors.opencodeGo.test.ts`

该文件位于 `src/services/api`，约 324 行，覆盖主题「errors.opencodeGo」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.270 `src/services/api/fetchWithProxyRetry.test.ts`

该文件位于 `src/services/api`，约 262 行，覆盖主题「fetchWithProxyRetry」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.271 `src/services/api/geminiVertexClient.test.ts`

该文件位于 `src/services/api`，约 551 行，覆盖主题「geminiVertexClient」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.272 `src/services/api/grove.test.ts`

该文件位于 `src/services/api`，约 106 行，覆盖主题「grove」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.273 `src/services/api/minimaxUsage.test.ts`

该文件位于 `src/services/api`，约 334 行，覆盖主题「minimaxUsage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.274 `src/services/api/openaiErrorClassification.test.ts`

该文件位于 `src/services/api`，约 535 行，覆盖主题「openaiErrorClassification」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.275 `src/services/api/openaiShim.architecture.test.ts`

该文件位于 `src/services/api`，约 52 行，覆盖主题「openaiShim.architecture」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.276 `src/services/api/openaiShim.compression.test.ts`

该文件位于 `src/services/api`，约 1468 行，覆盖主题「openaiShim.compression」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.277 `src/services/api/openaiShim.diagnostics.test.ts`

该文件位于 `src/services/api`，约 329 行，覆盖主题「openaiShim.diagnostics」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.278 `src/services/api/openaiShim.ollamaTextToolCalls.test.ts`

该文件位于 `src/services/api`，约 657 行，覆盖主题「openaiShim.ollamaTextToolCalls」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.279 `src/services/api/openaiShim.test.ts`

该文件位于 `src/services/api`，约 6419 行，覆盖主题「openaiShim」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.280 `src/services/api/openaiShim/clientDispatch.test.ts`

该文件位于 `src/services/api/openaiShim`，约 389 行，覆盖主题「clientDispatch」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.281 `src/services/api/openaiShim/codexDispatch.test.ts`

该文件位于 `src/services/api/openaiShim`，约 289 行，覆盖主题「codexDispatch」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.282 `src/services/api/openaiShim/geminiStreamConversion.test.ts`

该文件位于 `src/services/api/openaiShim`，约 146 行，覆盖主题「geminiStreamConversion」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.283 `src/services/api/openaiShim/markerEchoGuard.test.ts`

该文件位于 `src/services/api/openaiShim`，约 119 行，覆盖主题「markerEchoGuard」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.284 `src/services/api/openaiShim/messageConversion.test.ts`

该文件位于 `src/services/api/openaiShim`，约 350 行，覆盖主题「messageConversion」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.285 `src/services/api/openaiShim/ollamaAdapter.test.ts`

该文件位于 `src/services/api/openaiShim`，约 321 行，覆盖主题「ollamaAdapter」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.286 `src/services/api/openaiShim/providerCompatibility.test.ts`

该文件位于 `src/services/api/openaiShim`，约 610 行，覆盖主题「providerCompatibility」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.287 `src/services/api/openaiShim/providerStreamInterruptionTrace.test.ts`

该文件位于 `src/services/api/openaiShim`，约 366 行，覆盖主题「providerStreamInterruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.288 `src/services/api/openaiShim/rawToolCallParsing.test.ts`

该文件位于 `src/services/api/openaiShim`，约 133 行，覆盖主题「rawToolCallParsing」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.289 `src/services/api/openaiShim/requestExecutor.integration.test.ts`

该文件位于 `src/services/api/openaiShim`，约 4798 行，覆盖主题「requestExecutor.integration」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.290 `src/services/api/openaiShim/requestExecutor.test.ts`

该文件位于 `src/services/api/openaiShim`，约 2322 行，覆盖主题「requestExecutor」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.291 `src/services/api/openaiShim/requestPlanner.test.ts`

该文件位于 `src/services/api/openaiShim`，约 425 行，覆盖主题「requestPlanner」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.292 `src/services/api/openaiShim/requestPreparation.test.ts`

该文件位于 `src/services/api/openaiShim`，约 128 行，覆盖主题「requestPreparation」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.293 `src/services/api/openaiShim/responseAdapters.test.ts`

该文件位于 `src/services/api/openaiShim`，约 195 行，覆盖主题「responseAdapters」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.294 `src/services/api/openaiShim/responseConversion.test.ts`

该文件位于 `src/services/api/openaiShim`，约 228 行，覆盖主题「responseConversion」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.295 `src/services/api/openaiShim/streamControl.test.ts`

该文件位于 `src/services/api/openaiShim`，约 373 行，覆盖主题「streamControl」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.296 `src/services/api/openaiShim/streamConversion.test.ts`

该文件位于 `src/services/api/openaiShim`，约 612 行，覆盖主题「streamConversion」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.297 `src/services/api/openaiShim/toolConversion.test.ts`

该文件位于 `src/services/api/openaiShim`，约 116 行，覆盖主题「toolConversion」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.298 `src/services/api/openaiShim/transport.test.ts`

该文件位于 `src/services/api/openaiShim`，约 155 行，覆盖主题「transport」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.299 `src/services/api/openaiShim/xmlToolCallParsing.test.ts`

该文件位于 `src/services/api/openaiShim`，约 530 行，覆盖主题「xmlToolCallParsing」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.300 `src/services/api/promptCacheBreakDetection.test.ts`

该文件位于 `src/services/api`，约 575 行，覆盖主题「promptCacheBreakDetection」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.301 `src/services/api/providerConfig.codexSecureStorage.test.ts`

该文件位于 `src/services/api`，约 243 行，覆盖主题「providerConfig.codexSecureStorage」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.302 `src/services/api/providerConfig.envDiagnostics.test.ts`

该文件位于 `src/services/api`，约 161 行，覆盖主题「providerConfig.envDiagnostics」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.303 `src/services/api/providerConfig.github.test.ts`

该文件位于 `src/services/api`，约 139 行，覆盖主题「providerConfig.github」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.304 `src/services/api/providerConfig.local.test.ts`

该文件位于 `src/services/api`，约 560 行，覆盖主题「providerConfig.local」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.305 `src/services/api/providerConfig.localFastPath.test.ts`

该文件位于 `src/services/api`，约 109 行，覆盖主题「providerConfig.localFastPath」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.306 `src/services/api/providerConfig.protoAlias.test.ts`

该文件位于 `src/services/api`，约 134 行，覆盖主题「providerConfig.protoAlias」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.307 `src/services/api/providerConfig.runtimeCodexCredentials.test.ts`

该文件位于 `src/services/api`，约 128 行，覆盖主题「providerConfig.runtimeCodexCredentials」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.308 `src/services/api/providerConfig.test.ts`

该文件位于 `src/services/api`，约 528 行，覆盖主题「providerConfig」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.309 `src/services/api/providerMaxTokensCap.test.ts`

该文件位于 `src/services/api`，约 143 行，覆盖主题「providerMaxTokensCap」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.310 `src/services/api/smartModelRouting.test.ts`

该文件位于 `src/services/api`，约 191 行，覆盖主题「smartModelRouting」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.311 `src/services/api/smartRouting/index.test.ts`

该文件位于 `src/services/api/smartRouting`，约 395 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.312 `src/services/api/smartRouting/resolveConfig.test.ts`

该文件位于 `src/services/api/smartRouting`，约 93 行，覆盖主题「resolveConfig」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.313 `src/services/api/smartRouting/settings.test.ts`

该文件位于 `src/services/api/smartRouting`，约 111 行，覆盖主题「settings」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.314 `src/services/api/thinkTagSanitizer.test.ts`

该文件位于 `src/services/api`，约 183 行，覆盖主题「thinkTagSanitizer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.315 `src/services/api/toolArgumentNormalization.test.ts`

该文件位于 `src/services/api`，约 221 行，覆盖主题「toolArgumentNormalization」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.316 `src/services/api/vertexClient.test.ts`

该文件位于 `src/services/api`，约 248 行，覆盖主题「vertexClient」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.317 `src/services/api/withRetry.test.ts`

该文件位于 `src/services/api`，约 872 行，覆盖主题「withRetry」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.318 `src/services/api/xaiOAuthCallback.test.ts`

该文件位于 `src/services/api`，约 424 行，覆盖主题「xaiOAuthCallback」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.319 `src/services/api/xaiOAuthShared.test.ts`

该文件位于 `src/services/api`，约 145 行，覆盖主题「xaiOAuthShared」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.320 `src/services/autoFix/autoFixConfig.test.ts`

该文件位于 `src/services/autoFix`，约 106 行，覆盖主题「autoFixConfig」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.321 `src/services/autoFix/autoFixHook.test.ts`

该文件位于 `src/services/autoFix`，约 63 行，覆盖主题「autoFixHook」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.322 `src/services/autoFix/autoFixIntegration.test.ts`

该文件位于 `src/services/autoFix`，约 50 行，覆盖主题「autoFixIntegration」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.323 `src/services/autoFix/autoFixRunner.test.ts`

该文件位于 `src/services/autoFix`，约 105 行，覆盖主题「autoFixRunner」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.324 `src/services/awaySummary.test.ts`

该文件位于 `src/services`，约 182 行，覆盖主题「awaySummary」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.325 `src/services/compact/autoCompact.test.ts`

该文件位于 `src/services/compact`，约 980 行，覆盖主题「autoCompact」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.326 `src/services/compact/cachedMicrocompact.test.ts`

该文件位于 `src/services/compact`，约 65 行，覆盖主题「cachedMicrocompact」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.327 `src/services/compact/compact.test.ts`

该文件位于 `src/services/compact`，约 1058 行，覆盖主题「compact」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.328 `src/services/compact/microCompact.test.ts`

该文件位于 `src/services/compact`，约 239 行，覆盖主题「microCompact」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.329 `src/services/compact/resumeCompactPrompt.test.ts`

该文件位于 `src/services/compact`，约 73 行，覆盖主题「resumeCompactPrompt」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.330 `src/services/compact/snipCompact.test.ts`

该文件位于 `src/services/compact`，约 391 行，覆盖主题「snipCompact」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.331 `src/services/compact/snipProjection.test.ts`

该文件位于 `src/services/compact`，约 82 行，覆盖主题「snipProjection」。不变量：压缩后必须存在 boundary；冷却在手动 compact 后清除；阈值不得为负。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.332 `src/services/contextCollapse/collapseUtils.test.ts`

该文件位于 `src/services/contextCollapse`，约 163 行，覆盖主题「collapseUtils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.333 `src/services/contextCollapse/index.test.ts`

该文件位于 `src/services/contextCollapse`，约 641 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.334 `src/services/contextCollapse/operations.test.ts`

该文件位于 `src/services/contextCollapse`，约 281 行，覆盖主题「operations」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.335 `src/services/contextCollapse/persist.test.ts`

该文件位于 `src/services/contextCollapse`，约 72 行，覆盖主题「persist」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.336 `src/services/contextCollapse/spanSelection.test.ts`

该文件位于 `src/services/contextCollapse`，约 107 行，覆盖主题「spanSelection」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.337 `src/services/contextCollapse/spawnCtxAgent.test.ts`

该文件位于 `src/services/contextCollapse`，约 201 行，覆盖主题「spawnCtxAgent」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.338 `src/services/diagnosticTracking.test.ts`

该文件位于 `src/services`，约 152 行，覆盖主题「diagnosticTracking」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.339 `src/services/extractMemories/extractMemories.abort.test.ts`

该文件位于 `src/services/extractMemories`，约 383 行，覆盖主题「extractMemories.abort」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.340 `src/services/github/deviceFlow.test.ts`

该文件位于 `src/services/github`，约 290 行，覆盖主题「deviceFlow」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.341 `src/services/goal/controller.test.ts`

该文件位于 `src/services/goal`，约 598 行，覆盖主题「controller」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.342 `src/services/goal/evaluator.test.ts`

该文件位于 `src/services/goal`，约 250 行，覆盖主题「evaluator」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.343 `src/services/goal/state.test.ts`

该文件位于 `src/services/goal`，约 136 行，覆盖主题「state」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.344 `src/services/lsp/LSPDiagnosticRegistry.test.ts`

该文件位于 `src/services/lsp`，约 668 行，覆盖主题「LSPDiagnosticRegistry」。不变量：未连接即过滤；连接后可用。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.345 `src/services/mcp/auth.refreshLock.test.ts`

该文件位于 `src/services/mcp`，约 2131 行，覆盖主题「auth.refreshLock」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.346 `src/services/mcp/auth.test.ts`

该文件位于 `src/services/mcp`，约 61 行，覆盖主题「auth」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.347 `src/services/mcp/channelNotification.test.ts`

该文件位于 `src/services/mcp`，约 612 行，覆盖主题「channelNotification」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.348 `src/services/mcp/channelPermissions.test.ts`

该文件位于 `src/services/mcp`，约 29 行，覆盖主题「channelPermissions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.349 `src/services/mcp/client.activity.test.ts`

该文件位于 `src/services/mcp`，约 703 行，覆盖主题「client.activity」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.350 `src/services/mcp/client.pagination.test.ts`

该文件位于 `src/services/mcp`，约 950 行，覆盖主题「client.pagination」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.351 `src/services/mcp/client.test.ts`

该文件位于 `src/services/mcp`，约 397 行，覆盖主题「client」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.352 `src/services/mcp/doctor.test.ts`

该文件位于 `src/services/mcp`，约 546 行，覆盖主题「doctor」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.353 `src/services/mcp/envExpansion.test.ts`

该文件位于 `src/services/mcp`，约 49 行，覆盖主题「envExpansion」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.354 `src/services/mcp/officialRegistry.test.ts`

该文件位于 `src/services/mcp`，约 96 行，覆盖主题「officialRegistry」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.355 `src/services/mcp/xaa.test.ts`

该文件位于 `src/services/mcp`，约 123 行，覆盖主题「xaa」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.356 `src/services/mcp/xaaIdpLogin.test.ts`

该文件位于 `src/services/mcp`，约 98 行，覆盖主题「xaaIdpLogin」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.357 `src/services/oauth/auth-code-listener.analytics.test.ts`

该文件位于 `src/services/oauth`，约 167 行，覆盖主题「auth-code-listener.analytics」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.358 `src/services/oauth/auth-code-listener.test.ts`

该文件位于 `src/services/oauth`，约 29 行，覆盖主题「auth-code-listener」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.359 `src/services/oauth/client.populateAccountInfo.test.ts`

该文件位于 `src/services/oauth`，约 42 行，覆盖主题「client.populateAccountInfo」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.360 `src/services/oauth/crypto.test.ts`

该文件位于 `src/services/oauth`，约 27 行，覆盖主题「crypto」。不变量：state 不匹配不得 settle；刷新失败不得静默用过期 token。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.361 `src/services/plugins/pluginOperations.installLocation.test.ts`

该文件位于 `src/services/plugins`，约 75 行，覆盖主题「pluginOperations.installLocation」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.362 `src/services/settingsSync/settings.transaction.test.ts`

该文件位于 `src/services/settingsSync`，约 371 行，覆盖主题「settings.transaction」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.363 `src/services/teamMemorySync/index.test.ts`

该文件位于 `src/services/teamMemorySync`，约 174 行，覆盖主题「index」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.364 `src/services/teamMemorySync/watcher.test.ts`

该文件位于 `src/services/teamMemorySync`，约 270 行，覆盖主题「watcher」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.365 `src/services/tips/gitlawbEarn.test.ts`

该文件位于 `src/services/tips`，约 111 行，覆盖主题「gitlawbEarn」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.366 `src/services/tips/sponsoredTips.test.ts`

该文件位于 `src/services/tips`，约 218 行，覆盖主题「sponsoredTips」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.367 `src/services/tips/tipLink.test.ts`

该文件位于 `src/services/tips`，约 60 行，覆盖主题「tipLink」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.368 `src/services/tips/tipScheduler.test.ts`

该文件位于 `src/services/tips`，约 225 行，覆盖主题「tipScheduler」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.369 `src/services/tokenEstimation.test.ts`

该文件位于 `src/services`，约 80 行，覆盖主题「tokenEstimation」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.370 `src/services/tokenModelCompression.test.ts`

该文件位于 `src/services`，约 100 行，覆盖主题「tokenModelCompression」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.371 `src/services/tools/StreamingToolExecutor.test.ts`

该文件位于 `src/services/tools`，约 308 行，覆盖主题「StreamingToolExecutor」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.372 `src/services/tools/queryActivityLease.test.ts`

该文件位于 `src/services/tools`，约 331 行，覆盖主题「queryActivityLease」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.373 `src/services/tools/toolExecution.test.ts`

该文件位于 `src/services/tools`，约 797 行，覆盖主题「toolExecution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.374 `src/services/tools/toolHooks.test.ts`

该文件位于 `src/services/tools`，约 717 行，覆盖主题「toolHooks」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.375 `src/services/tools/toolOrchestration.test.ts`

该文件位于 `src/services/tools`，约 43 行，覆盖主题「toolOrchestration」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.376 `src/services/wiki/conventions.test.ts`

该文件位于 `src/services/wiki`，约 209 行，覆盖主题「conventions」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.377 `src/services/wiki/ingest.test.ts`

该文件位于 `src/services/wiki`，约 48 行，覆盖主题「ingest」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.378 `src/services/wiki/init.test.ts`

该文件位于 `src/services/wiki`，约 58 行，覆盖主题「init」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.379 `src/services/wiki/status.test.ts`

该文件位于 `src/services/wiki`，约 59 行，覆盖主题「status」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.380 `src/skills/bundled/claudeInChrome.test.ts`

该文件位于 `src/skills/bundled`，约 27 行，覆盖主题「claudeInChrome」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.381 `src/skills/bundled/loop.test.ts`

该文件位于 `src/skills/bundled`，约 136 行，覆盖主题「loop」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.382 `src/skills/bundled/pdf.test.ts`

该文件位于 `src/skills/bundled`，约 313 行，覆盖主题「pdf」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.383 `src/skills/bundled/updateConfig.test.ts`

该文件位于 `src/skills/bundled`，约 39 行，覆盖主题「updateConfig」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.384 `src/skills/loadSkillsDir.test.ts`

该文件位于 `src/skills`，约 382 行，覆盖主题「loadSkillsDir」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.385 `src/skills/mcpSkills.test.ts`

该文件位于 `src/skills`，约 173 行，覆盖主题「mcpSkills」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.386 `src/state/AppState.types.test.tsx`

该文件位于 `src/state`，约 105 行，覆盖主题「AppState.typesx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.387 `src/state/teammateViewHelpers.interruptionTrace.test.ts`

该文件位于 `src/state`，约 82 行，覆盖主题「teammateViewHelpers.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.388 `src/tasks/LocalAgentTask/progressTracker.test.ts`

该文件位于 `src/tasks/LocalAgentTask`，约 183 行，覆盖主题「progressTracker」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.389 `src/tasks/LocalMainSessionTask.test.ts`

该文件位于 `src/tasks`，约 408 行，覆盖主题「LocalMainSessionTask」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.390 `src/test/sharedMutationLock.test.ts`

该文件位于 `src/test`，约 45 行，覆盖主题「sharedMutationLock」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.391 `src/tools.lsp.test.ts`

该文件位于 `src`，约 81 行，覆盖主题「tools.lsp」。不变量：未连接即过滤；连接后可用。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.392 `src/tools/AgentTool/AgentTool.copilotScheduling.test.ts`

该文件位于 `src/tools/AgentTool`，约 252 行，覆盖主题「AgentTool.copilotScheduling」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.393 `src/tools/AgentTool/AgentTool.routing.test.ts`

该文件位于 `src/tools/AgentTool`，约 381 行，覆盖主题「AgentTool.routing」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.394 `src/tools/AgentTool/AgentTool.schema.test.ts`

该文件位于 `src/tools/AgentTool`，约 378 行，覆盖主题「AgentTool.schema」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.395 `src/tools/AgentTool/AgentTool.teammateModel.test.ts`

该文件位于 `src/tools/AgentTool`，约 506 行，覆盖主题「AgentTool.teammateModel」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.396 `src/tools/AgentTool/built-in/codeReviewerAgent.test.ts`

该文件位于 `src/tools/AgentTool/built-in`，约 262 行，覆盖主题「codeReviewerAgent」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.397 `src/tools/AgentTool/loadAgentsDir.test.ts`

该文件位于 `src/tools/AgentTool`，约 316 行，覆盖主题「loadAgentsDir」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.398 `src/tools/AgentTool/prompt.test.ts`

该文件位于 `src/tools/AgentTool`，约 61 行，覆盖主题「prompt」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.399 `src/tools/AgentTool/resumeAgent.test.ts`

该文件位于 `src/tools/AgentTool`，约 176 行，覆盖主题「resumeAgent」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.400 `src/tools/AgentTool/runAgent.persistence.test.ts`

该文件位于 `src/tools/AgentTool`，约 99 行，覆盖主题「runAgent.persistence」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.401 `src/tools/AgentTool/runAgent.routing.test.ts`

该文件位于 `src/tools/AgentTool`，约 304 行，覆盖主题「runAgent.routing」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.402 `src/tools/BashTool/BashTool.errorOutput.test.ts`

该文件位于 `src/tools/BashTool`，约 512 行，覆盖主题「BashTool.errorOutput」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.403 `src/tools/BashTool/BashTool.sandboxAnalysis.test.ts`

该文件位于 `src/tools/BashTool`，约 190 行，覆盖主题「BashTool.sandboxAnalysis」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.404 `src/tools/BashTool/bashCommandAnalysis.test.ts`

该文件位于 `src/tools/BashTool`，约 138 行，覆盖主题「bashCommandAnalysis」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.405 `src/tools/BashTool/bashPermissions.test.ts`

该文件位于 `src/tools/BashTool`，约 531 行，覆盖主题「bashPermissions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.406 `src/tools/BashTool/bashSecurity.safety.test.ts`

该文件位于 `src/tools/BashTool`，约 30 行，覆盖主题「bashSecurity.safety」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.407 `src/tools/BashTool/bashSecurity.test.ts`

该文件位于 `src/tools/BashTool`，约 361 行，覆盖主题「bashSecurity」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.408 `src/tools/BashTool/commandSemantics.test.ts`

该文件位于 `src/tools/BashTool`，约 595 行，覆盖主题「commandSemantics」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.409 `src/tools/BashTool/modeValidation.test.ts`

该文件位于 `src/tools/BashTool`，约 53 行，覆盖主题「modeValidation」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.410 `src/tools/BashTool/pathValidation.test.ts`

该文件位于 `src/tools/BashTool`，约 39 行，覆盖主题「pathValidation」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.411 `src/tools/BashTool/readOnlyValidation.test.ts`

该文件位于 `src/tools/BashTool`，约 41 行，覆盖主题「readOnlyValidation」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.412 `src/tools/BashTool/sedEditParser.test.ts`

该文件位于 `src/tools/BashTool`，约 531 行，覆盖主题「sedEditParser」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.413 `src/tools/BashTool/shouldUseSandbox.test.ts`

该文件位于 `src/tools/BashTool`，约 112 行，覆盖主题「shouldUseSandbox」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.414 `src/tools/BashTool/utils.test.ts`

该文件位于 `src/tools/BashTool`，约 300 行，覆盖主题「utils」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.415 `src/tools/CtxInspectTool/CtxInspectTool.test.ts`

该文件位于 `src/tools/CtxInspectTool`，约 79 行，覆盖主题「CtxInspectTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.416 `src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.test.ts`

该文件位于 `src/tools/ExitPlanModeTool`，约 197 行，覆盖主题「ExitPlanModeV2Tool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.417 `src/tools/FileEditTool/utils.test.ts`

该文件位于 `src/tools/FileEditTool`，约 235 行，覆盖主题「utils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.418 `src/tools/FileReadTool/FileReadTool.oversized.test.ts`

该文件位于 `src/tools/FileReadTool`，约 54 行，覆盖主题「FileReadTool.oversized」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.419 `src/tools/FileReadTool/prompt.vision.test.ts`

该文件位于 `src/tools/FileReadTool`，约 130 行，覆盖主题「prompt.vision」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.420 `src/tools/GrepTool/GrepTool.test.ts`

该文件位于 `src/tools/GrepTool`，约 101 行，覆盖主题「GrepTool」。不变量：忽略规则生效；相对路径输出。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.421 `src/tools/MCPTool/MCPTool.test.ts`

该文件位于 `src/tools/MCPTool`，约 261 行，覆盖主题「MCPTool」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.422 `src/tools/PowerShellTool/PowerShellTool.errorOutput.test.ts`

该文件位于 `src/tools/PowerShellTool`，约 281 行，覆盖主题「PowerShellTool.errorOutput」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.423 `src/tools/PowerShellTool/commandSemantics.test.ts`

该文件位于 `src/tools/PowerShellTool`，约 493 行，覆盖主题「commandSemantics」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.424 `src/tools/PowerShellTool/pathValidation.protoName.test.ts`

该文件位于 `src/tools/PowerShellTool`，约 96 行，覆盖主题「pathValidation.protoName」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.425 `src/tools/PowerShellTool/powershellPermissions.test.ts`

该文件位于 `src/tools/PowerShellTool`，约 330 行，覆盖主题「powershellPermissions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.426 `src/tools/RepoMapTool/RepoMapTool.test.ts`

该文件位于 `src/tools/RepoMapTool`，约 299 行，覆盖主题「RepoMapTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.427 `src/tools/SendMessageTool/shutdownInterruptionTrace.test.ts`

该文件位于 `src/tools/SendMessageTool`，约 66 行，覆盖主题「shutdownInterruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.428 `src/tools/SkillTool/SkillTool.test.ts`

该文件位于 `src/tools/SkillTool`，约 98 行，覆盖主题「SkillTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.429 `src/tools/SkillTool/prompt.test.ts`

该文件位于 `src/tools/SkillTool`，约 76 行，覆盖主题「prompt」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.430 `src/tools/SnipTool/SnipTool.test.ts`

该文件位于 `src/tools/SnipTool`，约 46 行，覆盖主题「SnipTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.431 `src/tools/SyntheticOutputTool/SyntheticOutputTool.test.ts`

该文件位于 `src/tools/SyntheticOutputTool`，约 102 行，覆盖主题「SyntheticOutputTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.432 `src/tools/TaskOutputTool/TaskOutputTool.activity.test.ts`

该文件位于 `src/tools/TaskOutputTool`，约 75 行，覆盖主题「TaskOutputTool.activity」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.433 `src/tools/WebFetchTool/applyPromptFallback.test.ts`

该文件位于 `src/tools/WebFetchTool`，约 96 行，覆盖主题「applyPromptFallback」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.434 `src/tools/WebFetchTool/domainCheck.test.ts`

该文件位于 `src/tools/WebFetchTool`，约 87 行，覆盖主题「domainCheck」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.435 `src/tools/WebSearchTool/WebSearchTool.test.ts`

该文件位于 `src/tools/WebSearchTool`，约 106 行，覆盖主题「WebSearchTool」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.436 `src/tools/WebSearchTool/providers/brave.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 156 行，覆盖主题「brave」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.437 `src/tools/WebSearchTool/providers/custom.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 457 行，覆盖主题「custom」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.438 `src/tools/WebSearchTool/providers/duckduckgo.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 103 行，覆盖主题「duckduckgo」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.439 `src/tools/WebSearchTool/providers/exa.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 154 行，覆盖主题「exa」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.440 `src/tools/WebSearchTool/providers/firecrawl.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 59 行，覆盖主题「firecrawl」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.441 `src/tools/WebSearchTool/providers/index.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 309 行，覆盖主题「index」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.442 `src/tools/WebSearchTool/providers/timeout.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 177 行，覆盖主题「timeout」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.443 `src/tools/WebSearchTool/providers/types.test.ts`

该文件位于 `src/tools/WebSearchTool/providers`，约 262 行，覆盖主题「types」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.444 `src/tools/firecrawl/client.test.ts`

该文件位于 `src/tools/firecrawl`，约 233 行，覆盖主题「client」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.445 `src/tools/shellToolResultMappers.test.ts`

该文件位于 `src/tools`，约 71 行，覆盖主题「shellToolResultMappers」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.446 `src/types/utils.types.test.ts`

该文件位于 `src/types`，约 63 行，覆盖主题「utils.types」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.447 `src/upstreamproxy/upstreamproxy.test.ts`

该文件位于 `src/upstreamproxy`，约 42 行，覆盖主题「upstreamproxy」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.448 `src/utils/Cursor.nfc.test.ts`

该文件位于 `src/utils`，约 31 行，覆盖主题「Cursor.nfc」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.449 `src/utils/QueryGuard.test.ts`

该文件位于 `src/utils`，约 821 行，覆盖主题「QueryGuard」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.450 `src/utils/ShellCommand.test.ts`

该文件位于 `src/utils`，约 125 行，覆盖主题「ShellCommand」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.451 `src/utils/abortReasons.test.ts`

该文件位于 `src/utils`，约 197 行，覆盖主题「abortReasons」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.452 `src/utils/advisor.modelGate.test.ts`

该文件位于 `src/utils`，约 37 行，覆盖主题「advisor.modelGate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.453 `src/utils/analyzeContext.mcp.test.ts`

该文件位于 `src/utils`，约 185 行，覆盖主题「analyzeContext.mcp」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.454 `src/utils/analyzeContext.messageBreakdown.test.ts`

该文件位于 `src/utils`，约 142 行，覆盖主题「analyzeContext.messageBreakdown」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.455 `src/utils/anthropicAttribution.test.ts`

该文件位于 `src/utils`，约 233 行，覆盖主题「anthropicAttribution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.456 `src/utils/anthropicBaseUrl.test.ts`

该文件位于 `src/utils`，约 200 行，覆盖主题「anthropicBaseUrl」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.457 `src/utils/api.swarmFields.test.ts`

该文件位于 `src/utils`，约 46 行，覆盖主题「api.swarmFields」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.458 `src/utils/api.test.ts`

该文件位于 `src/utils`，约 105 行，覆盖主题「api」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.459 `src/utils/apiPreconnect.test.ts`

该文件位于 `src/utils`，约 134 行，覆盖主题「apiPreconnect」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.460 `src/utils/argumentSubstitution.test.ts`

该文件位于 `src/utils`，约 82 行，覆盖主题「argumentSubstitution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.461 `src/utils/atomicReplace.test.ts`

该文件位于 `src/utils`，约 273 行，覆盖主题「atomicReplace」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.462 `src/utils/attachments.contextEfficiency.test.ts`

该文件位于 `src/utils`，约 121 行，覆盖主题「attachments.contextEfficiency」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.463 `src/utils/attachments.editedImage.test.ts`

该文件位于 `src/utils`，约 82 行，覆盖主题「attachments.editedImage」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.464 `src/utils/attachments.extractors.test.ts`

该文件位于 `src/utils`，约 102 行，覆盖主题「attachments.extractors」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.465 `src/utils/attachments.lspDiagnostics.test.ts`

该文件位于 `src/utils`，约 269 行，覆盖主题「attachments.lspDiagnostics」。不变量：未连接即过滤；连接后可用。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.466 `src/utils/attachments.nestedDirs.test.ts`

该文件位于 `src/utils`，约 116 行，覆盖主题「attachments.nestedDirs」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.467 `src/utils/attachments.performance.test.ts`

该文件位于 `src/utils`，约 221 行，覆盖主题「attachments.performance」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.468 `src/utils/attachments.ultracode.test.ts`

该文件位于 `src/utils`，约 113 行，覆盖主题「attachments.ultracode」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.469 `src/utils/attachments.ultrathink.test.ts`

该文件位于 `src/utils`，约 62 行，覆盖主题「attachments.ultrathink」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.470 `src/utils/attribution.test.ts`

该文件位于 `src/utils`，约 434 行，覆盖主题「attribution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.471 `src/utils/auth.test.ts`

该文件位于 `src/utils`，约 158 行，覆盖主题「auth」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.472 `src/utils/bash/shellPrefix.test.ts`

该文件位于 `src/utils/bash`，约 94 行，覆盖主题「shellPrefix」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.473 `src/utils/bash/shellQuote.test.ts`

该文件位于 `src/utils/bash`，约 81 行，覆盖主题「shellQuote」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.474 `src/utils/betas.test.ts`

该文件位于 `src/utils`，约 341 行，覆盖主题「betas」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.475 `src/utils/boundedAsync.test.ts`

该文件位于 `src/utils`，约 213 行，覆盖主题「boundedAsync」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.476 `src/utils/claudeDesktop.test.ts`

该文件位于 `src/utils`，约 65 行，覆盖主题「claudeDesktop」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.477 `src/utils/claudeInChrome/startup.test.ts`

该文件位于 `src/utils/claudeInChrome`，约 133 行，覆盖主题「startup」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.478 `src/utils/cleanup.test.ts`

该文件位于 `src/utils`，约 48 行，覆盖主题「cleanup」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.479 `src/utils/codeIndexing.proto.test.ts`

该文件位于 `src/utils`，约 37 行，覆盖主题「codeIndexing.proto」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.480 `src/utils/codexCredentials.test.ts`

该文件位于 `src/utils`，约 957 行，覆盖主题「codexCredentials」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.481 `src/utils/combinedAbortSignal.test.ts`

该文件位于 `src/utils`，约 190 行，覆盖主题「combinedAbortSignal」。不变量：中止原因与 transcript 文案一致，超时不是用户打断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.482 `src/utils/commitAttribution.modelName.test.ts`

该文件位于 `src/utils`，约 17 行，覆盖主题「commitAttribution.modelName」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.483 `src/utils/config.backupRecovery.test.ts`

该文件位于 `src/utils`，约 413 行，覆盖主题「config.backupRecovery」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.484 `src/utils/config.deferredWrite.test.ts`

该文件位于 `src/utils`，约 70 行，覆盖主题「config.deferredWrite」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.485 `src/utils/config.showCacheStats.test.ts`

该文件位于 `src/utils`，约 126 行，覆盖主题「config.showCacheStats」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.486 `src/utils/context.test.ts`

该文件位于 `src/utils`，约 1273 行，覆盖主题「context」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.487 `src/utils/contextPartitioning.test.ts`

该文件位于 `src/utils`，约 86 行，覆盖主题「contextPartitioning」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.488 `src/utils/conversationArc.test.ts`

该文件位于 `src/utils`，约 616 行，覆盖主题「conversationArc」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.489 `src/utils/conversationCache.test.ts`

该文件位于 `src/utils`，约 99 行，覆盖主题「conversationCache」。不变量：命中/未命中计数；probe 不污染用户 transcript。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.490 `src/utils/conversationRecovery.hooks.test.ts`

该文件位于 `src/utils`，约 200 行，覆盖主题「conversationRecovery.hooks」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.491 `src/utils/conversationRecovery.test.ts`

该文件位于 `src/utils`，约 756 行，覆盖主题「conversationRecovery」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.492 `src/utils/copilotOptimization.test.ts`

该文件位于 `src/utils`，约 228 行，覆盖主题「copilotOptimization」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.493 `src/utils/cwd.test.ts`

该文件位于 `src/utils`，约 39 行，覆盖主题「cwd」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.494 `src/utils/dangerousSkipFlags.test.ts`

该文件位于 `src/utils`，约 51 行，覆盖主题「dangerousSkipFlags」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.495 `src/utils/deferredConfigWrites.test.ts`

该文件位于 `src/utils`，约 207 行，覆盖主题「deferredConfigWrites」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.496 `src/utils/diagnostics/issueReport.test.ts`

该文件位于 `src/utils/diagnostics`，约 348 行，覆盖主题「issueReport」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.497 `src/utils/diagnostics/redaction.test.ts`

该文件位于 `src/utils/diagnostics`，约 779 行，覆盖主题「redaction」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.498 `src/utils/diff.test.ts`

该文件位于 `src/utils`，约 85 行，覆盖主题「diff」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.499 `src/utils/doctorDiagnostic.settingsPath.test.ts`

该文件位于 `src/utils`，约 97 行，覆盖主题「doctorDiagnostic.settingsPath」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.500 `src/utils/doctorDiagnostic.test.ts`

该文件位于 `src/utils`，约 32 行，覆盖主题「doctorDiagnostic」。不变量：缺 rg/Node 版本时给出可操作诊断。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.501 `src/utils/doomLoop.test.ts`

该文件位于 `src/utils`，约 131 行，覆盖主题「doomLoop」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.502 `src/utils/dragDropPaths.test.ts`

该文件位于 `src/utils`，约 113 行，覆盖主题「dragDropPaths」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.503 `src/utils/effort.codex.test.ts`

该文件位于 `src/utils`，约 1732 行，覆盖主题「effort.codex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.504 `src/utils/effort.test.ts`

该文件位于 `src/utils`，约 440 行，覆盖主题「effort」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.505 `src/utils/effort.ultracode-display.test.ts`

该文件位于 `src/utils`，约 123 行，覆盖主题「effort.ultracode-display」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.506 `src/utils/env.test.ts`

该文件位于 `src/utils`，约 233 行，覆盖主题「env」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.507 `src/utils/envFile.test.ts`

该文件位于 `src/utils`，约 539 行，覆盖主题「envFile」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.508 `src/utils/envProviderOption.test.ts`

该文件位于 `src/utils`，约 134 行，覆盖主题「envProviderOption」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.509 `src/utils/envValidation.test.ts`

该文件位于 `src/utils`，约 43 行，覆盖主题「envValidation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.510 `src/utils/execFileNoThrow.test.ts`

该文件位于 `src/utils`，约 69 行，覆盖主题「execFileNoThrow」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.511 `src/utils/exportFormats.test.ts`

该文件位于 `src/utils`，约 245 行，覆盖主题「exportFormats」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.512 `src/utils/exportRenderer.test.ts`

该文件位于 `src/utils`，约 976 行，覆盖主题「exportRenderer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.513 `src/utils/extraUsage.test.ts`

该文件位于 `src/utils`，约 44 行，覆盖主题「extraUsage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.514 `src/utils/fastMode.test.ts`

该文件位于 `src/utils`，约 319 行，覆盖主题「fastMode」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.515 `src/utils/file.test.ts`

该文件位于 `src/utils`，约 69 行，覆盖主题「file」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.516 `src/utils/format.test.ts`

该文件位于 `src/utils`，约 65 行，覆盖主题「format」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.517 `src/utils/fpsTracker.test.ts`

该文件位于 `src/utils`，约 51 行，覆盖主题「fpsTracker」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.518 `src/utils/frontmatterParser.test.ts`

该文件位于 `src/utils`，约 55 行，覆盖主题「frontmatterParser」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.519 `src/utils/fsOperations.mkdir.test.ts`

该文件位于 `src/utils`，约 58 行，覆盖主题「fsOperations.mkdir」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.520 `src/utils/geminiAuth.optionalRuntime.test.ts`

该文件位于 `src/utils`，约 61 行，覆盖主题「geminiAuth.optionalRuntime」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.521 `src/utils/geminiAuth.test.ts`

该文件位于 `src/utils`，约 283 行，覆盖主题「geminiAuth」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.522 `src/utils/geminiCredentials.test.ts`

该文件位于 `src/utils`，约 73 行，覆盖主题「geminiCredentials」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.523 `src/utils/gitDiff.test.ts`

该文件位于 `src/utils`，约 133 行，覆盖主题「gitDiff」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.524 `src/utils/gitSettings.test.ts`

该文件位于 `src/utils`，约 107 行，覆盖主题「gitSettings」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.525 `src/utils/githubModelsCredentials.hydrate.test.ts`

该文件位于 `src/utils`，约 123 行，覆盖主题「githubModelsCredentials.hydrate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.526 `src/utils/githubModelsCredentials.refresh.test.ts`

该文件位于 `src/utils`，约 422 行，覆盖主题「githubModelsCredentials.refresh」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.527 `src/utils/githubModelsCredentials.test.ts`

该文件位于 `src/utils`，约 160 行，覆盖主题「githubModelsCredentials」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.528 `src/utils/globalPackageManager.test.ts`

该文件位于 `src/utils`，约 124 行，覆盖主题「globalPackageManager」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.529 `src/utils/governancePolicy.test.ts`

该文件位于 `src/utils`，约 88 行，覆盖主题「governancePolicy」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.530 `src/utils/gracefulShutdown.interruptionTrace.test.ts`

该文件位于 `src/utils`，约 60 行，覆盖主题「gracefulShutdown.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.531 `src/utils/handlePromptSubmit.test.ts`

该文件位于 `src/utils`，约 732 行，覆盖主题「handlePromptSubmit」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.532 `src/utils/heapDumpService.test.ts`

该文件位于 `src/utils`，约 66 行，覆盖主题「heapDumpService」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.533 `src/utils/highlightMatch.test.tsx`

该文件位于 `src/utils`，约 69 行，覆盖主题「highlightMatchx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.534 `src/utils/hookChains.integration.test.ts`

该文件位于 `src/utils`，约 400 行，覆盖主题「hookChains.integration」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.535 `src/utils/hookChains.test.ts`

该文件位于 `src/utils`，约 485 行，覆盖主题「hookChains」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.536 `src/utils/hooks.fallbackAgent.test.ts`

该文件位于 `src/utils`，约 13 行，覆盖主题「hooks.fallbackAgent」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.537 `src/utils/hooks/hooksSettings.test.ts`

该文件位于 `src/utils/hooks`，约 13 行，覆盖主题「hooksSettings」。不变量：hook 拒绝能阻断；输出进入规定消息类型。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.538 `src/utils/http.test.ts`

该文件位于 `src/utils`，约 49 行，覆盖主题「http」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.539 `src/utils/hybridContextStrategy.test.ts`

该文件位于 `src/utils`，约 230 行，覆盖主题「hybridContextStrategy」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.540 `src/utils/imageMockLifecycle.consumer.test.ts`

该文件位于 `src/utils`，约 50 行，覆盖主题「imageMockLifecycle.consumer」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.541 `src/utils/imagePaste.clipboard-error.test.ts`

该文件位于 `src/utils`，约 86 行，覆盖主题「imagePaste.clipboard-error」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.542 `src/utils/imagePaste.test.ts`

该文件位于 `src/utils`，约 115 行，覆盖主题「imagePaste」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.543 `src/utils/imagePaste.win32.test.ts`

该文件位于 `src/utils`，约 247 行，覆盖主题「imagePaste.win32」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.544 `src/utils/imageResizer.test.ts`

该文件位于 `src/utils`，约 842 行，覆盖主题「imageResizer」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.545 `src/utils/incrementalTokenCounter.test.ts`

该文件位于 `src/utils`，约 300 行，覆盖主题「incrementalTokenCounter」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.546 `src/utils/interactivity.test.ts`

该文件位于 `src/utils`，约 110 行，覆盖主题「interactivity」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.547 `src/utils/interruptionCorrection.test.ts`

该文件位于 `src/utils`，约 615 行，覆盖主题「interruptionCorrection」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.548 `src/utils/interruptionTrace.test.ts`

该文件位于 `src/utils`，约 923 行，覆盖主题「interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.549 `src/utils/knowledgeGraph.test.ts`

该文件位于 `src/utils`，约 537 行，覆盖主题「knowledgeGraph」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.550 `src/utils/log.test.ts`

该文件位于 `src/utils`，约 110 行，覆盖主题「log」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.551 `src/utils/managedEnv.test.ts`

该文件位于 `src/utils`，约 169 行，覆盖主题「managedEnv」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.552 `src/utils/markdownConfigLoader.scaling.test.ts`

该文件位于 `src/utils`，约 155 行，覆盖主题「markdownConfigLoader.scaling」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.553 `src/utils/maxActiveMessages.test.ts`

该文件位于 `src/utils`，约 69 行，覆盖主题「maxActiveMessages」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.554 `src/utils/mcpValidation.test.ts`

该文件位于 `src/utils`，约 123 行，覆盖主题「mcpValidation」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.555 `src/utils/messageQueueManager.test.ts`

该文件位于 `src/utils`，约 54 行，覆盖主题「messageQueueManager」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.556 `src/utils/messages/apiTransform.test.ts`

该文件位于 `src/utils/messages`，约 417 行，覆盖主题「apiTransform」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.557 `src/utils/messages/content.test.ts`

该文件位于 `src/utils/messages`，约 89 行，覆盖主题「content」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.558 `src/utils/messages/factories.test.ts`

该文件位于 `src/utils/messages`，约 198 行，覆盖主题「factories」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.559 `src/utils/messages/normalize.test.ts`

该文件位于 `src/utils/messages`，约 172 行，覆盖主题「normalize」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.560 `src/utils/messages/planMode.test.ts`

该文件位于 `src/utils/messages`，约 92 行，覆盖主题「planMode」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.561 `src/utils/messages/streaming.test.ts`

该文件位于 `src/utils/messages`，约 78 行，覆盖主题「streaming」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.562 `src/utils/messages/systemFactories.test.ts`

该文件位于 `src/utils/messages`，约 233 行，覆盖主题「systemFactories」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.563 `src/utils/messages/toolPairing.test.ts`

该文件位于 `src/utils/messages`，约 361 行，覆盖主题「toolPairing」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.564 `src/utils/model/agent.test.ts`

该文件位于 `src/utils/model`，约 608 行，覆盖主题「agent」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.565 `src/utils/model/minimaxModels.test.ts`

该文件位于 `src/utils/model`，约 33 行，覆盖主题「minimaxModels」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.566 `src/utils/model/model.github.test.ts`

该文件位于 `src/utils/model`，约 76 行，覆盖主题「model.github」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.567 `src/utils/model/model.openai-shim-providers.test.ts`

该文件位于 `src/utils/model`，约 591 行，覆盖主题「model.openai-shim-providers」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.568 `src/utils/model/modelOptions.catalogDedup.test.ts`

该文件位于 `src/utils/model`，约 401 行，覆盖主题「modelOptions.catalogDedup」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.569 `src/utils/model/modelOptions.codexRecovery.test.ts`

该文件位于 `src/utils/model`，约 104 行，覆盖主题「modelOptions.codexRecovery」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.570 `src/utils/model/modelOptions.crossProfile.test.ts`

该文件位于 `src/utils/model`，约 921 行，覆盖主题「modelOptions.crossProfile」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.571 `src/utils/model/modelOptions.gateways.test.ts`

该文件位于 `src/utils/model`，约 273 行，覆盖主题「modelOptions.gateways」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.572 `src/utils/model/modelOptions.github.test.ts`

该文件位于 `src/utils/model`，约 132 行，覆盖主题「modelOptions.github」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.573 `src/utils/model/modelOptions.hicap.test.ts`

该文件位于 `src/utils/model`，约 171 行，覆盖主题「modelOptions.hicap」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.574 `src/utils/model/modelOptions.switchMarker.test.ts`

该文件位于 `src/utils/model`，约 46 行，覆盖主题「modelOptions.switchMarker」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.575 `src/utils/model/modelOptions.xiaomi-mimo.test.ts`

该文件位于 `src/utils/model`，约 101 行，覆盖主题「modelOptions.xiaomi-mimo」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.576 `src/utils/model/modelStrings.github.test.ts`

该文件位于 `src/utils/model`，约 71 行，覆盖主题「modelStrings.github」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.577 `src/utils/model/nvidiaNimModels.test.ts`

该文件位于 `src/utils/model`，约 186 行，覆盖主题「nvidiaNimModels」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.578 `src/utils/model/openaiContextWindows.test.ts`

该文件位于 `src/utils/model`，约 167 行，覆盖主题「openaiContextWindows」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.579 `src/utils/model/openaiModelDiscovery.test.ts`

该文件位于 `src/utils/model`，约 197 行，覆盖主题「openaiModelDiscovery」。不变量：消息/工具转换可逆或可接受损失；非法 schema 被消毒。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.580 `src/utils/model/parseUserSpecifiedModel.bestTag.test.ts`

该文件位于 `src/utils/model`，约 30 行，覆盖主题「parseUserSpecifiedModel.bestTag」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.581 `src/utils/model/parseUserSpecifiedModel.codexTag.test.ts`

该文件位于 `src/utils/model`，约 115 行，覆盖主题「parseUserSpecifiedModel.codexTag」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.582 `src/utils/model/parseUserSpecifiedModel.disable1m.test.ts`

该文件位于 `src/utils/model`，约 143 行，覆盖主题「parseUserSpecifiedModel.disable1m」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.583 `src/utils/model/providers.test.ts`

该文件位于 `src/utils/model`，约 341 行，覆盖主题「providers」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.584 `src/utils/model/routeCatalogOptions.test.ts`

该文件位于 `src/utils/model`，约 114 行，覆盖主题「routeCatalogOptions」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.585 `src/utils/modelCost.modelGate.test.ts`

该文件位于 `src/utils`，约 276 行，覆盖主题「modelCost.modelGate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.586 `src/utils/multiTurnContext.test.ts`

该文件位于 `src/utils`，约 225 行，覆盖主题「multiTurnContext」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.587 `src/utils/nodeRuntime.test.ts`

该文件位于 `src/utils`，约 59 行，覆盖主题「nodeRuntime」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.588 `src/utils/openclaudeInstallSurfaces.test.ts`

该文件位于 `src/utils`，约 553 行，覆盖主题「openclaudeInstallSurfaces」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.589 `src/utils/openclaudePaths.test.ts`

该文件位于 `src/utils`，约 380 行，覆盖主题「openclaudePaths」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.590 `src/utils/openclaudeUiSurfaces.test.ts`

该文件位于 `src/utils`，约 218 行，覆盖主题「openclaudeUiSurfaces」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.591 `src/utils/opencodeProfile.test.ts`

该文件位于 `src/utils`，约 440 行，覆盖主题「opencodeProfile」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.592 `src/utils/optionalRuntimeModule.test.ts`

该文件位于 `src/utils`，约 99 行，覆盖主题「optionalRuntimeModule」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.593 `src/utils/packageManagerUpdateGuidance.test.ts`

该文件位于 `src/utils`，约 64 行，覆盖主题「packageManagerUpdateGuidance」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.594 `src/utils/permissions/PermissionUpdate.test.ts`

该文件位于 `src/utils/permissions`，约 25 行，覆盖主题「PermissionUpdate」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.595 `src/utils/permissions/bypassPermissionsKillswitch.test.ts`

该文件位于 `src/utils/permissions`，约 104 行，覆盖主题「bypassPermissionsKillswitch」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.596 `src/utils/permissions/dangerousModePrompt.test.ts`

该文件位于 `src/utils/permissions`，约 78 行，覆盖主题「dangerousModePrompt」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.597 `src/utils/permissions/dangerousModePromptFlow.test.tsx`

该文件位于 `src/utils/permissions`，约 144 行，覆盖主题「dangerousModePromptFlowx」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.598 `src/utils/permissions/dangerousModePromptRuntime.test.ts`

该文件位于 `src/utils/permissions`，约 71 行，覆盖主题「dangerousModePromptRuntime」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.599 `src/utils/permissions/filesystem.test.ts`

该文件位于 `src/utils/permissions`，约 294 行，覆盖主题「filesystem」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.600 `src/utils/permissions/getNextPermissionMode.test.ts`

该文件位于 `src/utils/permissions`，约 25 行，覆盖主题「getNextPermissionMode」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.601 `src/utils/permissions/pathValidation.expandTilde.test.ts`

该文件位于 `src/utils/permissions`，约 43 行，覆盖主题「pathValidation.expandTilde」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.602 `src/utils/permissions/permissionRuleParser.protoName.test.ts`

该文件位于 `src/utils/permissions`，约 68 行，覆盖主题「permissionRuleParser.protoName」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.603 `src/utils/permissions/permissionSetup.test.ts`

该文件位于 `src/utils/permissions`，约 344 行，覆盖主题「permissionSetup」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.604 `src/utils/permissions/permissions.headlessPlanHooks.test.ts`

该文件位于 `src/utils/permissions`，约 1253 行，覆盖主题「permissions.headlessPlanHooks」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.605 `src/utils/permissions/permissions.test.ts`

该文件位于 `src/utils/permissions`，约 831 行，覆盖主题「permissions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.606 `src/utils/permissions/planFilePath.test.ts`

该文件位于 `src/utils/permissions`，约 544 行，覆盖主题「planFilePath」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.607 `src/utils/permissions/safetyLevel.test.ts`

该文件位于 `src/utils/permissions`，约 35 行，覆盖主题「safetyLevel」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.608 `src/utils/permissions/yoloClassifier.test.ts`

该文件位于 `src/utils/permissions`，约 79 行，覆盖主题「yoloClassifier」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.609 `src/utils/plugins/gitEnv.test.ts`

该文件位于 `src/utils/plugins`，约 113 行，覆盖主题「gitEnv」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.610 `src/utils/plugins/loadPluginAgents.test.ts`

该文件位于 `src/utils/plugins`，约 143 行，覆盖主题「loadPluginAgents」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.611 `src/utils/plugins/lspRecommendation.test.ts`

该文件位于 `src/utils/plugins`，约 269 行，覆盖主题「lspRecommendation」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.612 `src/utils/plugins/marketplaceHostPattern.test.ts`

该文件位于 `src/utils/plugins`，约 97 行，覆盖主题「marketplaceHostPattern」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.613 `src/utils/plugins/marketplaceManager.test.ts`

该文件位于 `src/utils/plugins`，约 902 行，覆盖主题「marketplaceManager」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.614 `src/utils/plugins/officialMarketplaceStartupCheck.test.ts`

该文件位于 `src/utils/plugins`，约 209 行，覆盖主题「officialMarketplaceStartupCheck」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.615 `src/utils/plugins/pluginLoader.test.ts`

该文件位于 `src/utils/plugins`，约 509 行，覆盖主题「pluginLoader」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.616 `src/utils/plugins/reconciler.test.ts`

该文件位于 `src/utils/plugins`，约 79 行，覆盖主题「reconciler」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.617 `src/utils/plugins/validateOfficialNameSource.test.ts`

该文件位于 `src/utils/plugins`，约 68 行，覆盖主题「validateOfficialNameSource」。不变量：校验失败不可启用；Windows 拷贝错误被容忍或重试。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.618 `src/utils/preflightChecks.test.ts`

该文件位于 `src/utils`，约 271 行，覆盖主题「preflightChecks」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.619 `src/utils/printFlag.test.ts`

该文件位于 `src/utils`，约 217 行，覆盖主题「printFlag」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.620 `src/utils/processUserInput/processBashCommand.test.tsx`

该文件位于 `src/utils/processUserInput`，约 117 行，覆盖主题「processBashCommandx」。不变量：危险命令被标记；只读命令可并发；路径不逃逸。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.621 `src/utils/processUserInput/processSlashCommand.goal.test.tsx`

该文件位于 `src/utils/processUserInput`，约 50 行，覆盖主题「processSlashCommand.goalx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.622 `src/utils/processUserInput/processSlashCommand.test.ts`

该文件位于 `src/utils/processUserInput`，约 69 行，覆盖主题「processSlashCommand」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.623 `src/utils/profilerRetention.test.ts`

该文件位于 `src/utils`，约 388 行，覆盖主题「profilerRetention」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.624 `src/utils/projectInstructions.test.ts`

该文件位于 `src/utils`，约 105 行，覆盖主题「projectInstructions」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.625 `src/utils/promptEditor.test.ts`

该文件位于 `src/utils`，约 27 行，覆盖主题「promptEditor」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.626 `src/utils/promptShellExecution.test.ts`

该文件位于 `src/utils`，约 320 行，覆盖主题「promptShellExecution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.627 `src/utils/providerAutoDetect.test.ts`

该文件位于 `src/utils`，约 399 行，覆盖主题「providerAutoDetect」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.628 `src/utils/providerCustomHeaders.test.ts`

该文件位于 `src/utils`，约 58 行，覆盖主题「providerCustomHeaders」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.629 `src/utils/providerDiscovery.test.ts`

该文件位于 `src/utils`，约 421 行，覆盖主题「providerDiscovery」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.630 `src/utils/providerFallback.test.ts`

该文件位于 `src/utils`，约 149 行，覆盖主题「providerFallback」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.631 `src/utils/providerFlag.test.ts`

该文件位于 `src/utils`，约 1850 行，覆盖主题「providerFlag」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.632 `src/utils/providerModels.test.ts`

该文件位于 `src/utils`，约 146 行，覆盖主题「providerModels」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.633 `src/utils/providerProfile.test.ts`

该文件位于 `src/utils`，约 3475 行，覆盖主题「providerProfile」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.634 `src/utils/providerProfiles.test.ts`

该文件位于 `src/utils`，约 5469 行，覆盖主题「providerProfiles」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.635 `src/utils/providerRecommendation.test.ts`

该文件位于 `src/utils`，约 194 行，覆盖主题「providerRecommendation」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.636 `src/utils/providerSecrets.test.ts`

该文件位于 `src/utils`，约 336 行，覆盖主题「providerSecrets」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.637 `src/utils/providerStartupOverrides.test.ts`

该文件位于 `src/utils`，约 62 行，覆盖主题「providerStartupOverrides」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.638 `src/utils/providerValidation.test.ts`

该文件位于 `src/utils`，约 961 行，覆盖主题「providerValidation」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.639 `src/utils/proxy.test.ts`

该文件位于 `src/utils`，约 92 行，覆盖主题「proxy」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.640 `src/utils/queryEventDriver.test.ts`

该文件位于 `src/utils`，约 79 行，覆盖主题「queryEventDriver」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.641 `src/utils/queryGuardConfig.test.ts`

该文件位于 `src/utils`，约 79 行，覆盖主题「queryGuardConfig」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.642 `src/utils/queryLifecycle.test.ts`

该文件位于 `src/utils`，约 132 行，覆盖主题「queryLifecycle」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.643 `src/utils/readFileInRange.test.ts`

该文件位于 `src/utils`，约 51 行，覆盖主题「readFileInRange」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.644 `src/utils/releaseNotes.test.ts`

该文件位于 `src/utils`，约 121 行，覆盖主题「releaseNotes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.645 `src/utils/relevancePruning.test.ts`

该文件位于 `src/utils`，约 193 行，覆盖主题「relevancePruning」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.646 `src/utils/replInterruption.test.ts`

该文件位于 `src/utils`，约 79 行，覆盖主题「replInterruption」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.647 `src/utils/replayFormat.test.ts`

该文件位于 `src/utils`，约 17 行，覆盖主题「replayFormat」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.648 `src/utils/replayIndex.test.ts`

该文件位于 `src/utils`，约 194 行，覆盖主题「replayIndex」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.649 `src/utils/replayIndexBuilder.test.ts`

该文件位于 `src/utils`，约 123 行，覆盖主题「replayIndexBuilder」。不变量：生成物稳定；scanner 文件目录分类正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.650 `src/utils/reportTask.test.ts`

该文件位于 `src/utils`，约 2191 行，覆盖主题「reportTask」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.651 `src/utils/requestImageValidation.test.ts`

该文件位于 `src/utils`，约 184 行，覆盖主题「requestImageValidation」。不变量：超限报错信息明确；缩放失败走规定回退。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.652 `src/utils/requestLogging.test.ts`

该文件位于 `src/utils`，约 87 行，覆盖主题「requestLogging」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.653 `src/utils/requestSizeBreakdown.test.ts`

该文件位于 `src/utils`，约 492 行，覆盖主题「requestSizeBreakdown」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.654 `src/utils/ripgrep.test.ts`

该文件位于 `src/utils`，约 115 行，覆盖主题「ripgrep」。不变量：忽略规则生效；相对路径输出。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.655 `src/utils/sandbox/sandbox-adapter.test.ts`

该文件位于 `src/utils/sandbox`，约 107 行，覆盖主题「sandbox-adapter」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.656 `src/utils/schemaSanitizer.test.ts`

该文件位于 `src/utils`，约 68 行，覆盖主题「schemaSanitizer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.657 `src/utils/secureStorage/platformStorage.test.ts`

该文件位于 `src/utils/secureStorage`，约 463 行，覆盖主题「platformStorage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.658 `src/utils/sentry.test.ts`

该文件位于 `src/utils`，约 87 行，覆盖主题「sentry」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.659 `src/utils/serializationStability.test.ts`

该文件位于 `src/utils`，约 142 行，覆盖主题「serializationStability」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.660 `src/utils/sessionPersistence.test.ts`

该文件位于 `src/utils`，约 94 行，覆盖主题「sessionPersistence」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.661 `src/utils/sessionRestore.goal.test.ts`

该文件位于 `src/utils`，约 151 行，覆盖主题「sessionRestore.goal」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.662 `src/utils/sessionRestore.test.ts`

该文件位于 `src/utils`，约 424 行，覆盖主题「sessionRestore」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.663 `src/utils/sessionStorage.atomicReplace.test.ts`

该文件位于 `src/utils`，约 807 行，覆盖主题「sessionStorage.atomicReplace」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.664 `src/utils/sessionStorage.liteTag.test.ts`

该文件位于 `src/utils`，约 66 行，覆盖主题「sessionStorage.liteTag」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.665 `src/utils/sessionStorage.test.ts`

该文件位于 `src/utils`，约 901 行，覆盖主题「sessionStorage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.666 `src/utils/sessionTitle.test.ts`

该文件位于 `src/utils`，约 502 行，覆盖主题「sessionTitle」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.667 `src/utils/settings/agentModelsSchema.test.ts`

该文件位于 `src/utils/settings`，约 35 行，覆盖主题「agentModelsSchema」。不变量：子 Agent 工具集被过滤；一步内置类型不带多余 trailer。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.668 `src/utils/settings/allowBypassPermissionsMode.test.ts`

该文件位于 `src/utils/settings`，约 27 行，覆盖主题「allowBypassPermissionsMode」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.669 `src/utils/settings/changeDetector.test.ts`

该文件位于 `src/utils/settings`，约 299 行，覆盖主题「changeDetector」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.670 `src/utils/settings/flagSettings.test.ts`

该文件位于 `src/utils/settings`，约 96 行，覆盖主题「flagSettings」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.671 `src/utils/settings/modelPricing.test.ts`

该文件位于 `src/utils/settings`，约 185 行，覆盖主题「modelPricing」。不变量：自定义价格精确匹配；缺 webSearch 用默认；项目 settings 不得覆盖。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.672 `src/utils/settings/modelPricingSchema.test.ts`

该文件位于 `src/utils/settings`，约 140 行，覆盖主题「modelPricingSchema」。不变量：自定义价格精确匹配；缺 webSearch 用默认；项目 settings 不得覆盖。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.673 `src/utils/settings/permissionValidation.protoName.test.ts`

该文件位于 `src/utils/settings`，约 80 行，覆盖主题「permissionValidation.protoName」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.674 `src/utils/settings/providerProfileModelPickerMode.test.ts`

该文件位于 `src/utils/settings`，约 26 行，覆盖主题「providerProfileModelPickerMode」。不变量：缺省模型、错误映射、路由 kind 与描述符一致。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.675 `src/utils/settings/settings.transaction.test.ts`

该文件位于 `src/utils/settings`，约 1279 行，覆盖主题「settings.transaction」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.676 `src/utils/settings/settingsMergeCustomizer.test.ts`

该文件位于 `src/utils/settings`，约 119 行，覆盖主题「settingsMergeCustomizer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.677 `src/utils/settings/worktreeSettings.test.ts`

该文件位于 `src/utils/settings`，约 44 行，覆盖主题「worktreeSettings」。不变量：隔离目录存在；多仓库父路径计算正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.678 `src/utils/setupScreenGates.test.ts`

该文件位于 `src/utils`，约 69 行，覆盖主题「setupScreenGates」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.679 `src/utils/sideQuery.attribution.test.ts`

该文件位于 `src/utils`，约 372 行，覆盖主题「sideQuery.attribution」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.680 `src/utils/skills/skillChangeDetector.test.ts`

该文件位于 `src/utils/skills`，约 373 行，覆盖主题「skillChangeDetector」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.681 `src/utils/sshPreParse.test.ts`

该文件位于 `src/utils`，约 193 行，覆盖主题「sshPreParse」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.682 `src/utils/stableStringify.test.ts`

该文件位于 `src/utils`，约 241 行，覆盖主题「stableStringify」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.683 `src/utils/stats.totalDays.test.ts`

该文件位于 `src/utils`，约 337 行，覆盖主题「stats.totalDays」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.684 `src/utils/status.routes.test.ts`

该文件位于 `src/utils`，约 425 行，覆盖主题「status.routes」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.685 `src/utils/status.test.ts`

该文件位于 `src/utils`，约 380 行，覆盖主题「status」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.686 `src/utils/statusNoticeDefinitions.safety.test.tsx`

该文件位于 `src/utils`，约 267 行，覆盖主题「statusNoticeDefinitions.safetyx」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.687 `src/utils/statusRedaction.test.ts`

该文件位于 `src/utils`，约 189 行，覆盖主题「statusRedaction」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.688 `src/utils/streamingOptimizer.test.ts`

该文件位于 `src/utils`，约 61 行，覆盖主题「streamingOptimizer」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.689 `src/utils/suggestions/commandSuggestions.test.ts`

该文件位于 `src/utils/suggestions`，约 1082 行，覆盖主题「commandSuggestions」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.690 `src/utils/swarm/inProcessPermissionAbort.test.ts`

该文件位于 `src/utils/swarm`，约 138 行，覆盖主题「inProcessPermissionAbort」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.691 `src/utils/swarm/spawnInProcess.interruptionTrace.test.ts`

该文件位于 `src/utils/swarm`，约 88 行，覆盖主题「spawnInProcess.interruptionTrace」。不变量：中断原因分类正确；trace 开关有效；租约不被泄漏。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.692 `src/utils/swarm/spawnUtils.test.ts`

该文件位于 `src/utils/swarm`，约 104 行，覆盖主题「spawnUtils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.693 `src/utils/swarm/teammateModel.test.ts`

该文件位于 `src/utils/swarm`，约 57 行，覆盖主题「teammateModel」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.694 `src/utils/taskNotificationIdentity.test.ts`

该文件位于 `src/utils`，约 134 行，覆盖主题「taskNotificationIdentity」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.695 `src/utils/thinking.test.ts`

该文件位于 `src/utils`，约 151 行，覆盖主题「thinking」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.696 `src/utils/thinkingTokens.test.ts`

该文件位于 `src/utils`，约 69 行，覆盖主题「thinkingTokens」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.697 `src/utils/timeouts.test.ts`

该文件位于 `src/utils`，约 51 行，覆盖主题「timeouts」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.698 `src/utils/tokens.test.ts`

该文件位于 `src/utils`，约 248 行，覆盖主题「tokens」。不变量：估计与真实 usage 的误差在测试夹具范围内。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.699 `src/utils/toolErrors.test.ts`

该文件位于 `src/utils`，约 152 行，覆盖主题「toolErrors」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.700 `src/utils/toolResultStorage.preview.test.ts`

该文件位于 `src/utils`，约 491 行，覆盖主题「toolResultStorage.preview」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.701 `src/utils/toolResultStorage.test.ts`

该文件位于 `src/utils`，约 120 行，覆盖主题「toolResultStorage」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.702 `src/utils/toolSearch.test.ts`

该文件位于 `src/utils`，约 85 行，覆盖主题「toolSearch」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.703 `src/utils/truncate.test.ts`

该文件位于 `src/utils`，约 26 行，覆盖主题「truncate」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.704 `src/utils/updateStrategy.test.ts`

该文件位于 `src/utils`，约 204 行，覆盖主题「updateStrategy」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.705 `src/utils/urlRedaction.test.ts`

该文件位于 `src/utils`，约 276 行，覆盖主题「urlRedaction」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.706 `src/utils/user.test.ts`

该文件位于 `src/utils`，约 158 行，覆盖主题「user」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.707 `src/utils/visionUtils.test.ts`

该文件位于 `src/utils`，约 297 行，覆盖主题「visionUtils」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.708 `src/utils/warningHandler.test.ts`

该文件位于 `src/utils`，约 194 行，覆盖主题「warningHandler」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.709 `src/utils/worktree.agentBase.test.ts`

该文件位于 `src/utils`，约 107 行，覆盖主题「worktree.agentBase」。不变量：隔离目录存在；多仓库父路径计算正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.710 `src/utils/worktree.git.test.ts`

该文件位于 `src/utils`，约 182 行，覆盖主题「worktree.git」。不变量：隔离目录存在；多仓库父路径计算正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.711 `src/utils/worktree.multiRepoParent.test.ts`

该文件位于 `src/utils`，约 132 行，覆盖主题「worktree.multiRepoParent」。不变量：隔离目录存在；多仓库父路径计算正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.712 `src/utils/worktree.test.ts`

该文件位于 `src/utils`，约 133 行，覆盖主题「worktree」。不变量：隔离目录存在；多仓库父路径计算正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.713 `src/utils/xaiCredentials.test.ts`

该文件位于 `src/utils`，约 99 行，覆盖主题「xaiCredentials」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.714 `src/utils/xml.test.ts`

该文件位于 `src/utils`，约 33 行，覆盖主题「xml」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.715 `tests/build/scanner-filedir.test.ts`

该文件位于 `tests/build`，约 214 行，覆盖主题「scanner-filedir」。不变量：生成物稳定；scanner 文件目录分类正确。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.716 `tests/sdk/casing.test.ts`

该文件位于 `tests/sdk`，约 92 行，覆盖主题「casing」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.717 `tests/sdk/engine-mutators.test.ts`

该文件位于 `tests/sdk`，约 204 行，覆盖主题「engine-mutators」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.718 `tests/sdk/generated-types.test.ts`

该文件位于 `tests/sdk`，约 436 行，覆盖主题「generated-types」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.719 `tests/sdk/mcp-cleanup.test.ts`

该文件位于 `tests/sdk`，约 237 行，覆盖主题「mcp-cleanup」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.720 `tests/sdk/package-consumer-types.test.ts`

该文件位于 `tests/sdk`，约 327 行，覆盖主题「package-consumer-types」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.721 `tests/sdk/permissions.test.ts`

该文件位于 `tests/sdk`，约 1400 行，覆盖主题「permissions」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.722 `tests/sdk/query-concurrency.test.ts`

该文件位于 `tests/sdk`，约 333 行，覆盖主题「query-concurrency」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.723 `tests/sdk/query-happy-path.test.ts`

该文件位于 `tests/sdk`，约 395 行，覆盖主题「query-happy-path」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.724 `tests/sdk/query-lifecycle.test.ts`

该文件位于 `tests/sdk`，约 730 行，覆盖主题「query-lifecycle」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.725 `tests/sdk/query-methods.test.ts`

该文件位于 `tests/sdk`，约 244 行，覆盖主题「query-methods」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.726 `tests/sdk/sdk-context-isolation.test.ts`

该文件位于 `tests/sdk`，约 451 行，覆盖主题「sdk-context-isolation」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.727 `tests/sdk/sdk-factories.test.ts`

该文件位于 `tests/sdk`，约 100 行，覆盖主题「sdk-factories」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.728 `tests/sdk/sdk-mcp-sdk-tools.test.ts`

该文件位于 `tests/sdk`，约 126 行，覆盖主题「sdk-mcp-sdk-tools」。不变量：断开后无句柄泄漏；工具名编码稳定。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.729 `tests/sdk/sdk-preserved-segment.test.ts`

该文件位于 `tests/sdk`，约 506 行，覆盖主题「sdk-preserved-segment」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.730 `tests/sdk/sdk-v2-lifecycle.test.ts`

该文件位于 `tests/sdk`，约 751 行，覆盖主题「sdk-v2-lifecycle」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.731 `tests/sdk/session-functions.test.ts`

该文件位于 `tests/sdk`，约 385 行，覆盖主题「session-functions」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.732 `tests/sdk/shared-utils.test.ts`

该文件位于 `tests/sdk`，约 167 行，覆盖主题「shared-utils」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.733 `tests/sdk/stub-leak-detect.test.ts`

该文件位于 `tests/sdk`，约 90 行，覆盖主题「stub-leak-detect」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.734 `tests/sdk/tool-schema-cache.test.ts`

该文件位于 `tests/sdk`，约 97 行，覆盖主题「tool-schema-cache」。不变量：公开类型与运行时字段一致；多次 query 互不污染。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.735 `vscode-extension/openclaude-vscode/src/chat/permissionResponse.test.js`

该文件位于 `vscode-extension/openclaude-vscode/src/chat`，约 42 行，覆盖主题「permissionResponse」。不变量：未授权的工具不得 call；规则优先级 deny>ask>allow；模式切换可逆。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.736 `vscode-extension/openclaude-vscode/src/extension.test.js`

该文件位于 `vscode-extension/openclaude-vscode/src`，约 273 行，覆盖主题「extension」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.737 `vscode-extension/openclaude-vscode/src/presentation.test.js`

该文件位于 `vscode-extension/openclaude-vscode/src`，约 291 行，覆盖主题「presentation」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.738 `vscode-extension/openclaude-vscode/src/state.test.js`

该文件位于 `vscode-extension/openclaude-vscode/src`，约 246 行，覆盖主题「state」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。

### 17.739 `web/scripts/verify-dist.test.ts`

该文件位于 `web/scripts`，约 148 行，覆盖主题「verify-dist」。不变量：该模块的公开行为在重构后保持；失败信息稳定；无未关闭的异步句柄。 编写新用例时请保持：不访问真实外网、不依赖本机已登录账号、对时间与 UUID 使用可注入时钟或固定种子、对文件系统使用临时目录。若测试必须开 feature flag，应在文件内或 bun 参数中显式声明，避免默认路径误绿。


### 17.x 重点用例的不变量摘录

- **project onboarding：** 有 AGENTS.md 或 CLAUDE.md 即视为完成，包括祖先目录。
- **QueryEngine 自定义定价：** 零价与正价都必须进入 maxBudgetUsd 计算，禁止 NaN。
- **中断 trace：** 关闭 tracing 时不记生命周期；开启时 query-root 中断必须先于 abort。
- **abort 分类：** timeout/hard-max/background 不得生成用户打断文案。
- **cost format：** 无 token 时省略 Token usage 段；有 token 时显示 Input/Output 条；cache 行仅非零出现；大数字用 compact notation。
- **LSP 工具池：** 基础池包含 LSPTool，但未连接前对模型不可用。
- **repo map：** 默认 flag 关闭返回 null。
- **SDK：** 生命周期、并发、隔离、MCP cleanup、schema cache、大小写转换。
- **permissions：** 规则增删改、模式切换、headless plan hooks、tilde 展开、killswitch。
- **openaiShim：** 架构约束、Ollama 文本工具调用、压缩、诊断。
- **WebSearch providers：** 超时、各供应商适配、index 选择。
- **Bash：** sandbox 分析、错误输出、安全、只读、sed、路径。
- **Agent：** schema、routing、copilot scheduling、teammate model、persistence、resume。

补充测试策略：对运输层用 vcr/fixture；对 UI 用 Ink 测试渲染器；对权限用 TestingPermissionTool；对隐私用 `verify-no-phone-home`；对打包用 `verify-clean-install`。CI 的 pr-checks.yml 是权威矩阵，本地 `CONTRIBUTING.md` 的 Validation 节是推送前契约。


## 18. 构建、发布、诊断与迁移
`scripts/build.ts` 产出 `dist/cli.mjs` 与 `dist/sdk.mjs`。`bin/openclaude` 指向 CLI。`prepack` 在 npm pack 前构建。`scripts/system-check.ts` 是 doctor 运行时。`scripts/pr-intent-scan.ts` 扫描 PR 意图。`migrations/` 处理配置迁移，保证老用户目录升级到 `.openclaude`。
## 19. 风险、非目标与显式权衡
非目标：成为 IDE；成为通用聊天应用；在未讨论的情况下引入 Python 运行时；把站点做成手工发行说明库。权衡：为了兼容上百家模型，运输层复杂度集中在 shim，而不是让每个工具感知供应商。为了安全，默认多问一次用户，而不是默认全自动。为了 cache，工具列表顺序被当作 ABI。为了开源可构建，遥测可被 stub。
## 20. 设计说明书结语
OpenClaude 的设计可以概括为：一个对模型供应商中立的编码 Agent 内核，外加可描述的集成层、可审计的权限层、可压缩的上下文层、可嵌入的 SDK 层。任何功能改动都应回答：它落在哪一层？它如何被测试？它是否破坏 prompt cache、权限不变量或中断语义？若不能回答，就不应该合并。
后续章节的文件级说明见 `4SOURCE.md`；面向使用者的操作说明见 `2HELP.md`；面向 2026 年 Agent 趋势的演进方案见 `3SOLUTION.md`。


### 附录 A 启动时序细化

main.tsx 在 Commander parse 之后根据是否 --print、是否子命令、是否 --bg 分支。信任对话框仅交互模式显示。init() 加载插件缓存、命令表、工具表。launchRepl 创建 Ink Root，注入 AppStateStore。seedEarlyInput 允许在 UI 起来前把击键存住。settingsChangeDetector 与 skillChangeDetector 进入监听。GrowthBook 在鉴权变化后刷新。Ollama 模型列表预取失败必须静默，不能挡住 REPL。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 B 消息规范化

normalizeMessagesForAPI 删除 progress、部分 system subtype、重复记忆附件，合并连续 user，把 collapse summary 转成 user。getMessagesAfterCompactBoundary 只取最近一次 compact 之后。这对 resume 与 fork 至关重要：fork 复制边界后的消息到新 session id，不复制工作树。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 C 子 Agent 上下文

createSubagentContext 克隆 FileStateCache、可选克隆 contentReplacementState、冻结 renderedSystemPrompt、设置 agentId/agentType、把 setAppState 置空、保留 setAppStateForTasks。权限上下文可缩小工具集。异步 teammate 需要 localDenialTracking，否则拒绝计数无法累加。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 D 协调器模式

COORDINATOR_MODE 下主线程像调度器：SendMessage、TaskStop、Agent。工人拿到 Bash/Read/Edit 或完整池的过滤版。mailbox 与 UDS inbox 传递消息。关机必须有 interruption trace，避免工人悬挂。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 E 计划模式文件

计划通常写入特定 plan 文件路径，有 planFilePath 测试。模型在 plan 模式被拒绝写代码后，应把方案写入计划，用户 `/plan` 或 ExitPlanMode 再执行。这是人机协同的设计，而不是全自动。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 F 归因与提交

commit attribution 可配置 co-author。fileHistory 与 git 操作跟踪（gitOperationTracking）让 rewind 与 diff 按回合切片。/commit 与 /commit-push-pr 把 git 工作流产品化，但仍走 Bash 权限。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 G 远程与 Teleport

teleport API、environments、gitBundle 把会话迁到远程环境。--from-pr 按 PR 恢复。session QR 用于移动端续聊。远程命令白名单防止远端执行本地独有的危险命令。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 H 语音

voiceModeEnabled、voiceStreamSTT、voiceKeyterms。特性 VOICE_MODE。语音只是另一种输入，最终仍变成 UserMessage。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 I i18n

src/i18n 语言包。命令描述、帮助、部分 UI 字符串走 i18n。文档站目前以英文数据文件为源，本设计书以源码为准。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 J gRPC 与 server

dev:grpc 路径用于内部调试。src/server、src/grpc 不是默认用户路径。开源用户应把它视为可选实验面。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 K 原生 TS 与 PDF 技能

src/native-ts 与 pdf 技能提供无外部工具的 PDF 生成。这体现“能用 TypeScript 做的不要 shell 出去”的安全偏好。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 L 预算与 goal

goal 服务把完成条件变成可查询状态，SDK 有 GoalStatus 映射。maxBudgetUsd 在 QueryEngine 累计 usage*price 后停止。自定义零价用于本地模型。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 M 插件信任

插件可提供命令、技能、MCP、钩子。信任警告 UI 必须在执行第三方代码前展示。ValidatePlugin 做静态校验。市场缓存损坏时应可重建而不是崩溃。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 N Windows 特例

PowerShell 工具、路径规范化、Alt+V 粘贴、插件拷贝 ENOENT、别名脚本 scripts/windows/openclaude-aliases.ps1。任何新的文件系统 API 都必须有 win32 分支或使用已封装的 fsOperations。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

### 附录 O 特性旗标清单（设计影响）

PROACTIVE、KAIROS、COORDINATOR_MODE、VOICE_MODE、WORKFLOW_SCRIPTS、WEB_BROWSER_TOOL、HISTORY_SNIP、CONTEXT_COLLAPSE、UDS_INBOX、AGENT_TRIGGERS_REMOTE、MONITOR_TOOL、ULTRAPLAN、TORCH、BRIDGE_MODE、DAEMON、EXPERIMENTAL_SKILL_SEARCH、TRANSCRIPT_CLASSIFIER、REPO_MAP、TERMINAL_PANEL、OVERFLOW_TEST_TOOL。每个旗标关闭时，对应工具与命令应彻底消失，而不是以禁用态占 schema。从实现角度看，这一部分往往跨越多个目录，因此不能只看单一文件。评审相关 PR 时，应同时检查类型定义、运行时、UI、测试与文档站数据文件是否同步。若只改其中一层，必然出现“命令存在但帮助没有”“工具能调但权限模式不知道”“SDK 类型落后于运行时”这类漂移。OpenClaude 已经用 generated artifacts 与 web/src/data 的 seeded 注释对抗漂移，新增模块必须加入同一管道。性能上，这些子系统多数在启动关键路径之外，应继续保持延迟加载与 feature() 消除。安全上，任何新增的外部输入（URL、插件、MCP、远程会话）都必须进入权限模型，而不是只在 UI 上加一行警告。可靠性上，失败要可分类、可重试、可对用户解释，并且不要被记成错误的中断类型。兼容性上，别名、旧文件名 CLAUDE.md、CLAUDE_CONFIG_DIR、Task 工具旧名都必须继续工作一个完整的主版本周期。

## 21. 按目录划分的设计责任

### `src/utils`

共享工具集，近千文件，是最大的复杂度池。新增工具函数前必须搜索是否已有同类实现（path、fs、tokens、debug、abort）。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/services/api`

运输层心脏。改这里等于改所有供应商。必须跑 test:provider。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/services/compact`

压缩质量直接影响长会话能否继续。改阈值必须有 issue 引用与正负测试。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/services/mcp`

外部代码执行面。安全审查优先级最高之一。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/hooks`

React 钩子把命令式 CLI 状态变成声明式 UI。注意不要在渲染路径做重 IO。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/components`

Ink 组件。视觉改动要配截图，按贡献指南。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/bridge`

远程桥。涉及网络暴露，默认应关闭。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/buddy`

娱乐功能，不得出现在 --bare 或 SDK 默认路径。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/coordinator`

多 Agent 调度。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/memdir`

记忆目录布局与 freshness。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/query`

从 query.ts 拆出的辅助：config、deps、stopHooks、agentStepLimit、transitions。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/cli`

非 REPL 子命令实现。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/ink`

渲染器定制，升级 React/Ink 时的高风险区。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/integrations`

描述符时代的集成。改模型窗口或 effort 映射要读 reasoning-effort.md。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/entrypoints`

对外 ABI。破坏这里就是破坏 SDK 用户。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/state`

单一数据源。避免再引入平行 store。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/tasks`

后台任务多态。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/plugins`

插件加载。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/skills`

技能加载。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/keybindings`

键位。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/constants`

魔法数与产品名。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/context`

系统上下文拼装。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/screens`

全屏面板。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/vim`

Vim 引擎。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/voice`

语音开关。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/migrations`

配置迁移。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/schemas`

JSON schema。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/bootstrap`

会话 id 与持久化开关。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/remote`

远程会话。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/ssh`

SSH 预解析。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/upstreamproxy`

上游代理。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/assistant`

助手模式。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/proactive`

主动行为。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/jobs`

任务分类。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/outputStyles`

输出风格。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/environment-runner`

环境运行器。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/self-hosted-runner`

自托管。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。

### `src/daemon`

可选守护。 该目录的公共 API 应保持小而稳定；内部文件可以频繁重构，但跨目录 import 应通过 index 或明确模块。循环依赖已在多处用 lazy require 与“纯类型文件”化解，新增代码禁止再制造新的大环。测试应放在被测文件旁，便于移动模块时一起搬走。文档与 `web/src/data` 若暴露了该目录的用户面行为，必须同步。


## 22. 数据流时序图（文字形式）

T0 用户回车 → T1 processUserInput 解析附件与斜杠 → T2 若斜杠则本地执行并可能注入 XML 包装输出 → T3 否则入队 query → T4 QueryGuard 开租约 → T5 组装 system prompt 与 tool schemas（可能 defer）→ T6 运输层流式返回 → T7 若 tool_use 则权限流水线 → T8 runTools → T9 结果预算 → T10 继续或终止 → T11 stop hooks → T12 记账与 transcript flush → T13 UI 解除 spinner。

任何在 T7 之前执行副作用的工具都是漏洞。任何在 T12 之前丢失的中断原因都会污染分析。任何在 T5 改变工具顺序的改动都会打碎 prompt cache。

## 23. 质量属性场景

- **安全性：** 不可信仓库打开时，默认询问；策略文件可强制 deny；YOLO 需额外允许。
- **可靠性：** 运输失败重试；压缩失败冷却；MCP 失败隔离。
- **性能：** 启动并行预取；工具并行；schema 缓存；结果落盘降内存。
- **可观测性：** debug 分类过滤、doctor、cache-stats、api_metrics、interruption trace。
- **可移植性：** Node 22+；Windows PowerShell 路径；可选 AUR 包装。
- **可扩展性：** 描述符加供应商；插件加命令；MCP 加工具；Agent 目录加角色。

## 24. 对实现者的检查清单

1. 是否更新了 Zod schema 与 prompt.ts？  
2. 是否声明只读/并发/破坏性？  
3. 是否加入权限测试？  
4. 是否避免在渲染路径做 IO？  
5. 是否保持工具注册顺序稳定？  
6. 是否在 web/src/data 更新用户文档？  
7. 是否跑了 CONTRIBUTING 的 validation？  
8. 是否处理了 Windows 路径？  
9. 是否把错误分类进现有错误表？  
10. 是否避免引入新的运行时依赖？

本设计说明书到此完成对架构、模型、功能、函数、算法与测试的源代码级描述。细节文件清单以 `4SOURCE.md` 为准，随仓库演进应以脚本重新生成附录中的测试文件列表。
