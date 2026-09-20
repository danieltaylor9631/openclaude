# OpenClaude 使用说明书

**产品：** OpenClaude（npm 包 `@gitlawb/openclaude`）  
**版本基准：** 0.30.0  
**适用读者：** 准备在本地或 CI 中使用编码 Agent 的开发者、团队管理员、以及把 CLI 嵌进脚本的集成方。  
**配套文档：** 设计见 `1DESIGN.md`，演进方案见 `3SOLUTION.md`，源码地图见 `4SOURCE.md`。

本说明书覆盖：整体功能、安装与启动、每一个用户可感知功能的操作步骤、配置项、权限与安全实践、以及 100 个常见问题。所有命令与标志以当前源码和文档站数据文件为准。

---

## 1. 整体功能概述

OpenClaude 是一个运行在终端里的编码 Agent。你用自然语言描述要做的事，它会读取仓库、搜索代码、编辑文件、运行测试、查询网页、调用 MCP 工具，并在需要时征求你的许可。它不绑定单一模型厂商：你可以通过 `/provider` 选择 Anthropic、OpenAI 兼容网关、Gemini、GitHub Models、Codex OAuth、Ollama、LM Studio、Bedrock、Vertex 以及文档中列出的其它后端。

一次典型工作流是：在仓库根目录执行 `openclaude` → 用 `/provider` 配好模型 → 用普通句子提出任务 → 在权限对话框里允许或拒绝 Bash/Edit/网络 → 用 `/diff` 查看改动 → 用 `/commit` 或自己提交。长任务可用 `/plan` 先规划，用 `/tasks` 看后台，用 `--worktree` 隔离实验。

产品同时提供三条使用面：

1. **交互 REPL：** 全功能终端 UI，斜杠命令、主题、Vim、buddy、状态行。

2. **无头 `--print`：** 适合脚本；可输出 text、json、stream-json；可强制 JSON Schema。

3. **SDK：** `import { query } from '@gitlawb/openclaude/sdk'`，把同一引擎嵌进你的程序。

此外还有 VS Code 扩展（启动集成与主题）、后台会话（`openclaude --bg` + `ps/logs/kill`）、技能与插件、MCP、LSP、Chrome 集成（Beta）、以及可选的语音/桥接等特性开关功能。

OpenClaude 的设计目标是“一套工作流走遍所有模型”，因此你在切换 provider 时，工具名、权限规则、斜杠命令、会话恢复方式保持不变。会变的只是模型质量、上下文窗口、价格、以及是否支持思考/工具调用。

请记住几条产品级约束，否则容易误判为故障：它**不会**自动加载项目 `.env`；后台会话**不是**网络守护进程；`--fork-session` **只分叉对话不分叉文件**；`--print` **会跳过工作区信任对话框**，只应在你信任的目录使用。

## 2. 安装与环境要求

### 2.1 运行时

安装与运行需要 Node.js `>=22.0.0`。从源码构建、跑测试、跑仓库脚本需要 Bun。两者不要混淆：你给终端用户的指令是 npm 全局安装；你给贡献者的指令才是 bun。

### 2.2 全局安装

```bash
npm install -g @gitlawb/openclaude@latest
openclaude --version
```

若提示 `ripgrep not found`，请先在系统安装 ripgrep，并在同一终端确认 `rg --version` 可用。Grep 工具依赖它（或构建嵌入的搜索二进制）。

Arch Linux 可使用社区 AUR 包 `openclaude`（例如 `paru -S openclaude`）。

### 2.3 校验安装

```bash
openclaude --version
npm view @gitlawb/openclaude dist-tags
openclaude doctor
```

`/doctor` 或 `openclaude doctor` 会检查 Node 版本、配置目录、常见缺失依赖。源码开发者还可 `bun run doctor:runtime`。

### 2.4 从源码运行

```bash
git clone https://github.com/Gitlawb/openclaude.git
cd openclaude
bun install
bun run build
node bin/openclaude
```

开发脚本包括 `bun run dev`、`bun run dev:openai`、`bun run dev:gemini`、`bun run dev:ollama`、`bun run profile:init` 等，见 package.json。

### 2.5 VS Code 扩展

仓库内 `vscode-extension/openclaude-vscode/`。安装扩展后，可从编辑器拉起 OpenClaude 并查看 diff。扩展不替代 CLI，只是宿主。

### 2.6 升级

REPL 内 `/update`，或重新 `npm install -g @gitlawb/openclaude@latest`。注意区分 latest 与 stable dist-tag。

## 3. 第一次启动

在你要工作的 git 仓库目录执行 `openclaude`。首次会询问是否信任该工作区。只对你拥有或已审查的代码点信任。

进入 REPL 后，优先做两件事：

1. `/provider`：选择供应商、填写密钥或完成 OAuth、保存 profile。

2. `/status`：确认模型、鉴权、工具状态都是绿的。

然后直接用自然语言，例如：“解释 src/query.ts 的自动压缩路径，并补一个失败冷却的测试”。Agent 会 Read/Grep/Edit。当它要跑 `bun test` 时，你会看到权限提示，选择允许一次、允许会话、或拒绝。

推荐的安全起步：权限模式保持 default；不要对含生产密钥的目录开 YOLO；需要批量改文件再用 acceptEdits；需要先摸清代码再用 `/plan`。

输入技巧：Shift+Enter 换行（若无效则 `/terminal-setup`）；Ctrl+V 粘贴截图；Ctrl+G 用 $EDITOR 编辑长提示；Ctrl+R 搜历史；Shift+Tab 循环权限模式；Esc 关闭对话框；Ctrl+C 中断当前回合；Ctrl+D 退出。

会话默认会保存。下次在同一目录 `openclaude --continue` 继续最近一次；`openclaude --resume` 打开选择器。加 `--fork-session` 则复制一份对话历史到新 ID，原会话不动。

## 4. 功能详解（按能力域）

### 4.1 交互式编程助手

在 REPL 中用自然语言驱动 Read/Edit/Bash/Grep/Glob。适合修 bug、写测试、重构、阅读陌生代码。注意给足仓库上下文，或先 `/init` 生成项目指令。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.2 多模型提供商

`/provider` 保存多套 profile，可热切换。环境变量作为补充，不作为项目内默认。Opengateway 等网关可智能路由。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.3 权限与计划模式

Shift+Tab 或 `/permissions`。plan 模式只读探索；acceptEdits 自动改文件仍问命令；bypass 仅沙箱。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.4 会话恢复与分叉

--resume / --continue / --fork-session / /rename / /tag / /export / /replay。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.5 上下文压缩

接近窗口时自动 compact；也可 `/compact`。`/context`、`/request-size`、`/cache-stats` 观察占用。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.6 记忆与知识

`/memory`、`/dream`、`/knowledge`、`/wiki`、CLAUDE.md / AGENTS.md。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.7 子 Agent 与团队

`/agents` 配置角色；模型可调用 Agent 工具做探索/审查；swarm 特性下有 Team 与 SendMessage。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.8 技能

`/skills` 列出；`/batch` `/simplify` `/debug` `/pdf` `/update-config` 等 bundled 技能。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.9 插件与市场

`/plugin` 安装、启用、校验、管理 marketplace。`/reload-plugins` 热加载。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.10 MCP

`/mcp` 与 `openclaude mcp add`。把浏览器、数据库、内部 API 变成工具。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.11 LSP

`/lsp` 连接语言服务器，让 Agent 使用诊断与跳转。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.12 IDE 与 Chrome

`/ide`、`--ide`、`/chrome`。把终端 Agent 接到编辑器选区或浏览器。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.13 Git 与评审

`/diff` `/commit` `/review` `/security-review` `/pr-comments` `/bughunter*` `/install-github-app`。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.14 后台与工作树

`--bg`、`--worktree`、`--tmux`、`/tasks`。长任务不阻塞当前终端，实验不脏主工作区。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.15 无头与 CI

`--print`、`--output-format`、`--json-schema`、`--heartbeat`、`--bare`。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.16 成本与用量

`/cost` `/usage` `/extra-usage`，自定义 modelPricing。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.17 外观

`/theme` `/logo` `/color` `/statusline` `/vim` `/keybindings` `/buddy`。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.18 诊断

`/doctor` `/status` `/diagnostics` `/stats` `/insights` `/feedback`。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.19 SDK 嵌入

同一 QueryEngine，权限回调，流式消息。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。

### 4.20 后台进程管理

`openclaude ps` 查看，`logs -f` 跟随，`kill` 结束。

使用建议：先用最小权限验证这条能力是否在你的 provider/模型上可用（例如某些本地小模型不能稳定产出 tool_use），再把它写进团队工作流。若失败，先 `/status` 与 `/doctor`，再看 `/cost` 是否已经触达预算，最后才怀疑是产品 bug。把可重复的步骤沉淀为 Skill、插件或 `--print` 脚本，而不是每次手打长提示。
## 5. 每一个斜杠命令的使用说明

斜杠命令在 REPL 输入框以 `/` 开头。可用 Tab 补全。隐藏的、特性未开的、仅内部用户的命令不会出现在补全里。下面按命令名字母顺序列出使用方法、参数、注意事项。未单独展开的命令同样存在于 `src/commands/`，行为以 `/help` 现场文案为准。

### 5.1 `/add-dir`

`/add-dir ../shared-lib` 让工具能碰额外目录。 调用方式：在 REPL 输入 `/add-dir` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.2 `/ads`

 调用方式：在 REPL 输入 `/ads` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.3 `/advisor`

 调用方式：在 REPL 输入 `/advisor` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.4 `/agents`

创建/编辑自定义 Agent。向导会问位置、工具、模型、颜色。 调用方式：在 REPL 输入 `/agents` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.5 `/auto-fix`

 调用方式：在 REPL 输入 `/auto-fix` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.6 `/autofix-pr`

 调用方式：在 REPL 输入 `/autofix-pr` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.7 `/backfill-sessions`

 调用方式：在 REPL 输入 `/backfill-sessions` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.8 `/branch`

 调用方式：在 REPL 输入 `/branch` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.9 `/break-cache`

 调用方式：在 REPL 输入 `/break-cache` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.10 `/bridge`

 调用方式：在 REPL 输入 `/bridge` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.11 `/brief`

 调用方式：在 REPL 输入 `/brief` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.12 `/btw`

 调用方式：在 REPL 输入 `/btw` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.13 `/buddy`

`/buddy status|mute|unmute|name 名字|set 形态`。 调用方式：在 REPL 输入 `/buddy` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.14 `/bughunter`

系统化找 bug，时间与 token 消耗都更大，适合发布前。 调用方式：在 REPL 输入 `/bughunter` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.15 `/bughunter-perf`

 调用方式：在 REPL 输入 `/bughunter-perf` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.16 `/bughunter-security`

 调用方式：在 REPL 输入 `/bughunter-security` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.17 `/cache-probe`

 调用方式：在 REPL 输入 `/cache-probe` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.18 `/cache-stats`

 调用方式：在 REPL 输入 `/cache-stats` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.19 `/chrome`

 调用方式：在 REPL 输入 `/chrome` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.20 `/clear`

新开对话，不删磁盘上的旧 session。 调用方式：在 REPL 输入 `/clear` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.21 `/clear-context-window`

 调用方式：在 REPL 输入 `/clear-context-window` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.22 `/color`

 调用方式：在 REPL 输入 `/color` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.23 `/commit`

让 Agent 写提交信息并提交。先 `/diff`。 调用方式：在 REPL 输入 `/commit` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.24 `/commit-message`

 调用方式：在 REPL 输入 `/commit-message` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.25 `/commit-push-pr`

 调用方式：在 REPL 输入 `/commit-push-pr` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.26 `/compact`

立刻压缩。可附“保留关于数据库迁移的细节”。 调用方式：在 REPL 输入 `/compact` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.27 `/config`

 调用方式：在 REPL 输入 `/config` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.28 `/context`

 调用方式：在 REPL 输入 `/context` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.29 `/copy`

 调用方式：在 REPL 输入 `/copy` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.30 `/cost`

看钱。本地模型应接近零（若配置了零价）。 调用方式：在 REPL 输入 `/cost` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.31 `/ctx`

 调用方式：在 REPL 输入 `/ctx` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.32 `/desktop`

 调用方式：在 REPL 输入 `/desktop` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.33 `/diagnostics`

 调用方式：在 REPL 输入 `/diagnostics` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.34 `/diff`

看 Agent 这一段改了什么。 调用方式：在 REPL 输入 `/diff` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.35 `/doctor`

更深入的环境诊断。 调用方式：在 REPL 输入 `/doctor` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.36 `/dream`

触发记忆巩固，可能花一些时间和 token。 调用方式：在 REPL 输入 `/dream` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.37 `/effort`

`/effort high` 等。不是所有模型都支持；不支持时会被忽略或提示。 调用方式：在 REPL 输入 `/effort` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.38 `/exit`

离开 REPL。Ctrl+D 同等。 调用方式：在 REPL 输入 `/exit` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.39 `/export`

`/export notes.md` 或无参走剪贴板。 调用方式：在 REPL 输入 `/export` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.40 `/extra-usage`

 调用方式：在 REPL 输入 `/extra-usage` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.41 `/fast`

 调用方式：在 REPL 输入 `/fast` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.42 `/feedback`

 调用方式：在 REPL 输入 `/feedback` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.43 `/files`

 调用方式：在 REPL 输入 `/files` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.44 `/goal`

 调用方式：在 REPL 输入 `/goal` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.45 `/help`

显示帮助。直接 `/help` 或 `/help provider` 看某一命令。 调用方式：在 REPL 输入 `/help` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.46 `/hooks`

 调用方式：在 REPL 输入 `/hooks` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.47 `/ide`

 调用方式：在 REPL 输入 `/ide` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.48 `/init`

扫描仓库写项目指令。提交前请人工审阅，避免泄露内部 URL。 调用方式：在 REPL 输入 `/init` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.49 `/insights`

 调用方式：在 REPL 输入 `/insights` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.50 `/install-github-app`

 调用方式：在 REPL 输入 `/install-github-app` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.51 `/install-slack-app`

 调用方式：在 REPL 输入 `/install-slack-app` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.52 `/issue`

 调用方式：在 REPL 输入 `/issue` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.53 `/keybindings`

 调用方式：在 REPL 输入 `/keybindings` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.54 `/knowledge`

 调用方式：在 REPL 输入 `/knowledge` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.55 `/login`

登录 Anthropic。浏览器回调。SSH 环境看命令提示是否支持粘贴回调。 调用方式：在 REPL 输入 `/login` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.56 `/logo`

 调用方式：在 REPL 输入 `/logo` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.57 `/logout`

清除登录态。 调用方式：在 REPL 输入 `/logout` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.58 `/lsp`

 调用方式：在 REPL 输入 `/lsp` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.59 `/mcp`

`/mcp` 列出；`/mcp disable servername` 临时关掉。 调用方式：在 REPL 输入 `/mcp` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.60 `/memory`

打开记忆文件编辑。不要写入密钥。 调用方式：在 REPL 输入 `/memory` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.61 `/mobile`

 调用方式：在 REPL 输入 `/mobile` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.62 `/model`

`/model sonnet` 或完整 id。不清楚就先 `/model` 打开选择器。 调用方式：在 REPL 输入 `/model` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.63 `/onboard-github`

GitHub Copilot/Models 设备码登录。 调用方式：在 REPL 输入 `/onboard-github` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.64 `/output-style`

 调用方式：在 REPL 输入 `/output-style` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.65 `/passes`

 调用方式：在 REPL 输入 `/passes` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.66 `/peers`

 调用方式：在 REPL 输入 `/peers` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.67 `/permissions`

图形化编辑允许/拒绝规则。也可直接改 settings.json。 调用方式：在 REPL 输入 `/permissions` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.68 `/plan`

`/plan` 进入计划；`/plan 实现登录` 带初始描述。 调用方式：在 REPL 输入 `/plan` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.69 `/plugin`

进入插件 UI。安装前读信任警告。 调用方式：在 REPL 输入 `/plugin` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.70 `/pr-comments`

 调用方式：在 REPL 输入 `/pr-comments` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.71 `/privacy-settings`

 调用方式：在 REPL 输入 `/privacy-settings` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.72 `/provider`

打开提供商向导。选择 Anthropic、Codex OAuth、OpenAI 兼容、Ollama 等，保存到 profile。这是配置密钥的首选方式。 调用方式：在 REPL 输入 `/provider` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.73 `/rate-limit-options`

 调用方式：在 REPL 输入 `/rate-limit-options` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.74 `/release-notes`

 调用方式：在 REPL 输入 `/release-notes` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.75 `/reload-plugins`

 调用方式：在 REPL 输入 `/reload-plugins` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.76 `/remote-env`

 调用方式：在 REPL 输入 `/remote-env` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.77 `/remote-setup`

 调用方式：在 REPL 输入 `/remote-setup` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.78 `/rename`

 调用方式：在 REPL 输入 `/rename` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.79 `/replay`

 调用方式：在 REPL 输入 `/replay` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.80 `/repomap`

 调用方式：在 REPL 输入 `/repomap` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.81 `/request-size`

 调用方式：在 REPL 输入 `/request-size` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.82 `/resume`

交互选择历史会话。 调用方式：在 REPL 输入 `/resume` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.83 `/review`

对 PR 做审查。需要 gh 认证时按提示。 调用方式：在 REPL 输入 `/review` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.84 `/rewind`

回滚代码与/或对话。操作前确认 file history 已启用。 调用方式：在 REPL 输入 `/rewind` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.85 `/sandbox-toggle`

 调用方式：在 REPL 输入 `/sandbox-toggle` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.86 `/security-review`

针对当前分支变更的安全审查，不是全仓审计。 调用方式：在 REPL 输入 `/security-review` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.87 `/session`

 调用方式：在 REPL 输入 `/session` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.88 `/set-context-window`

 调用方式：在 REPL 输入 `/set-context-window` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.89 `/share`

 调用方式：在 REPL 输入 `/share` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.90 `/skills`

列出技能及来源。 调用方式：在 REPL 输入 `/skills` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.91 `/smartroute`

`/smartroute on` 后配置 simple/strong 模型，省钱。 调用方式：在 REPL 输入 `/smartroute` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.92 `/stats`

 调用方式：在 REPL 输入 `/stats` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.93 `/status`

健康检查总览。模型连不上时从这里看。 调用方式：在 REPL 输入 `/status` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.94 `/statusline`

 调用方式：在 REPL 输入 `/statusline` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.95 `/stickers`

 调用方式：在 REPL 输入 `/stickers` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.96 `/tag`

 调用方式：在 REPL 输入 `/tag` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.97 `/tasks`

 调用方式：在 REPL 输入 `/tasks` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.98 `/teleport`

 调用方式：在 REPL 输入 `/teleport` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.99 `/terminal-setup`

 调用方式：在 REPL 输入 `/terminal-setup` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.100 `/theme`

换终端主题。 调用方式：在 REPL 输入 `/theme` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.101 `/thinkback`

 调用方式：在 REPL 输入 `/thinkback` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.102 `/ultraplan`

 调用方式：在 REPL 输入 `/ultraplan` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.103 `/update`

`/update latest` 或指定版本。 调用方式：在 REPL 输入 `/update` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.104 `/upgrade`

 调用方式：在 REPL 输入 `/upgrade` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.105 `/usage`

看订阅用量。 调用方式：在 REPL 输入 `/usage` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.106 `/version`

 调用方式：在 REPL 输入 `/version` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.107 `/vim`

进入 Vim 键位。 调用方式：在 REPL 输入 `/vim` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.108 `/voice`

 调用方式：在 REPL 输入 `/voice` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.109 `/wiki`

 调用方式：在 REPL 输入 `/wiki` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。

### 5.110 `/workflows`

 调用方式：在 REPL 输入 `/workflows` 及可选参数后回车。若该命令打开面板，使用方向键或 Ctrl+P/N 移动，Enter 确认，Esc 取消。若该命令立即执行，输出会出现在 transcript，并可能以系统消息形式进入后续模型上下文。远程/桥接会话中，不在安全名单里的命令会被隐藏。若命令无响应，先 Ctrl+C 中断，再 `/status`。把常用命令写进团队的 CLAUDE.md，例如“提交前必须 /diff 与 /security-review”。


## 5.x 与命令相关的技能调用

部分技能看起来像命令：`/batch`、`/simplify`、`/debug`、`/pdf`、`/update-config`。它们走技能加载器，可能消耗一次模型回合来执行说明书里的流程。`/batch` 适合大规模机械改动（5–30 个隔离 worktree Agent 开 PR）；`/simplify` 审查重复与质量；`/pdf` 生成 PDF，无需系统安装 TeX。

## 6. CLI 标志与子命令

### 6.1 核心

`openclaude` 进入 REPL。`-p/--print` 打印一次结果后退出。`--bare` 最小模式，适合你自己提供全部上下文。`-d/--debug [filter]` 开调试，例如 `api,hooks` 或 `!file`。`--debug-file path` 写日志。`--verbose` 覆盖配置。

### 6.2 无头 IO

`--output-format text|json|stream-json`。`--input-format text|stream-json`。`--json-schema` 校验最终结构。`--include-hook-events`、`--include-partial-messages`、`--replay-user-messages` 用于把 Agent 嵌进编排器。`--heartbeat 30s` 在静默时保活。

### 6.3 模型

`--model` `--provider` `--effort` `--fallback-model` `--agent` `--betas`。

### 6.4 会话

`-c/--continue` `-r/--resume [id]` `--fork-session` `--from-pr` `--session-id` `-n/--name` `--no-session-persistence` `-w/--worktree` `--tmux`。

### 6.5 权限

`--permission-mode` `--allowed-tools` `--disallowed-tools` `--tools` `--yolo` `--allow-dangerously-skip-permissions` `--add-dir`。

### 6.6 提示词

`--system-prompt` 替换；`--append-system-prompt` 追加。

### 6.7 MCP 与配置

`--mcp-config` `--strict-mcp-config` `--settings` `--setting-sources user,project,local` `--agents json` `--plugin-dir` `--ide` `--chrome` `--no-chrome` `--disable-slash-commands` `--provider-env-file` `--file file_id:path`。

### 6.8 子命令

`openclaude mcp [add|remove|list|doctor]`；`openclaude auth login|status|logout`；`openclaude auth xai login|device|logout|status`；`openclaude skills ...`；`openclaude plugin ...`；后台 `ps` `logs` `kill`。`--print` 时跳过子命令。

### 6.9 后台示例

```bash
openclaude --bg "fix failing tests"
openclaude --bg --name auth-refactor "refactor auth middleware"
openclaude ps
openclaude logs auth-refactor -f
openclaude kill auth-refactor
```

权限/模型/设置标志会原样传给子进程，和前台 `--print` 一致。

## 7. 配置说明书
### 7.1 配置文件位置
- 用户：`~/.openclaude/settings.json`（`OPENCLAUDE_CONFIG_DIR` 可改根，遗留 `CLAUDE_CONFIG_DIR` 仅在前者未设时生效）
- 项目：`.openclaude/settings.json`（可提交）
- 本机项目：`.openclaude/settings.local.json`（应 gitignore）
- 全局杂项：`~/.openclaude.json` 含 theme、verbose、autoUpdates 等
- 键位：`~/.openclaude/keybindings.json`
- 项目指令：`CLAUDE.md`、`.claude/CLAUDE.md`、`AGENTS.md`
- 提供商档案：`.openclaude-profile.json`（由 `/provider` 维护）
### 7.2 常用 settings 键
model、effortLevel、agent、permissions、env、hooks、smartRouting、modelLimits、modelPricing、providerFallbackChain、agentModels、agentRouting、advisorModel、subscriptionType（仅 user 级）。
### 7.3 权限规则写法
在 settings 的 permissions.allow / deny 中写 `Bash(git *)`、`Edit`、`mcp__server` 这样的规则。空内容表示整把工具。优先在 local 文件写宽松规则，把严格策略放 user 或托管 policy。
### 7.4 环境变量
- `ANTHROPIC_API_KEY`：Anthropic 密钥；--bare 下的严格鉴权路径。
- `ANTHROPIC_AUTH_TOKEN`：Bearer 替代。
- `OPENAI_API_KEY / OPENAI_API_KEYS`：兼容端点密钥；KEYS 为逗号池，失败轮换。
- `OPENAI_BASE_URL / OPENAI_MODEL`：兼容端点。
- `OPENGATEWAY_API_KEY / OPENGATEWAY_BASE_URL`：Gitlawb 网关。
- `GEMINI_API_KEY`：Gemini，注意不是 GOOGLE_API_KEY。
- `XAI_API_KEY`：Grok；或 OAuth。
- `GITHUB_TOKEN`：GitHub Models 与 PR 工作流。
- `DASHSCOPE_API_KEY / MIMO_API_KEY / AIMLAPI_API_KEY / APISMART_API_KEY / NVIDIA_API_KEY / CLOUDFLARE_API_TOKEN / OPENCODE_API_KEY / NEARAI_API_KEY / KIMI_API_KEY`：各厂商/网关。
- `OPENCLAUDE_CONFIG_DIR`：配置根。
- `BASH_MAX_OUTPUT_LENGTH`：命令输出截断。
- `HTTP_PROXY / HTTPS_PROXY / NODE_EXTRA_CA_CERTS`：企业网。
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`：减少非必要网络。
- `OPENCLAUDE_SMART_ROUTING*`：智能路由默认。
- `CLAUDE_CODE_OPENAI_CONTEXT_WINDOWS / CLAUDE_CODE_OPENAI_MAX_OUTPUT_TOKENS`：JSON 映射覆盖窗口。
- `CLAUDE_CODE_SIMPLE`：只启用 Bash/Read/Edit。
- `CLAUDE_CODE_VERIFY_PLAN`：计划执行校验工具。
### 7.5 权限模式使用场景
- `default`（默认模式）：每次敏感工具调用都询问用户，适合日常交互与不可信仓库。
- `acceptEdits`（接受编辑）：自动批准文件编辑类工具，仍询问 Bash、网络与破坏性操作。
- `plan`（计划模式）：只允许只读探索与计划文件写入，禁止直接改代码，直到退出计划。
- `bypassPermissions`（绕过权限）：跳过交互式询问，仅用于隔离沙箱或用户明确授权的无人值守场景。
- `dontAsk`（不问）：尽量不弹窗，结合规则与分类器决策。
- `fullAccess`（完全访问）：宽权限模式，仍受策略文件与危险模式熔断约束。
- `auto`（自动分类）：在 TRANSCRIPT_CLASSIFIER 特性开启时，用分类器决定是否放行。
- `bubble`（气泡模式）：内部权限模式，用于特定宿主 UI。

### 7.6 推荐的团队配置策略

把主题、自动更新放用户全局；把 hooks、deny 规则、默认 modelLimits 放项目共享；把 API 密钥只放本机环境或 `/provider` profile，永不进 git。对开源贡献者仓库使用更严的 deny（禁止 curl | sh）。对内部可信单体仓库可以 acceptEdits。CI 使用 --print 加明确 --allowed-tools，而不是 YOLO。

### 7.7 Hook 配置入门

在 settings.hooks 中为 PreToolUse / PostToolUse / Stop 配置要执行的命令。可用于强制 lint、阻止访问生产集群、在每次 Edit 后跑单测。无头可加 --include-hook-events 观察。错误的 hook 会阻断回合，先用 `/hooks` 查看。

### 7.8 模型窗口与定价

遇到“不明模型上下文只有 8k”时，用 modelLimits 或 CLAUDE_CODE_OPENAI_CONTEXT_WINDOWS。要让 `/cost` 对自建模型准确，用 user 级 modelPricing。不要把定价写进可被 fork 的项目 settings。

## 8. 提供商使用要点

### `anthropic` Anthropic Claude

订阅/API Key。官方 Messages API、OAuth 登录与 API Key 直连 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `codex-oauth` Codex OAuth / ChatGPT

订阅。浏览器登录 ChatGPT，走 Responses API，覆盖 GPT-5.6 家族 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `xai-oauth` xAI Grok

OAuth/API Key。浏览器 OAuth 或设备码，适配远程主机 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `github-models` GitHub Models / Copilot

Token/OAuth。/onboard-github 引导，支持 Copilot Enterprise 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `kimi-code` Moonshot Kimi Code

订阅。Kimi K3 1M/256K 上下文变体 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `zai` Z.AI GLM Coding Plan

订阅。默认 glm-5.2 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `dashscope` 阿里云百炼 Coding Plan

API Key。国际站与中国站，默认 qwen3.6-plus 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `xiaomi-mimo-token` Xiaomi MiMo Token Plan

订阅。默认 mimo-v2.5-pro 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `opengateway` Gitlawb Opengateway

网关。智能路由，聚合 MiMo/MiniMax/Qwen/GLM/Gemini 与免费模型 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `aimlapi` AI/ML API

网关。一千以上模型，CLI 内充值与发钥 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `openrouter` OpenRouter

网关。OpenAI 兼容聚合 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `llmtr` LLMTR

网关。默认 deepseek-v4-flash 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `novita` Novita AI

网关。合作网关 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `atlas-cloud` Atlas Cloud

网关。合作网关 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `apismart` ApiSmart

网关。默认 DEEPSEEK_V4_FLASH 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `concentrate` Concentrate

网关。合作网关 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `openai` OpenAI

厂商 API。官方 OpenAI 与兼容端点 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `gemini` Google Gemini

厂商 API。GEMINI_API_KEY，含 Vertex 路径 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `deepseek` DeepSeek

厂商 API。DeepSeek Chat/Reasoner 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `minimax` MiniMax

厂商 API。含 usage 解析 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `xai` xAI 直连

厂商 API。XAI_API_KEY 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `mistral` Mistral

厂商 API。Mistral 与相关网关 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `fireworks` Fireworks

厂商 API。托管开源模型 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `together` Together

网关。托管推理 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `groq` Groq

网关。低延迟推理 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `nvidia-nim` NVIDIA NIM

网关。NVIDIA_API_KEY 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `cloudflare` Cloudflare Workers AI

网关。CLOUDFLARE_API_TOKEN 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `bedrock` Amazon Bedrock

云路由。STS/凭证链 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `vertex` Google Vertex AI

云路由。GCP 凭证 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `azure-openai` Azure OpenAI

云路由。Azure Identity 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `ollama` Ollama

本地。本机或局域网 OpenAI 兼容 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `lmstudio` LM Studio

本地。本机兼容端点 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `atomic-chat` Atomic Chat

网关。合作伙伴路由 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。

### `custom` 自定义端点

自建。任意 OpenAI 兼容 /v1 配置：优先 `/provider` 选择该项并按向导操作。若改用环境变量，确保没有互相冲突的 OPENAI_BASE_URL。验证：`/status` 能列出模型；随便问一题并确认出现工具调用。失败时检查密钥是否导出到当前 shell（OpenClaude 不读 .env），代理证书，以及该模型是否支持 tools。
## 9. 常见问题 100 问（含补充）

以下共 176 条。先搜索关键词再提问。

### Q1. 安装后命令找不到

确认 npm 全局 bin 在 PATH。npx @gitlawb/openclaude 可临时跑。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q2. 提示需要 Node 22

用 nvm/fnm 安装 22+。Bun 不能替代用户运行时。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q3. ripgrep not found

系统包管理器安装 rg，重开终端。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q4. 它不读我的 .env

设计如此。用 /provider 或 --provider-env-file 或在 shell 里 export。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q5. API key 无效

检查是否贴了换行/引号；是否用错变量名（Gemini 要用 GEMINI_API_KEY）。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q6. OAuth 回调失败

SSH 无 localhost 时用设备码（xAI device、GitHub onboard）。xAI 会校验 state。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q7. 模型不调工具

换支持 tools 的模型；检查 --tools 是否把工具禁了；看是否 SIMPLE 模式。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q8. 一直在问权限

Shift+Tab 到 acceptEdits，或写 allow 规则，或 /permissions。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q9. YOLO 很危险吗

是。等于允许任意命令。只在无外网沙箱用。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q10. plan 模式不能改文件

这是特性。退出计划后再执行。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q11. 上下文满了

/compact 或 /clear；减小 CLAUDE.md；用 ToolSearch；提高窗口 modelLimits。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q12. 自动压缩循环

过小的窗口会导致阈值问题；升级到当前版本（有 floor buffer 修复）。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q13. resume 找不到会话

必须在同一配置目录与工作区概念下；--no-session-persistence 的会话不可恢复。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q14. fork-session 代码没分叉

只分叉对话。要用 --worktree。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q15. --continue 开错项目

它按当前目录找最近会话。先 cd。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q16. 后台任务看不到

openclaude ps；确认不是另一次安装的全局二进制。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q17. 如何停后台

openclaude kill 名称；或 /tasks。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q18. Windows 粘贴图片不行

用 Alt+V。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q19. Shift+Enter 没换行

/terminal-setup，或换终端。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q20. Vim 模式出不去

/vim 再切一次；Esc 是 Vim 语义。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q21. 中文乱码

终端 UTF-8；字体含 CJK。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q22. 代理 SSL 错误

NODE_EXTRA_CA_CERTS 指向公司 CA。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q23. 要走 HTTP 代理

HTTP_PROXY/HTTPS_PROXY。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q24. MCP 连不上

openclaude mcp doctor；检查命令是否在 PATH；看 stderr。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q25. 插件安装失败（Windows）

升级到含 marketplace cache ENOENT 修复的版本；关杀毒对缓存目录的锁。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q26. 技能不出现

/skills；看目录结构；特性开关；/reload-plugins。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q27. LSP 没诊断

/lsp status；先 install/recommend；确认语言服务器进程在。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q28. IDE 连不上

只自动连当恰好有一个有效 IDE；或多个时用 /ide。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q29. Chrome 集成没有

/chrome 或 --chrome；Beta。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q30. 成本不准

为该模型配 modelPricing；确认没有看错 cache 行。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q31. 本地 Ollama 报错

ollama serve 在跑；模型已 pull；base url 默认 11434。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q32. LM Studio 连不上

打开本地 server，端口与 /provider 一致。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q33. GitHub Models 401

重新 /onboard-github；token 权限。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q34. Bedrock 凭证

AWS 默认链；确保预取安全条件满足。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q35. Vertex 凭证

gcloud auth 应用默认凭证。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q36. Codex OAuth 后仍失败

看是否 Responses API 模型名；重新 /provider。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q37. DeepSeek 无思考

选 reasoner 类模型；effort 映射因模型而异。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q38. 工具结果被截断

正常。Agent 应再 Read 落盘文件。调 BASH_MAX_OUTPUT_LENGTH。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q39. 它删了不该删的文件

/rewind；收紧权限；把 rm 放 deny。立刻 git checkout。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q40. 如何只允许测试命令

allow Bash(bun test *) 同时默认询问其它 Bash。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q41. 如何禁止网络

deny WebFetch、WebSearch、以及 Bash(curl *) 等。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q42. 如何在 CI 跑

print 模式 + 明确 tools + 把密钥放 CI secret + 可信 checkout。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q43. json 输出解析失败

用 --output-format json 并读文档字段；stream-json 要按行解析。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q44. schema 校验失败

模型可能不遵守；换强模型或开 SyntheticOutput 路径。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q45. 心跳是干什么的

编排器检测无头进程还活着。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q46. --bare 少了 CLAUDE.md

故意的。自己 --system-prompt 或 --add-dir。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q47. 如何附加系统提示

--append-system-prompt，不要轻易 --system-prompt 全换，会丢掉工具说明。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q48. 自定义 Agent 不生效

/agents 看是否加载；--agent 名字是否匹配；工具是否被 CUSTOM_AGENT_DISALLOWED_TOOLS 去掉。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q49. 子 Agent 花很多钱

限制类型为 Explore；设预算；关 swarm。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q50. 如何设预算

SDK maxBudgetUsd；或自定义价格后看 /cost 人工停。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q51. smartroute 乱路由

先 off；simple 模型太弱会误伤。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q52. provider 回退不触发

检查 providerFallbackChain id 是否是 profile id；错误是否被分类为额度类。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q53. cache 从不命中

/cache-probe；避免每回合改系统提示；工具列表要稳定。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q54. 如何看请求谁最大

/request-size。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q55. repomap 没有

特性/命令 /repomap；大仓首次构建可能超时。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q56. knowledge graph 空

/knowledge enable yes 再 ingest。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q57. wiki 怎么用

/wiki init 然后 scan/ingest。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q58. dream 会不会把秘密写进记忆

可能。先扫；teamMemorySync 有 secretScanner，但仍要人工。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q59. 如何关闭遥测

隐私设置 /privacy-settings；以及非必要流量环境变量。开源构建有 stub。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q60. buddy 怎么关

/buddy mute。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q61. 广告怎么关

/ads off。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q62. 主题太花

/theme 选暗色。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q63. 状态行太吵

/statusline 配置。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q64. 如何换模型别名

/model 列表；描述符 catalog。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q65. qwen 窗口不对

modelLimits 写 1048576 等。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q66. 图片提示 No image found

可能超 5MB/8000px，看具体错误。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q67. PDF 读不全

有页数阈值；拆文件。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q68. Edit 说文件被修改

外部改过。让它先 Read。不要并行人手编辑同一文件。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q69. Grep 忽略了 node_modules

默认 ignore。需要时缩小路径。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q70. 工作区信任点错

清配置里的信任记录，重启。不要对陌生目录点信任。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q71. 多仓库 monorepo

在根开；或 /add-dir 子项目；worktree.multiRepo 逻辑处理父路径。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q72. tmux 没用

需要 --worktree；或 --tmux=classic。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q73. QR 扫不了

/session 或 /mobile；终端要真彩/支持图像协议效果更好。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q74. 远程会话安全吗

走官方远程时看账号；自建桥接是实验面，勿暴露公网。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q75. 语音不听

VOICE_MODE 未开或没有麦克风权限。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q76. Slack 安装失败

/install-slack-app 按提示；要有管理权限。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q77. GitHub App 权限过大

审查安装页；可只用 gh token 做只读 review。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q78. PR 评论拉不到

gh auth；/pr-comments；网络。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q79. auto-fix 循环

测试本身红；先修测试。/auto-fix 关掉。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q80. commit 信息带 AI 署名

/commit-message 配置。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q81. 如何完全离线

Ollama + 关非必要流量 + 不用 MCP 网络服务器。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q82. 公司禁止 npm 全局

用项目 devDependency 或 npx；或内部 registry。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q83. 版本与文档不符

以 --version 为准；站点 seeded from 源码，旧发行版会落后。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q84. 如何报告 bug

/feedback 或 GitHub issue；附 /doctor 与脱敏 /status。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q85. 安全漏洞

按 SECURITY.md，不要公开 issue。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q86. 贡献代码

读 CONTRIBUTING 与 AGENTS.md；先开 issue；跑 validation。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q87. 重复 PR 会被关

先搜现有 PR。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q88. CodeRabbit 评论要理吗

要。超范围可拒绝并说明。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q89. 测试跑很慢

max-concurrency=1 是故意的；只跑相关文件 bun test ./path。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q90. typecheck 失败

Node/Bun 版本；生成类型 scripts。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q91. smoke 失败

先 build；看 dist/cli.mjs --version。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q92. 隐私扫描红

不要把电话回家的 URL 加进产物。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q93. Docker 里跑

装 Node22、rg；挂载仓库；配密钥；注意 --print 跳过信任。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q94. 作为 systemd 服务

不支持官方 daemon 用户路径；用 --bg 或自己的编排器调 --print。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q95. 能否多用户一台机器

用不同 OPENCLAUDE_CONFIG_DIR 与系统用户隔离。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q96. 密钥写进 CLAUDE.md 了

立刻轮换密钥；从 git 历史清；加 secret scan。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q97. 模型乱改无关文件

提示里限定路径；deny 其它目录；用 worktree。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q98. 如何让它先测试再改

写进 CLAUDE.md 与 PreToolUse hook。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q99. SDK 里如何做权限

提供 canUseTool 回调；测试见 tests/sdk/permissions.test.ts。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q100. 输出要稳定 JSON

--print --output-format json --json-schema ... 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q101. 如何恢复误 compact

transcript 仍可能有全文；rewind/resume 旧会话。没有绝对保证。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q102. Ultraplan/Torch 没有

特性旗标未在你的构建打开。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q103. Coordinator 怎么开

构建特性 COORDINATOR_MODE；普通 npm 包可能没有。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q104. 和 Claude Code 关系

OpenClaude 是开源编码 Agent CLI，强调多模型；命令与工具概念相近但实现与集成层独立。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q105. 商业使用

看 LICENSE；依赖各自厂商 ToS。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q106. 数据是否上传

请求会发到你选的 provider。隐私设置控制非必要遥测。用本地模型可减少外传。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q107. 如何删除会话

删配置目录下的 session 存储；先 /export 备份。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q108. 配置目录在哪

/doctor 会打印；默认 ~/.openclaude。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q109. 为何启动慢

钥匙串、MDM、插件同步。--bare 更快。后续版本持续做预取并行。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q110. Enter 时有像素人射箭

Buddy。/buddy mute。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q111. 如何关闭更新检查

全局配置 autoUpdates。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q112. 模型列表太长

/provider 选好默认；/model 搜。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q113. 工具太多 schema 爆了

产品有 ToolSearch 延迟加载；否则 --tools 限制集合。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q114. PowerShell 而不是 Bash

Windows 上若启用 PowerShell 工具会走它；也可明确要求 bash。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q115. 路径是 Windows 反斜杠

内部会规范化；规则里写路径时注意。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q116. 多个 OPENAI_BASE_URL 冲突

unset 其它；用 profile 而不是一堆 env。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q117. ApiSmart 没选中

不要同时设冲突端点；APISMART_API_KEY 在无冲突时选中。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q118. 免费模型很弱

换强模型做执行，用 smartroute 把闲聊分出去。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q119. 如何教它项目规范

CLAUDE.md + 示例 + hooks 拒绝违规命令。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q120. 团队共享 MCP

项目 settings 写 mcpServers，密钥用环境变量占位。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q121. 如何审计它跑过的命令

transcript /replay；hooks 记日志。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q122. 误允许了危险命令

立即 Ctrl+C；检查 git status；轮换可能泄露的密钥。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q123. max turns 到了

提高上限或拆任务；看 REPL max turns 参数。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q124. goal 不结束

/goal status；条件太模糊。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q125. btw 问了但主任务丢了

btw 是旁路；主对话应仍在。用 /resume。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q126. tag 有什么用

方便以后 /resume 搜索。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q127. replay 卡顿

大会话正常；导出后再看。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q128. insights 报告空

要有足够会话历史。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q129. stats 不准确

跨机器不共享除非同步配置目录。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q130. release-notes 打不开

看网络；或去 GitHub Releases。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q131. stickers 是认真的吗

是彩蛋式命令，不影响编码。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q132. color 只改当前会话

对，提示条颜色。主题才是全局。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q133. logo 只是启动画面

对。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q134. desktop/mobile 命令

用于官方客户端接力；开源用户可能不可用。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q135. teleport 失败

环境选择与 git bundle 要求干净状态；先提交或 worktree。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q136. from-pr 找不到

PR 未关联会话；或 gh 未登录。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q137. session-id 必须 UUID

是，非法 id 会被拒绝。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q138. name 显示在哪

/resume 列表与终端标题。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q139. disable-slash-commands 后技能也没了

文档写明含 skill-backed。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q140. strict-mcp-config 忽略项目 MCP

对，只信命令行，适合 CI。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q141. setting-sources 只读 user

可用来忽略被投毒的项目 settings。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q142. agents JSON 格式错

启动会报 InvalidConfig；用 /agents UI 更安全。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q143. plugin-dir 重复加载

可重复传；注意信任。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q144. file 下载失败

--file file_id:relative_path 需要 Files API 配置。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q145. heartbeat 在 REPL 无效

仅 --print。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q146. include-partial-messages 很大

流式 token 级，磁盘会涨。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q147. json-schema 与工具并存

最终答案仍要符合 schema；工具过程不受 schema 约束。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q148. input-format stream-json 怎么写

按 SDK/编排文档逐行写 user 消息。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q149. 为何 --print 不问信任

脚本无法点对话框。所以只在可信目录用。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q150. 沙箱没开

/sandbox-toggle；不是所有平台都有 sandbox-runtime。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q151. classifier 乱拒绝

auto 模式实验性；改回 default。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q152. dontAsk 仍问

有 alwaysAsk 规则或破坏性检测。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q153. fullAccess 仍被拦

policy 与 killswitch 高于模式。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q154. bubble 模式是什么

内部宿主 UI，一般用户看不到。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q155. 如何循环模式

Shift+Tab。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q156. Ctrl+E 没反应

只在权限对话框打开解释面板。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q157. Ctrl+T 待办空

模型还没 TodoWrite。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q158. Ctrl+O 太长

那是全文 transcript。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q159. 历史搜索很慢

会话太多；tag 或改名。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q160. 外部编辑器没开

设 $EDITOR。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q161. 暂存草稿丢了

Ctrl+S 后再 Ctrl+S 或按 UI 取回；不要清会话。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q162. Undo 快捷键

Ctrl+_ 或 Ctrl+Shift+-。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q163. 菜单移动

Ctrl+P/N。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q164. 重绘花屏

Ctrl+L。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q165. 双重 Ctrl+C 才退出

第一次中断回合，第二次才可能走退出流。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q166. 它不停跑测试

提示里限制；或 deny Bash(bun test *) 以外。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q167. 如何让它用中文回答

在提示或 CLAUDE.md 写“请用中文”。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q168. 如何固定用某一个 profile

/provider 设默认；或 --provider。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q169. profile 文件能否提交

通常含密钥，不要提交。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q170. settings.local 示例

本机 allow 更宽的 Bash，不带密钥。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q171. 托管策略从哪来

远程 managed settings / MDM；用户不可用项目文件覆盖某些项。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q172. subscriptionType 被项目改了

不会，只有 user 级生效，防伪装。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q173. 如何确认没有 phone-home

bun run verify:privacy；关非必要流量。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q174. 扩展与 CLI 版本不一致

让扩展调用 PATH 里同一二进制。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q175. 文档站与 CLI 命令不一致

提交修复 web/src/data；本使用说明书以 CLI 源码为准。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。

### Q176. 这个项目去年的 Agent 和现在有何不同

工具更多、压缩更复杂、多模型集成描述符化。用法上你仍从 REPL 说话，但应开始用 plan、worktree、skills、MCP。详见 3SOLUTION.md。 若这条仍不能解决，请收集：`openclaude --version`、`/doctor` 输出、脱敏后的 `/status`、你使用的 provider/model、复现步骤、是否 --bare/--print、权限模式、以及相关日志（`--debug-file`）。不要把 API 密钥贴到 issue。安全类问题走 SECURITY.md。


## 10. 推荐工作流清单

1. 信任工作区 → 2. /provider → 3. /status → 4. 写或审 CLAUDE.md → 5. 默认权限模式做一次小改动 → 6. /diff → 7. 跑测试 → 8. 大改用 /plan 或 --worktree → 9. /commit → 10. 长会话 /compact 或新开会话。

把 OpenClaude 当同事而不是当编译器：它会错，会幻觉路径，会在弱模型上不调工具。你的职责是权限、测试、diff 和提交。

## 11. 获取更多帮助

项目讨论区、Discord、GitHub Issues、`/feedback`、`/help`。贡献前读 CONTRIBUTING.md。本使用说明书不替代厂商 API 文档，也不替代你所在公司的保密规定。
