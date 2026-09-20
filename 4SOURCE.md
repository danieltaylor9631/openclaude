# OpenClaude 源代码说明
**文档编号：** OC-SOURCE-2026-09
**统计范围：** 仓库内 TypeScript/JavaScript 源文件，排除 `node_modules`、`.git`、`dist`、`coverage`。
**统计日期：** 以生成该文档时的工作区快照为准。

## 1. 源代码整体介绍

### 1.1 语言与模块

OpenClaude 的主语言是 **TypeScript**（严格模式、ESM）。交互界面使用 **TSX**（React 组件，运行于定制 Ink 渲染器）。 VS Code 扩展目录以 **JavaScript** 为主。构建脚本以 TypeScript 编写，由 **Bun** 执行。 安装给终端用户的 CLI 运行在 **Node.js >= 22** 上，产物为打包后的 `dist/cli.mjs` 与 `dist/sdk.mjs`。

### 1.2 开发工具

- 包管理与测试：Bun（lockfile、`bun test`、`bun run` 脚本）
- 类型检查：TypeScript 5.9（`tsc --noEmit` 与 type-tests）
- 打包：`scripts/build.ts`（Bun bundler）
- 死代码：knip
- 校验：Zod
- CLI：commander
- 终端 UI：React 19 + Ink 定制层
- 协议：MCP SDK、可选 gRPC
- 编辑器：任意，仓库含 VS Code 扩展与 launch 配置
- CI：GitHub Actions `pr-checks.yml`
- 代码审阅辅助：CodeRabbit（流程要求见 CONTRIBUTING）

### 1.3 规模数字

- 源代码文件总数（ts/tsx/js/mjs/cjs）：**3313**
- 源代码总行数：**903157**
- `.ts` 文件：2627；`.tsx` 文件：645；`js/mjs/cjs` 文件：41
- 测试文件（文件名含 `.test.`）：**739**，测试行数合计 **230228**
- 运行时 npm `dependencies` 仅 Orama 与 ripgrep，其余库在开发依赖中并由打包器吸入 CLI 产物。
- 产品版本见根 `package.json`：`@gitlawb/openclaude`。

### 1.4 按顶层目录的文件数与行数

| 目录 | 文件数 | 行数 |
| --- | ---: | ---: |
| `src` | 3218 | 878287 |
| `scripts` | 39 | 9711 |
| `tests` | 22 | 7567 |
| `vscode-extension` | 16 | 5932 |
| `web` | 13 | 1394 |
| `bin` | 4 | 263 |
| `vendor` | 1 | 3 |

其中 `src` 是绝对主体。`src/utils` 是最大的共享层，其次是 `src/services`、`src/components`、`src/tools`、`src/commands`。 阅读源码的建议顺序：`package.json` → `src/entrypoints/cli.tsx` / `src/main.tsx` → `src/query.ts` → `src/Tool.ts` → `src/tools.ts` → `src/commands.ts` → `src/integrations/`。

### 1.5 源码树心智模型

可以把源码想成五条纵切：入口（entrypoints/main/cli）、会话内核（query/QueryEngine/state）、能力（tools/commands/skills/plugins/mcp）、适配（services/api 与 integrations）、表现（components/ink/vscode/web）。横切关注点放在 utils、types、permissions、constants。 测试文件普遍与实现并列，便于移动模块。生成物在 `src/integrations/generated` 与 SDK generated types。 不要手改生成物。

### 1.6 构建产物与不在统计内的内容

`dist/` 不计入源码行数。`node_modules/` 不计入。Markdown 文档、JSON 配置、图片、proto 定义若不是 ts/js 则不在本清单的数字里，但它们对运行同样重要：例如 `docs/integrations/` 是集成作者的规范，`.github/workflows/pr-checks.yml` 是质量门禁。

## 2. 源代码文件清单

下列清单覆盖统计范围内的每一个源文件。字段包括：路径（即文件名与所在目录）、行数、分类功能、补充说明。 行数为物理行，含空行与注释，用于体量判断，不等于圈复杂度。

### 2.1 目录组 `bin/heap-limit.mjs`（1 个文件，约 220 行）

该组位于仓库相对路径 `bin/heap-limit.mjs`。可执行入口包装。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `heap-limit.mjs`

- 所在目录：`bin`
- 完整路径：`bin/heap-limit.mjs`
- 行数：220
- 主要功能简介：可执行入口包装。
- 说明：所在目录 `bin` 的职责见上文分类。

### 2.2 目录组 `bin/import-specifier.mjs`（1 个文件，约 13 行）

该组位于仓库相对路径 `bin/import-specifier.mjs`。可执行入口包装。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `import-specifier.mjs`

- 所在目录：`bin`
- 完整路径：`bin/import-specifier.mjs`
- 行数：13
- 主要功能简介：可执行入口包装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `bin` 的职责见上文分类。

### 2.3 目录组 `bin/import-specifier.test.mjs`（1 个文件，约 13 行）

该组位于仓库相对路径 `bin/import-specifier.test.mjs`。可执行入口包装。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `import-specifier.test.mjs`

- 所在目录：`bin`
- 完整路径：`bin/import-specifier.test.mjs`
- 行数：13
- 主要功能简介：可执行入口包装。
- 说明：运行方式：`bun test ./bin/import-specifier.test.mjs`。短文件，多为常量、再导出或薄包装。所在目录 `bin` 的职责见上文分类。

### 2.4 目录组 `bin/node-compile-cache.mjs`（1 个文件，约 17 行）

该组位于仓库相对路径 `bin/node-compile-cache.mjs`。可执行入口包装。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `node-compile-cache.mjs`

- 所在目录：`bin`
- 完整路径：`bin/node-compile-cache.mjs`
- 行数：17
- 主要功能简介：可执行入口包装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `bin` 的职责见上文分类。

### 2.5 目录组 `scripts/benchmark-openclaude-startup.mjs`（1 个文件，约 206 行）

该组位于仓库相对路径 `scripts/benchmark-openclaude-startup.mjs`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `benchmark-openclaude-startup.mjs`

- 所在目录：`scripts`
- 完整路径：`scripts/benchmark-openclaude-startup.mjs`
- 行数：206
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.6 目录组 `scripts/build.ts`（1 个文件，约 1083 行）

该组位于仓库相对路径 `scripts/build.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `build.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/build.ts`
- 行数：1083
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：体量较大（约 1083 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `scripts` 的职责见上文分类。

### 2.7 目录组 `scripts/externals.ts`（1 个文件，约 208 行）

该组位于仓库相对路径 `scripts/externals.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `externals.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/externals.ts`
- 行数：208
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.8 目录组 `scripts/externalsValidation.test.ts`（1 个文件，约 344 行）

该组位于仓库相对路径 `scripts/externalsValidation.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `externalsValidation.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/externalsValidation.test.ts`
- 行数：344
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/externalsValidation.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.9 目录组 `scripts/externalsValidation.ts`（1 个文件，约 332 行）

该组位于仓库相对路径 `scripts/externalsValidation.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `externalsValidation.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/externalsValidation.ts`
- 行数：332
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.10 目录组 `scripts/feature-flags-source-guard.test.ts`（1 个文件，约 48 行）

该组位于仓库相对路径 `scripts/feature-flags-source-guard.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `feature-flags-source-guard.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/feature-flags-source-guard.test.ts`
- 行数：48
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/feature-flags-source-guard.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.11 目录组 `scripts/fixtures`（1 个文件，约 24 行）

该组位于仓库相对路径 `scripts/fixtures`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `instrument-node-compile-cache.mjs`

- 所在目录：`scripts/fixtures`
- 完整路径：`scripts/fixtures/instrument-node-compile-cache.mjs`
- 行数：24
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `scripts/fixtures` 的职责见上文分类。

### 2.12 目录组 `scripts/generate-integrations-artifacts.ts`（1 个文件，约 20 行）

该组位于仓库相对路径 `scripts/generate-integrations-artifacts.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `generate-integrations-artifacts.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/generate-integrations-artifacts.ts`
- 行数：20
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `scripts` 的职责见上文分类。

### 2.13 目录组 `scripts/generate-sdk-types.ts`（1 个文件，约 474 行）

该组位于仓库相对路径 `scripts/generate-sdk-types.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `generate-sdk-types.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/generate-sdk-types.ts`
- 行数：474
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.14 目录组 `scripts/grpc-cli.ts`（1 个文件，约 121 行）

该组位于仓库相对路径 `scripts/grpc-cli.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `grpc-cli.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/grpc-cli.ts`
- 行数：121
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.15 目录组 `scripts/missing-module-stub.test.ts`（1 个文件，约 60 行）

该组位于仓库相对路径 `scripts/missing-module-stub.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `missing-module-stub.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/missing-module-stub.test.ts`
- 行数：60
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/missing-module-stub.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.16 目录组 `scripts/no-ant-employee-gates.test.ts`（1 个文件，约 137 行）

该组位于仓库相对路径 `scripts/no-ant-employee-gates.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `no-ant-employee-gates.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/no-ant-employee-gates.test.ts`
- 行数：137
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/no-ant-employee-gates.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.17 目录组 `scripts/no-raw-abort-signal-timeout.test.ts`（1 个文件，约 118 行）

该组位于仓库相对路径 `scripts/no-raw-abort-signal-timeout.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `no-raw-abort-signal-timeout.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/no-raw-abort-signal-timeout.test.ts`
- 行数：118
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/no-raw-abort-signal-timeout.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.18 目录组 `scripts/no-telemetry-growthbook-stub.test.ts`（1 个文件，约 171 行）

该组位于仓库相对路径 `scripts/no-telemetry-growthbook-stub.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `no-telemetry-growthbook-stub.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/no-telemetry-growthbook-stub.test.ts`
- 行数：171
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/no-telemetry-growthbook-stub.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.19 目录组 `scripts/no-telemetry-plugin.ts`（1 个文件，约 140 行）

该组位于仓库相对路径 `scripts/no-telemetry-plugin.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `no-telemetry-plugin.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/no-telemetry-plugin.ts`
- 行数：140
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.20 目录组 `scripts/openclaude-bin-compile-cache.test.ts`（1 个文件，约 204 行）

该组位于仓库相对路径 `scripts/openclaude-bin-compile-cache.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `openclaude-bin-compile-cache.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/openclaude-bin-compile-cache.test.ts`
- 行数：204
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/openclaude-bin-compile-cache.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.21 目录组 `scripts/openclaude-bin-heap.test.ts`（1 个文件，约 318 行）

该组位于仓库相对路径 `scripts/openclaude-bin-heap.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `openclaude-bin-heap.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/openclaude-bin-heap.test.ts`
- 行数：318
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/openclaude-bin-heap.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.22 目录组 `scripts/optionalRuntimeSpecifiers.test.ts`（1 个文件，约 99 行）

该组位于仓库相对路径 `scripts/optionalRuntimeSpecifiers.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `optionalRuntimeSpecifiers.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/optionalRuntimeSpecifiers.test.ts`
- 行数：99
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/optionalRuntimeSpecifiers.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.23 目录组 `scripts/pr-intent-scan.test.ts`（1 个文件，约 198 行）

该组位于仓库相对路径 `scripts/pr-intent-scan.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `pr-intent-scan.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/pr-intent-scan.test.ts`
- 行数：198
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/pr-intent-scan.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.24 目录组 `scripts/pr-intent-scan.ts`（1 个文件，约 470 行）

该组位于仓库相对路径 `scripts/pr-intent-scan.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `pr-intent-scan.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/pr-intent-scan.ts`
- 行数：470
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.25 目录组 `scripts/provider-bootstrap.ts`（1 个文件，约 197 行）

该组位于仓库相对路径 `scripts/provider-bootstrap.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-bootstrap.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-bootstrap.ts`
- 行数：197
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.26 目录组 `scripts/provider-discovery.ts`（1 个文件，约 13 行）

该组位于仓库相对路径 `scripts/provider-discovery.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-discovery.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-discovery.ts`
- 行数：13
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `scripts` 的职责见上文分类。

### 2.27 目录组 `scripts/provider-launch.test.ts`（1 个文件，约 94 行）

该组位于仓库相对路径 `scripts/provider-launch.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-launch.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-launch.test.ts`
- 行数：94
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/provider-launch.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.28 目录组 `scripts/provider-launch.ts`（1 个文件，约 300 行）

该组位于仓库相对路径 `scripts/provider-launch.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-launch.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-launch.ts`
- 行数：300
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.29 目录组 `scripts/provider-recommend.test.ts`（1 个文件，约 96 行）

该组位于仓库相对路径 `scripts/provider-recommend.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-recommend.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-recommend.test.ts`
- 行数：96
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/provider-recommend.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.30 目录组 `scripts/provider-recommend.ts`（1 个文件，约 277 行）

该组位于仓库相对路径 `scripts/provider-recommend.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `provider-recommend.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/provider-recommend.ts`
- 行数：277
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.31 目录组 `scripts/reactJsxDevRuntimeProductionShim.js`（1 个文件，约 18 行）

该组位于仓库相对路径 `scripts/reactJsxDevRuntimeProductionShim.js`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `reactJsxDevRuntimeProductionShim.js`

- 所在目录：`scripts`
- 完整路径：`scripts/reactJsxDevRuntimeProductionShim.js`
- 行数：18
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `scripts` 的职责见上文分类。

### 2.32 目录组 `scripts/reactJsxDevRuntimeProductionShim.test.ts`（1 个文件，约 43 行）

该组位于仓库相对路径 `scripts/reactJsxDevRuntimeProductionShim.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `reactJsxDevRuntimeProductionShim.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/reactJsxDevRuntimeProductionShim.test.ts`
- 行数：43
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/reactJsxDevRuntimeProductionShim.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.33 目录组 `scripts/render-coverage-heatmap.ts`（1 个文件，约 393 行）

该组位于仓库相对路径 `scripts/render-coverage-heatmap.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `render-coverage-heatmap.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/render-coverage-heatmap.ts`
- 行数：393
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.34 目录组 `scripts/start-grpc.ts`（1 个文件，约 47 行）

该组位于仓库相对路径 `scripts/start-grpc.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `start-grpc.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/start-grpc.ts`
- 行数：47
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.35 目录组 `scripts/stubMarkerGuard.test.ts`（1 个文件，约 87 行）

该组位于仓库相对路径 `scripts/stubMarkerGuard.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `stubMarkerGuard.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/stubMarkerGuard.test.ts`
- 行数：87
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/stubMarkerGuard.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.36 目录组 `scripts/stubMarkerGuard.ts`（1 个文件，约 65 行）

该组位于仓库相对路径 `scripts/stubMarkerGuard.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `stubMarkerGuard.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/stubMarkerGuard.ts`
- 行数：65
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.37 目录组 `scripts/system-check.test.ts`（1 个文件，约 1015 行）

该组位于仓库相对路径 `scripts/system-check.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `system-check.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/system-check.test.ts`
- 行数：1015
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/system-check.test.ts`。体量较大（约 1015 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `scripts` 的职责见上文分类。

### 2.38 目录组 `scripts/system-check.ts`（1 个文件，约 1449 行）

该组位于仓库相对路径 `scripts/system-check.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `system-check.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/system-check.ts`
- 行数：1449
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：体量较大（约 1449 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `scripts` 的职责见上文分类。

### 2.39 目录组 `scripts/typecheck-type-tests.ts`（1 个文件，约 89 行）

该组位于仓库相对路径 `scripts/typecheck-type-tests.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `typecheck-type-tests.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/typecheck-type-tests.ts`
- 行数：89
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.40 目录组 `scripts/validate-externals.ts`（1 个文件，约 143 行）

该组位于仓库相对路径 `scripts/validate-externals.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `validate-externals.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/validate-externals.ts`
- 行数：143
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.41 目录组 `scripts/verify-clean-install.test.ts`（1 个文件，约 76 行）

该组位于仓库相对路径 `scripts/verify-clean-install.test.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `verify-clean-install.test.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/verify-clean-install.test.ts`
- 行数：76
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./scripts/verify-clean-install.test.ts`。所在目录 `scripts` 的职责见上文分类。

### 2.42 目录组 `scripts/verify-clean-install.ts`（1 个文件，约 487 行）

该组位于仓库相对路径 `scripts/verify-clean-install.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `verify-clean-install.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/verify-clean-install.ts`
- 行数：487
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.43 目录组 `scripts/verify-no-phone-home.ts`（1 个文件，约 47 行）

该组位于仓库相对路径 `scripts/verify-no-phone-home.ts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `verify-no-phone-home.ts`

- 所在目录：`scripts`
- 完整路径：`scripts/verify-no-phone-home.ts`
- 行数：47
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `scripts` 的职责见上文分类。

### 2.44 目录组 `src/QueryEngine.autoCompactCooldown.test.ts`（1 个文件，约 41 行）

该组位于仓库相对路径 `src/QueryEngine.autoCompactCooldown.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `QueryEngine.autoCompactCooldown.test.ts`

- 所在目录：`src`
- 完整路径：`src/QueryEngine.autoCompactCooldown.test.ts`
- 行数：41
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/QueryEngine.autoCompactCooldown.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.45 目录组 `src/QueryEngine.customPricingBudget.test.ts`（1 个文件，约 37 行）

该组位于仓库相对路径 `src/QueryEngine.customPricingBudget.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `QueryEngine.customPricingBudget.test.ts`

- 所在目录：`src`
- 完整路径：`src/QueryEngine.customPricingBudget.test.ts`
- 行数：37
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/QueryEngine.customPricingBudget.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.46 目录组 `src/QueryEngine.interruptionTrace.test.ts`（1 个文件，约 183 行）

该组位于仓库相对路径 `src/QueryEngine.interruptionTrace.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `QueryEngine.interruptionTrace.test.ts`

- 所在目录：`src`
- 完整路径：`src/QueryEngine.interruptionTrace.test.ts`
- 行数：183
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/QueryEngine.interruptionTrace.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.47 目录组 `src/QueryEngine.ts`（1 个文件，约 1555 行）

该组位于仓库相对路径 `src/QueryEngine.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `QueryEngine.ts`

- 所在目录：`src`
- 完整路径：`src/QueryEngine.ts`
- 行数：1555
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：体量较大（约 1555 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.48 目录组 `src/Task.ts`（1 个文件，约 124 行）

该组位于仓库相对路径 `src/Task.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `Task.ts`

- 所在目录：`src`
- 完整路径：`src/Task.ts`
- 行数：124
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.49 目录组 `src/Tool.ts`（1 个文件，约 827 行）

该组位于仓库相对路径 `src/Tool.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `Tool.ts`

- 所在目录：`src`
- 完整路径：`src/Tool.ts`
- 行数：827
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：体量较大（约 827 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.50 目录组 `src/__tests__`（6 个文件，约 1606 行）

该组位于仓库相对路径 `src/__tests__`。跨模块测试。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `bugfixes.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/bugfixes.test.ts`
- 行数：715
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/bugfixes.test.ts`。所在目录 `src/__tests__` 的职责见上文分类。

#### `doctorContextWarnings.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/doctorContextWarnings.test.ts`
- 行数：178
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/doctorContextWarnings.test.ts`。所在目录 `src/__tests__` 的职责见上文分类。

#### `process-title.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/process-title.test.ts`
- 行数：13
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/process-title.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/__tests__` 的职责见上文分类。

#### `providerCounts.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/providerCounts.test.ts`
- 行数：55
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/providerCounts.test.ts`。所在目录 `src/__tests__` 的职责见上文分类。

#### `security-hardening.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/security-hardening.test.ts`
- 行数：191
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/security-hardening.test.ts`。所在目录 `src/__tests__` 的职责见上文分类。

#### `statusNoticeLocalModel.test.ts`

- 所在目录：`src/__tests__`
- 完整路径：`src/__tests__/statusNoticeLocalModel.test.ts`
- 行数：454
- 主要功能简介：跨模块测试。
- 说明：运行方式：`bun test ./src/__tests__/statusNoticeLocalModel.test.ts`。所在目录 `src/__tests__` 的职责见上文分类。

### 2.51 目录组 `src/assistant`（7 个文件，约 677 行）

该组位于仓库相对路径 `src/assistant`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `AssistantSessionChooser.tsx`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/AssistantSessionChooser.tsx`
- 行数：10
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/assistant` 的职责见上文分类。

#### `gate.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/gate.ts`
- 行数：12
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/assistant` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/index.ts`
- 行数：51
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/assistant` 的职责见上文分类。

#### `sessionDiscovery.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/sessionDiscovery.ts`
- 行数：30
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/assistant` 的职责见上文分类。

#### `sessionHistory.test.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/sessionHistory.test.ts`
- 行数：249
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/assistant/sessionHistory.test.ts`。所在目录 `src/assistant` 的职责见上文分类。

#### `sessionHistory.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/sessionHistory.ts`
- 行数：210
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/assistant` 的职责见上文分类。

#### `sessionHistorySerialization.ts`

- 所在目录：`src/assistant`
- 完整路径：`src/assistant/sessionHistorySerialization.ts`
- 行数：115
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/assistant` 的职责见上文分类。

### 2.52 目录组 `src/bootstrap`（2 个文件，约 1902 行）

该组位于仓库相对路径 `src/bootstrap`。会话引导状态。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `state.modelUsageProto.test.ts`

- 所在目录：`src/bootstrap`
- 完整路径：`src/bootstrap/state.modelUsageProto.test.ts`
- 行数：98
- 主要功能简介：会话引导状态。
- 说明：运行方式：`bun test ./src/bootstrap/state.modelUsageProto.test.ts`。所在目录 `src/bootstrap` 的职责见上文分类。

#### `state.ts`

- 所在目录：`src/bootstrap`
- 完整路径：`src/bootstrap/state.ts`
- 行数：1804
- 主要功能简介：会话引导状态。
- 说明：体量较大（约 1804 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/bootstrap` 的职责见上文分类。

### 2.53 目录组 `src/bridge`（37 个文件，约 13202 行）

该组位于仓库相对路径 `src/bridge`。远程桥接。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `bridgeApi.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeApi.ts`
- 行数：539
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeConfig.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeConfig.ts`
- 行数：41
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeDebug.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeDebug.ts`
- 行数：135
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeEnabled.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeEnabled.ts`
- 行数：202
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeMain.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeMain.ts`
- 行数：2984
- 主要功能简介：远程桥接。
- 说明：体量较大（约 2984 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeMessaging.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeMessaging.ts`
- 行数：467
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgePermissionCallbacks.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgePermissionCallbacks.ts`
- 行数：43
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgePointer.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgePointer.ts`
- 行数：210
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeStatusUtil.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeStatusUtil.ts`
- 行数：163
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `bridgeUI.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/bridgeUI.ts`
- 行数：530
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `capacityWake.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/capacityWake.ts`
- 行数：56
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `codeSessionApi.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/codeSessionApi.ts`
- 行数：168
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `createSession.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/createSession.ts`
- 行数：398
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `debugUtils.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/debugUtils.ts`
- 行数：141
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `envLessBridgeConfig.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/envLessBridgeConfig.ts`
- 行数：165
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `flushGate.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/flushGate.ts`
- 行数：71
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `inboundAttachments.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/inboundAttachments.ts`
- 行数：175
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `inboundMessages.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/inboundMessages.ts`
- 行数：84
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `initReplBridge.titleTruncation.test.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/initReplBridge.titleTruncation.test.ts`
- 行数：62
- 主要功能简介：远程桥接。
- 说明：运行方式：`bun test ./src/bridge/initReplBridge.titleTruncation.test.ts`。所在目录 `src/bridge` 的职责见上文分类。

#### `initReplBridge.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/initReplBridge.ts`
- 行数：602
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `jwtUtils.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/jwtUtils.ts`
- 行数：256
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `peerSessions.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/peerSessions.ts`
- 行数：30
- 主要功能简介：远程桥接。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/bridge` 的职责见上文分类。

#### `pollConfig.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/pollConfig.ts`
- 行数：111
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `pollConfigDefaults.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/pollConfigDefaults.ts`
- 行数：82
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `remoteBridgeCore.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/remoteBridgeCore.ts`
- 行数：1019
- 主要功能简介：远程桥接。
- 说明：体量较大（约 1019 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/bridge` 的职责见上文分类。

#### `replBridge.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/replBridge.ts`
- 行数：2451
- 主要功能简介：远程桥接。
- 说明：体量较大（约 2451 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/bridge` 的职责见上文分类。

#### `replBridgeHandle.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/replBridgeHandle.ts`
- 行数：36
- 主要功能简介：远程桥接。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/bridge` 的职责见上文分类。

#### `replBridgeTransport.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/replBridgeTransport.ts`
- 行数：377
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `replBridgeTransport.types.test.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/replBridgeTransport.types.test.ts`
- 行数：99
- 主要功能简介：远程桥接。
- 说明：运行方式：`bun test ./src/bridge/replBridgeTransport.types.test.ts`。所在目录 `src/bridge` 的职责见上文分类。

#### `sessionIdCompat.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/sessionIdCompat.ts`
- 行数：57
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `sessionRunner.test.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/sessionRunner.test.ts`
- 行数：85
- 主要功能简介：远程桥接。
- 说明：运行方式：`bun test ./src/bridge/sessionRunner.test.ts`。所在目录 `src/bridge` 的职责见上文分类。

#### `sessionRunner.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/sessionRunner.ts`
- 行数：601
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `trustedDevice.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/trustedDevice.ts`
- 行数：210
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/types.ts`
- 行数：262
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

#### `webhookSanitizer.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/webhookSanitizer.ts`
- 行数：20
- 主要功能简介：远程桥接。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/bridge` 的职责见上文分类。

#### `workSecret.test.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/workSecret.test.ts`
- 行数：104
- 主要功能简介：远程桥接。
- 说明：运行方式：`bun test ./src/bridge/workSecret.test.ts`。所在目录 `src/bridge` 的职责见上文分类。

#### `workSecret.ts`

- 所在目录：`src/bridge`
- 完整路径：`src/bridge/workSecret.ts`
- 行数：166
- 主要功能简介：远程桥接。
- 说明：所在目录 `src/bridge` 的职责见上文分类。

### 2.54 目录组 `src/buddy`（20 个文件，约 3428 行）

该组位于仓库相对路径 `src/buddy`。像素伙伴。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `CompanionActionFX.test.tsx`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/CompanionActionFX.test.tsx`
- 行数：84
- 主要功能简介：像素伙伴。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/buddy/CompanionActionFX.test.tsx`。所在目录 `src/buddy` 的职责见上文分类。

#### `CompanionActionFX.tsx`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/CompanionActionFX.tsx`
- 行数：79
- 主要功能简介：像素伙伴。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/buddy` 的职责见上文分类。

#### `CompanionSprite.test.tsx`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/CompanionSprite.test.tsx`
- 行数：166
- 主要功能简介：像素伙伴。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/buddy/CompanionSprite.test.tsx`。所在目录 `src/buddy` 的职责见上文分类。

#### `CompanionSprite.tsx`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/CompanionSprite.tsx`
- 行数：506
- 主要功能简介：像素伙伴。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/buddy` 的职责见上文分类。

#### `actionEffects.test.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/actionEffects.test.ts`
- 行数：105
- 主要功能简介：像素伙伴。
- 说明：运行方式：`bun test ./src/buddy/actionEffects.test.ts`。所在目录 `src/buddy` 的职责见上文分类。

#### `actionEffects.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/actionEffects.ts`
- 行数：312
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

#### `companion.test.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/companion.test.ts`
- 行数：85
- 主要功能简介：像素伙伴。
- 说明：运行方式：`bun test ./src/buddy/companion.test.ts`。所在目录 `src/buddy` 的职责见上文分类。

#### `companion.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/companion.ts`
- 行数：157
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

#### `deterministic.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/deterministic.ts`
- 行数：17
- 主要功能简介：像素伙伴。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/buddy` 的职责见上文分类。

#### `feature.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/feature.ts`
- 行数：3
- 主要功能简介：像素伙伴。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/buddy` 的职责见上文分类。

#### `observer.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/observer.ts`
- 行数：53
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

#### `pixelSprites.test.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/pixelSprites.test.ts`
- 行数：98
- 主要功能简介：像素伙伴。
- 说明：运行方式：`bun test ./src/buddy/pixelSprites.test.ts`。所在目录 `src/buddy` 的职责见上文分类。

#### `pixelSprites.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/pixelSprites.ts`
- 行数：869
- 主要功能简介：像素伙伴。
- 说明：体量较大（约 869 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/buddy` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/prompt.ts`
- 行数：43
- 主要功能简介：像素伙伴。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/buddy` 的职责见上文分类。

#### `sprites.test.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/sprites.test.ts`
- 行数：108
- 主要功能简介：像素伙伴。
- 说明：运行方式：`bun test ./src/buddy/sprites.test.ts`。所在目录 `src/buddy` 的职责见上文分类。

#### `sprites.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/sprites.ts`
- 行数：398
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

#### `types.test.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/types.test.ts`
- 行数：33
- 主要功能简介：像素伙伴。
- 说明：运行方式：`bun test ./src/buddy/types.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/buddy` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/types.ts`
- 行数：141
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

#### `useBuddyNotification.tsx`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/useBuddyNotification.tsx`
- 行数：95
- 主要功能简介：像素伙伴。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/buddy` 的职责见上文分类。

#### `useShotClock.ts`

- 所在目录：`src/buddy`
- 完整路径：`src/buddy/useShotClock.ts`
- 行数：76
- 主要功能简介：像素伙伴。
- 说明：所在目录 `src/buddy` 的职责见上文分类。

### 2.55 目录组 `src/cli`（57 个文件，约 27275 行）

该组位于仓库相对路径 `src/cli`。非 REPL 子命令。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `aimlapiCommand.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/aimlapiCommand.test.ts`
- 行数：166
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/aimlapiCommand.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `aimlapiCommand.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/aimlapiCommand.ts`
- 行数：101
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

#### `bg.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bg.test.ts`
- 行数：1845
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/bg.test.ts`。体量较大（约 1845 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `bg.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bg.ts`
- 行数：1235
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 1235 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `bgFinalizer.fixture.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgFinalizer.fixture.ts`
- 行数：36
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli` 的职责见上文分类。

#### `bgFinalizer.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgFinalizer.test.ts`
- 行数：621
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/bgFinalizer.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `bgFinalizer.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgFinalizer.ts`
- 行数：238
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

#### `bgRegistry.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgRegistry.test.ts`
- 行数：1867
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/bgRegistry.test.ts`。体量较大（约 1867 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `bgRegistry.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgRegistry.ts`
- 行数：1176
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 1176 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `bgRouting.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/bgRouting.ts`
- 行数：55
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

#### `taskReport.ts`

- 所在目录：`src/cli/commands`
- 完整路径：`src/cli/commands/taskReport.ts`
- 行数：75
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/commands` 的职责见上文分类。

#### `exit.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/exit.ts`
- 行数：31
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli` 的职责见上文分类。

#### `agents.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/agents.ts`
- 行数：70
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `aimlapi.test.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/aimlapi.test.ts`
- 行数：85
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/handlers/aimlapi.test.ts`。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `aimlapi.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/aimlapi.ts`
- 行数：29
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `auth.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/auth.ts`
- 行数：330
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `autoMode.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/autoMode.ts`
- 行数：169
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `doctorReport.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/doctorReport.ts`
- 行数：35
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `mcp.tsx`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/mcp.tsx`
- 行数：459
- 主要功能简介：非 REPL 子命令。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `plugins.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/plugins.ts`
- 行数：878
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 878 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skills.test.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skills.test.ts`
- 行数：1345
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/handlers/skills.test.ts`。体量较大（约 1345 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skills.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skills.ts`
- 行数：229
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsCli.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsCli.ts`
- 行数：312
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsInstall.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsInstall.ts`
- 行数：807
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 807 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsListFormat.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsListFormat.ts`
- 行数：218
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsRemoveMessage.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsRemoveMessage.ts`
- 行数：45
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsValidation.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsValidation.ts`
- 行数：210
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsVerify.test.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsVerify.test.ts`
- 行数：496
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/handlers/skillsVerify.test.ts`。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `skillsVerify.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/skillsVerify.ts`
- 行数：351
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `taskReport.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/taskReport.ts`
- 行数：105
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `templateJobs.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/templateJobs.ts`
- 行数：12
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `util.tsx`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/util.tsx`
- 行数：109
- 主要功能简介：非 REPL 子命令。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `xaiAuth.test.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/xaiAuth.test.ts`
- 行数：190
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/handlers/xaiAuth.test.ts`。所在目录 `src/cli/handlers` 的职责见上文分类。

#### `xaiAuth.ts`

- 所在目录：`src/cli/handlers`
- 完整路径：`src/cli/handlers/xaiAuth.ts`
- 行数：289
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/handlers` 的职责见上文分类。

#### `headlessHeartbeat.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/headlessHeartbeat.test.ts`
- 行数：531
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/headlessHeartbeat.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `headlessHeartbeat.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/headlessHeartbeat.ts`
- 行数：367
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

#### `ndjsonSafeStringify.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/ndjsonSafeStringify.ts`
- 行数：32
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli` 的职责见上文分类。

#### `print.interruptionTrace.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/print.interruptionTrace.test.ts`
- 行数：108
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/print.interruptionTrace.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `print.sdkModelOptions.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/print.sdkModelOptions.test.ts`
- 行数：76
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/print.sdkModelOptions.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `print.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/print.ts`
- 行数：5877
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 5877 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `printHeartbeat.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/printHeartbeat.test.ts`
- 行数：196
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/printHeartbeat.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `printInterruption.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/printInterruption.ts`
- 行数：37
- 主要功能简介：非 REPL 子命令。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/cli` 的职责见上文分类。

#### `printMaxTurns.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/printMaxTurns.test.ts`
- 行数：223
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/printMaxTurns.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `remoteIO.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/remoteIO.ts`
- 行数：255
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

#### `structuredIO.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/structuredIO.ts`
- 行数：924
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 924 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli` 的职责见上文分类。

#### `HybridTransport.test.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/HybridTransport.test.ts`
- 行数：207
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/transports/HybridTransport.test.ts`。所在目录 `src/cli/transports` 的职责见上文分类。

#### `HybridTransport.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/HybridTransport.ts`
- 行数：314
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `SSETransport.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/SSETransport.ts`
- 行数：711
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `SerialBatchEventUploader.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/SerialBatchEventUploader.ts`
- 行数：275
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `Transport.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/Transport.ts`
- 行数：51
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `WebSocketTransport.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/WebSocketTransport.ts`
- 行数：800
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `WorkerStateUploader.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/WorkerStateUploader.ts`
- 行数：131
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `ccrClient.test.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/ccrClient.test.ts`
- 行数：318
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/transports/ccrClient.test.ts`。所在目录 `src/cli/transports` 的职责见上文分类。

#### `ccrClient.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/ccrClient.ts`
- 行数：1054
- 主要功能简介：非 REPL 子命令。
- 说明：体量较大（约 1054 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/cli/transports` 的职责见上文分类。

#### `transportUtils.ts`

- 所在目录：`src/cli/transports`
- 完整路径：`src/cli/transports/transportUtils.ts`
- 行数：45
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli/transports` 的职责见上文分类。

#### `update.test.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/update.test.ts`
- 行数：61
- 主要功能简介：非 REPL 子命令。
- 说明：运行方式：`bun test ./src/cli/update.test.ts`。所在目录 `src/cli` 的职责见上文分类。

#### `update.ts`

- 所在目录：`src/cli`
- 完整路径：`src/cli/update.ts`
- 行数：463
- 主要功能简介：非 REPL 子命令。
- 说明：所在目录 `src/cli` 的职责见上文分类。

### 2.56 目录组 `src/commands`（297 个文件，约 47772 行）

该组位于仓库相对路径 `src/commands`。斜杠命令或 CLI 命令实现。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `add-dir.tsx`

- 所在目录：`src/commands/add-dir`
- 完整路径：`src/commands/add-dir/add-dir.tsx`
- 行数：96
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/add-dir` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/add-dir`
- 完整路径：`src/commands/add-dir/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/add-dir` 的职责见上文分类。

#### `validation.ts`

- 所在目录：`src/commands/add-dir`
- 完整路径：`src/commands/add-dir/validation.ts`
- 行数：110
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/add-dir` 的职责见上文分类。

#### `ads.test.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/ads.test.ts`
- 行数：90
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/ads.test.ts`。所在目录 `src/commands` 的职责见上文分类。

#### `ads.tsx`

- 所在目录：`src/commands`
- 完整路径：`src/commands/ads.tsx`
- 行数：158
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands` 的职责见上文分类。

#### `advisor.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/advisor.ts`
- 行数：109
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/agents-platform`
- 完整路径：`src/commands/agents-platform/index.ts`
- 行数：2
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/agents-platform` 的职责见上文分类。

#### `agents.tsx`

- 所在目录：`src/commands/agents`
- 完整路径：`src/commands/agents/agents.tsx`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/agents` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/agents`
- 完整路径：`src/commands/agents/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/agents` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/ant-trace`
- 完整路径：`src/commands/ant-trace/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/ant-trace` 的职责见上文分类。

#### `AssistantSessionChooser.ts`

- 所在目录：`src/commands/assistant`
- 完整路径：`src/commands/assistant/AssistantSessionChooser.ts`
- 行数：2
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/assistant` 的职责见上文分类。

#### `assistant.ts`

- 所在目录：`src/commands/assistant`
- 完整路径：`src/commands/assistant/assistant.ts`
- 行数：31
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/assistant` 的职责见上文分类。

#### `auto-fix.test.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/auto-fix.test.ts`
- 行数：19
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/auto-fix.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/commands` 的职责见上文分类。

#### `auto-fix.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/auto-fix.ts`
- 行数：28
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/autofix-pr`
- 完整路径：`src/commands/autofix-pr/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/autofix-pr` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/backfill-sessions`
- 完整路径：`src/commands/backfill-sessions/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/backfill-sessions` 的职责见上文分类。

#### `branch.test.ts`

- 所在目录：`src/commands/branch`
- 完整路径：`src/commands/branch/branch.test.ts`
- 行数：993
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/branch/branch.test.ts`。体量较大（约 993 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/branch` 的职责见上文分类。

#### `branch.ts`

- 所在目录：`src/commands/branch`
- 完整路径：`src/commands/branch/branch.ts`
- 行数：558
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/branch` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/branch`
- 完整路径：`src/commands/branch/index.ts`
- 行数：15
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/branch` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/break-cache`
- 完整路径：`src/commands/break-cache/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/break-cache` 的职责见上文分类。

#### `bridge-kick.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/bridge-kick.ts`
- 行数：200
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `bridge.tsx`

- 所在目录：`src/commands/bridge`
- 完整路径：`src/commands/bridge/bridge.tsx`
- 行数：508
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/bridge` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/bridge`
- 完整路径：`src/commands/bridge/index.ts`
- 行数：26
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/bridge` 的职责见上文分类。

#### `brief.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/brief.ts`
- 行数：130
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `btw.tsx`

- 所在目录：`src/commands/btw`
- 完整路径：`src/commands/btw/btw.tsx`
- 行数：242
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/btw` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/btw`
- 完整路径：`src/commands/btw/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/btw` 的职责见上文分类。

#### `buddy.tsx`

- 所在目录：`src/commands/buddy`
- 完整路径：`src/commands/buddy/buddy.tsx`
- 行数：323
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/buddy` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/buddy`
- 完整路径：`src/commands/buddy/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/buddy` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/bughunter-perf`
- 完整路径：`src/commands/bughunter-perf/index.ts`
- 行数：264
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/bughunter-perf` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/bughunter-security`
- 完整路径：`src/commands/bughunter-security/index.ts`
- 行数：284
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/bughunter-security` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/bughunter`
- 完整路径：`src/commands/bughunter/index.ts`
- 行数：235
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/bughunter` 的职责见上文分类。

#### `cache-probe.test.ts`

- 所在目录：`src/commands/cache-probe`
- 完整路径：`src/commands/cache-probe/cache-probe.test.ts`
- 行数：205
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/cache-probe/cache-probe.test.ts`。所在目录 `src/commands/cache-probe` 的职责见上文分类。

#### `cache-probe.ts`

- 所在目录：`src/commands/cache-probe`
- 完整路径：`src/commands/cache-probe/cache-probe.ts`
- 行数：474
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/cache-probe` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/cache-probe`
- 完整路径：`src/commands/cache-probe/index.ts`
- 行数：17
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/cache-probe` 的职责见上文分类。

#### `cacheStats.test.ts`

- 所在目录：`src/commands/cacheStats`
- 完整路径：`src/commands/cacheStats/cacheStats.test.ts`
- 行数：171
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/cacheStats/cacheStats.test.ts`。所在目录 `src/commands/cacheStats` 的职责见上文分类。

#### `cacheStats.ts`

- 所在目录：`src/commands/cacheStats`
- 完整路径：`src/commands/cacheStats/cacheStats.ts`
- 行数：74
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/cacheStats` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/cacheStats`
- 完整路径：`src/commands/cacheStats/index.ts`
- 行数：24
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/cacheStats` 的职责见上文分类。

#### `chrome.tsx`

- 所在目录：`src/commands/chrome`
- 完整路径：`src/commands/chrome/chrome.tsx`
- 行数：284
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/chrome` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/chrome`
- 完整路径：`src/commands/chrome/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/chrome` 的职责见上文分类。

#### `clear-context-window.ts`

- 所在目录：`src/commands/clear-context-window`
- 完整路径：`src/commands/clear-context-window/clear-context-window.ts`
- 行数：21
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/clear-context-window` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/clear-context-window`
- 完整路径：`src/commands/clear-context-window/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/clear-context-window` 的职责见上文分类。

#### `caches.ts`

- 所在目录：`src/commands/clear`
- 完整路径：`src/commands/clear/caches.ts`
- 行数：153
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/clear` 的职责见上文分类。

#### `clear.ts`

- 所在目录：`src/commands/clear`
- 完整路径：`src/commands/clear/clear.ts`
- 行数：7
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/clear` 的职责见上文分类。

#### `conversation.goal.test.ts`

- 所在目录：`src/commands/clear`
- 完整路径：`src/commands/clear/conversation.goal.test.ts`
- 行数：49
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/clear/conversation.goal.test.ts`。所在目录 `src/commands/clear` 的职责见上文分类。

#### `conversation.ts`

- 所在目录：`src/commands/clear`
- 完整路径：`src/commands/clear/conversation.ts`
- 行数：263
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/clear` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/clear`
- 完整路径：`src/commands/clear/index.ts`
- 行数：19
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/clear` 的职责见上文分类。

#### `color.ts`

- 所在目录：`src/commands/color`
- 完整路径：`src/commands/color/color.ts`
- 行数：93
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/color` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/color`
- 完整路径：`src/commands/color/index.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/color` 的职责见上文分类。

#### `commit-message.test.ts`

- 所在目录：`src/commands/commit-message`
- 完整路径：`src/commands/commit-message/commit-message.test.ts`
- 行数：90
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/commit-message/commit-message.test.ts`。所在目录 `src/commands/commit-message` 的职责见上文分类。

#### `commit-message.ts`

- 所在目录：`src/commands/commit-message`
- 完整路径：`src/commands/commit-message/commit-message.ts`
- 行数：173
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/commit-message` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/commit-message`
- 完整路径：`src/commands/commit-message/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/commit-message` 的职责见上文分类。

#### `commit-push-pr.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/commit-push-pr.ts`
- 行数：164
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `commit.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/commit.ts`
- 行数：98
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `compact.ts`

- 所在目录：`src/commands/compact`
- 完整路径：`src/commands/compact/compact.ts`
- 行数：298
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/compact` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/compact`
- 完整路径：`src/commands/compact/index.ts`
- 行数：15
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/compact` 的职责见上文分类。

#### `config.tsx`

- 所在目录：`src/commands/config`
- 完整路径：`src/commands/config/config.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/config` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/config`
- 完整路径：`src/commands/config/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/config` 的职责见上文分类。

#### `context-noninteractive.ts`

- 所在目录：`src/commands/context`
- 完整路径：`src/commands/context/context-noninteractive.ts`
- 行数：332
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/context` 的职责见上文分类。

#### `context.tsx`

- 所在目录：`src/commands/context`
- 完整路径：`src/commands/context/context.tsx`
- 行数：72
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/context` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/context`
- 完整路径：`src/commands/context/index.ts`
- 行数：24
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/context` 的职责见上文分类。

#### `copy.tsx`

- 所在目录：`src/commands/copy`
- 完整路径：`src/commands/copy/copy.tsx`
- 行数：370
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/copy` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/copy`
- 完整路径：`src/commands/copy/index.ts`
- 行数：15
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/copy` 的职责见上文分类。

#### `cost.ts`

- 所在目录：`src/commands/cost`
- 完整路径：`src/commands/cost/cost.ts`
- 行数：24
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/cost` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/cost`
- 完整路径：`src/commands/cost/index.ts`
- 行数：23
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/cost` 的职责见上文分类。

#### `createMovedToPluginCommand.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/createMovedToPluginCommand.ts`
- 行数：73
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `ctx-noninteractive.ts`

- 所在目录：`src/commands/ctx_viz`
- 完整路径：`src/commands/ctx_viz/ctx-noninteractive.ts`
- 行数：249
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/ctx_viz` 的职责见上文分类。

#### `ctx_viz.test.ts`

- 所在目录：`src/commands/ctx_viz`
- 完整路径：`src/commands/ctx_viz/ctx_viz.test.ts`
- 行数：253
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/ctx_viz/ctx_viz.test.ts`。所在目录 `src/commands/ctx_viz` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/ctx_viz`
- 完整路径：`src/commands/ctx_viz/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/ctx_viz` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/debug-tool-call`
- 完整路径：`src/commands/debug-tool-call/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/debug-tool-call` 的职责见上文分类。

#### `desktop.tsx`

- 所在目录：`src/commands/desktop`
- 完整路径：`src/commands/desktop/desktop.tsx`
- 行数：8
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/desktop` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/desktop`
- 完整路径：`src/commands/desktop/index.ts`
- 行数：26
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/desktop` 的职责见上文分类。

#### `diagnostics.test.ts`

- 所在目录：`src/commands/diagnostics`
- 完整路径：`src/commands/diagnostics/diagnostics.test.ts`
- 行数：318
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/diagnostics/diagnostics.test.ts`。所在目录 `src/commands/diagnostics` 的职责见上文分类。

#### `diagnostics.ts`

- 所在目录：`src/commands/diagnostics`
- 完整路径：`src/commands/diagnostics/diagnostics.ts`
- 行数：279
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/diagnostics` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/commands/diagnostics`
- 完整路径：`src/commands/diagnostics/index.test.ts`
- 行数：40
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/commands/diagnostics/index.test.ts`。所在目录 `src/commands/diagnostics` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/diagnostics`
- 完整路径：`src/commands/diagnostics/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/diagnostics` 的职责见上文分类。

#### `diff.tsx`

- 所在目录：`src/commands/diff`
- 完整路径：`src/commands/diff/diff.tsx`
- 行数：8
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/diff` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/diff`
- 完整路径：`src/commands/diff/index.ts`
- 行数：8
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/diff` 的职责见上文分类。

#### `doctor.test.tsx`

- 所在目录：`src/commands/doctor`
- 完整路径：`src/commands/doctor/doctor.test.tsx`
- 行数：185
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/doctor/doctor.test.tsx`。所在目录 `src/commands/doctor` 的职责见上文分类。

#### `doctor.tsx`

- 所在目录：`src/commands/doctor`
- 完整路径：`src/commands/doctor/doctor.tsx`
- 行数：113
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/doctor` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/doctor`
- 完整路径：`src/commands/doctor/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/doctor` 的职责见上文分类。

#### `dream.ts`

- 所在目录：`src/commands/dream`
- 完整路径：`src/commands/dream/dream.ts`
- 行数：68
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/dream` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/dream`
- 完整路径：`src/commands/dream/index.ts`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/dream` 的职责见上文分类。

#### `effort.test.tsx`

- 所在目录：`src/commands/effort`
- 完整路径：`src/commands/effort/effort.test.tsx`
- 行数：236
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/effort/effort.test.tsx`。所在目录 `src/commands/effort` 的职责见上文分类。

#### `effort.tsx`

- 所在目录：`src/commands/effort`
- 完整路径：`src/commands/effort/effort.tsx`
- 行数：228
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/effort` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/effort`
- 完整路径：`src/commands/effort/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/effort` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/env`
- 完整路径：`src/commands/env/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/env` 的职责见上文分类。

#### `exit.tsx`

- 所在目录：`src/commands/exit`
- 完整路径：`src/commands/exit/exit.tsx`
- 行数：32
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/exit` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/exit`
- 完整路径：`src/commands/exit/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/exit` 的职责见上文分类。

#### `export.test.ts`

- 所在目录：`src/commands/export`
- 完整路径：`src/commands/export/export.test.ts`
- 行数：294
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/export/export.test.ts`。所在目录 `src/commands/export` 的职责见上文分类。

#### `export.tsx`

- 所在目录：`src/commands/export`
- 完整路径：`src/commands/export/export.tsx`
- 行数：99
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/export` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/export`
- 完整路径：`src/commands/export/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/export` 的职责见上文分类。

#### `extra-usage-core.ts`

- 所在目录：`src/commands/extra-usage`
- 完整路径：`src/commands/extra-usage/extra-usage-core.ts`
- 行数：118
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/extra-usage` 的职责见上文分类。

#### `extra-usage-noninteractive.ts`

- 所在目录：`src/commands/extra-usage`
- 完整路径：`src/commands/extra-usage/extra-usage-noninteractive.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/extra-usage` 的职责见上文分类。

#### `extra-usage.tsx`

- 所在目录：`src/commands/extra-usage`
- 完整路径：`src/commands/extra-usage/extra-usage.tsx`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/extra-usage` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/extra-usage`
- 完整路径：`src/commands/extra-usage/index.ts`
- 行数：31
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/extra-usage` 的职责见上文分类。

#### `fast.customPricing.test.ts`

- 所在目录：`src/commands/fast`
- 完整路径：`src/commands/fast/fast.customPricing.test.ts`
- 行数：36
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/fast/fast.customPricing.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/fast` 的职责见上文分类。

#### `fast.tsx`

- 所在目录：`src/commands/fast`
- 完整路径：`src/commands/fast/fast.tsx`
- 行数：267
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/fast` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/fast`
- 完整路径：`src/commands/fast/index.ts`
- 行数：26
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/fast` 的职责见上文分类。

#### `feedback.tsx`

- 所在目录：`src/commands/feedback`
- 完整路径：`src/commands/feedback/feedback.tsx`
- 行数：24
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/feedback` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/feedback`
- 完整路径：`src/commands/feedback/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/feedback` 的职责见上文分类。

#### `files.ts`

- 所在目录：`src/commands/files`
- 完整路径：`src/commands/files/files.ts`
- 行数：19
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/files` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/files`
- 完整路径：`src/commands/files/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/files` 的职责见上文分类。

#### `goal.test.ts`

- 所在目录：`src/commands/goal`
- 完整路径：`src/commands/goal/goal.test.ts`
- 行数：213
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/goal/goal.test.ts`。所在目录 `src/commands/goal` 的职责见上文分类。

#### `goal.ts`

- 所在目录：`src/commands/goal`
- 完整路径：`src/commands/goal/goal.ts`
- 行数：137
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/goal` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/goal`
- 完整路径：`src/commands/goal/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/goal` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/good-claude`
- 完整路径：`src/commands/good-claude/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/good-claude` 的职责见上文分类。

#### `heapdump.ts`

- 所在目录：`src/commands/heapdump`
- 完整路径：`src/commands/heapdump/heapdump.ts`
- 行数：17
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/heapdump` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/heapdump`
- 完整路径：`src/commands/heapdump/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/heapdump` 的职责见上文分类。

#### `help.tsx`

- 所在目录：`src/commands/help`
- 完整路径：`src/commands/help/help.tsx`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/help` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/help`
- 完整路径：`src/commands/help/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/help` 的职责见上文分类。

#### `hooks.tsx`

- 所在目录：`src/commands/hooks`
- 完整路径：`src/commands/hooks/hooks.tsx`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/hooks` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/hooks`
- 完整路径：`src/commands/hooks/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/hooks` 的职责见上文分类。

#### `ide.tsx`

- 所在目录：`src/commands/ide`
- 完整路径：`src/commands/ide/ide.tsx`
- 行数：645
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/ide` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/ide`
- 完整路径：`src/commands/ide/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/ide` 的职责见上文分类。

#### `init-verifiers.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/init-verifiers.ts`
- 行数：262
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `init.test.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/init.test.ts`
- 行数：55
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/init.test.ts`。所在目录 `src/commands` 的职责见上文分类。

#### `init.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/init.ts`
- 行数：250
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `initMode.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/initMode.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands` 的职责见上文分类。

#### `insights.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/insights.ts`
- 行数：2934
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：体量较大（约 2934 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands` 的职责见上文分类。

#### `ApiKeyStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/ApiKeyStep.tsx`
- 行数：230
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `CheckExistingSecretStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/CheckExistingSecretStep.tsx`
- 行数：189
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `CheckGitHubStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/CheckGitHubStep.tsx`
- 行数：14
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `ChooseRepoStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/ChooseRepoStep.tsx`
- 行数：210
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `CreatingStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/CreatingStep.tsx`
- 行数：64
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `ErrorStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/ErrorStep.tsx`
- 行数：84
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `ExistingWorkflowStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/ExistingWorkflowStep.tsx`
- 行数：102
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `InstallAppStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/InstallAppStep.tsx`
- 行数：93
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `OAuthFlowStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/OAuthFlowStep.tsx`
- 行数：276
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `SuccessStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/SuccessStep.tsx`
- 行数：95
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `WarningsStep.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/WarningsStep.tsx`
- 行数：72
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `install-github-app.tsx`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/install-github-app.tsx`
- 行数：586
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `repoSlug.test.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/repoSlug.test.ts`
- 行数：48
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/install-github-app/repoSlug.test.ts`。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `repoSlug.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/repoSlug.ts`
- 行数：54
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `setupGitHubActions.test.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/setupGitHubActions.test.ts`
- 行数：279
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/install-github-app/setupGitHubActions.test.ts`。所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `setupGitHubActions.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/setupGitHubActions.ts`
- 行数：331
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/commands/install-github-app`
- 完整路径：`src/commands/install-github-app/types.ts`
- 行数：49
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/install-github-app` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/install-slack-app`
- 完整路径：`src/commands/install-slack-app/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/install-slack-app` 的职责见上文分类。

#### `install-slack-app.ts`

- 所在目录：`src/commands/install-slack-app`
- 完整路径：`src/commands/install-slack-app/install-slack-app.ts`
- 行数：30
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/install-slack-app` 的职责见上文分类。

#### `install.tsx`

- 所在目录：`src/commands`
- 完整路径：`src/commands/install.tsx`
- 行数：327
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/issue`
- 完整路径：`src/commands/issue/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/issue` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/keybindings`
- 完整路径：`src/commands/keybindings/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/keybindings` 的职责见上文分类。

#### `keybindings.ts`

- 所在目录：`src/commands/keybindings`
- 完整路径：`src/commands/keybindings/keybindings.ts`
- 行数：53
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/keybindings` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/knowledge`
- 完整路径：`src/commands/knowledge/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/knowledge` 的职责见上文分类。

#### `knowledge.test.ts`

- 所在目录：`src/commands/knowledge`
- 完整路径：`src/commands/knowledge/knowledge.test.ts`
- 行数：153
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/knowledge/knowledge.test.ts`。所在目录 `src/commands/knowledge` 的职责见上文分类。

#### `knowledge.ts`

- 所在目录：`src/commands/knowledge`
- 完整路径：`src/commands/knowledge/knowledge.ts`
- 行数：91
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/knowledge` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/login`
- 完整路径：`src/commands/login/index.ts`
- 行数：14
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/login` 的职责见上文分类。

#### `login.tsx`

- 所在目录：`src/commands/login`
- 完整路径：`src/commands/login/login.tsx`
- 行数：134
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/login` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/logo`
- 完整路径：`src/commands/logo/index.ts`
- 行数：21
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/logo` 的职责见上文分类。

#### `logo.tsx`

- 所在目录：`src/commands/logo`
- 完整路径：`src/commands/logo/logo.tsx`
- 行数：50
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/logo` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/logout`
- 完整路径：`src/commands/logout/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/logout` 的职责见上文分类。

#### `logout.tsx`

- 所在目录：`src/commands/logout`
- 完整路径：`src/commands/logout/logout.tsx`
- 行数：75
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/logout` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/lsp`
- 完整路径：`src/commands/lsp/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/lsp` 的职责见上文分类。

#### `lsp.test.ts`

- 所在目录：`src/commands/lsp`
- 完整路径：`src/commands/lsp/lsp.test.ts`
- 行数：690
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/lsp/lsp.test.ts`。所在目录 `src/commands/lsp` 的职责见上文分类。

#### `lsp.ts`

- 所在目录：`src/commands/lsp`
- 完整路径：`src/commands/lsp/lsp.ts`
- 行数：814
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：体量较大（约 814 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/lsp` 的职责见上文分类。

#### `addCommand.ts`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/addCommand.ts`
- 行数：280
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/mcp` 的职责见上文分类。

#### `doctorCommand.test.ts`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/doctorCommand.test.ts`
- 行数：19
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/mcp/doctorCommand.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/mcp` 的职责见上文分类。

#### `doctorCommand.ts`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/doctorCommand.ts`
- 行数：25
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/mcp` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/mcp` 的职责见上文分类。

#### `mcp.tsx`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/mcp.tsx`
- 行数：79
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/mcp` 的职责见上文分类。

#### `xaaIdpCommand.ts`

- 所在目录：`src/commands/mcp`
- 完整路径：`src/commands/mcp/xaaIdpCommand.ts`
- 行数：266
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/mcp` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/memory`
- 完整路径：`src/commands/memory/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/memory` 的职责见上文分类。

#### `memory.tsx`

- 所在目录：`src/commands/memory`
- 完整路径：`src/commands/memory/memory.tsx`
- 行数：89
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/memory` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/mobile`
- 完整路径：`src/commands/mobile/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/mobile` 的职责见上文分类。

#### `mobile.tsx`

- 所在目录：`src/commands/mobile`
- 完整路径：`src/commands/mobile/mobile.tsx`
- 行数：273
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/mobile` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/mock-limits`
- 完整路径：`src/commands/mock-limits/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/mock-limits` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/model`
- 完整路径：`src/commands/model/index.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/model` 的职责见上文分类。

#### `model.fastModeSwitch.test.ts`

- 所在目录：`src/commands/model`
- 完整路径：`src/commands/model/model.fastModeSwitch.test.ts`
- 行数：94
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/model/model.fastModeSwitch.test.ts`。所在目录 `src/commands/model` 的职责见上文分类。

#### `model.test.tsx`

- 所在目录：`src/commands/model`
- 完整路径：`src/commands/model/model.test.tsx`
- 行数：3830
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/model/model.test.tsx`。体量较大（约 3830 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/model` 的职责见上文分类。

#### `model.tsx`

- 所在目录：`src/commands/model`
- 完整路径：`src/commands/model/model.tsx`
- 行数：1394
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1394 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/model` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/oauth-refresh`
- 完整路径：`src/commands/oauth-refresh/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/oauth-refresh` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/onboard-github`
- 完整路径：`src/commands/onboard-github/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/onboard-github` 的职责见上文分类。

#### `onboard-github.test.ts`

- 所在目录：`src/commands/onboard-github`
- 完整路径：`src/commands/onboard-github/onboard-github.test.ts`
- 行数：211
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/onboard-github/onboard-github.test.ts`。所在目录 `src/commands/onboard-github` 的职责见上文分类。

#### `onboard-github.tsx`

- 所在目录：`src/commands/onboard-github`
- 完整路径：`src/commands/onboard-github/onboard-github.tsx`
- 行数：658
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/onboard-github` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/onboarding`
- 完整路径：`src/commands/onboarding/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/onboarding` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/output-style`
- 完整路径：`src/commands/output-style/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/output-style` 的职责见上文分类。

#### `output-style.tsx`

- 所在目录：`src/commands/output-style`
- 完整路径：`src/commands/output-style/output-style.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/output-style` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/passes`
- 完整路径：`src/commands/passes/index.ts`
- 行数：22
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/passes` 的职责见上文分类。

#### `passes.tsx`

- 所在目录：`src/commands/passes`
- 完整路径：`src/commands/passes/passes.tsx`
- 行数：23
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/passes` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/peers`
- 完整路径：`src/commands/peers/index.ts`
- 行数：21
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/peers` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/perf-issue`
- 完整路径：`src/commands/perf-issue/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/perf-issue` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/permissions`
- 完整路径：`src/commands/permissions/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/permissions` 的职责见上文分类。

#### `permissions.tsx`

- 所在目录：`src/commands/permissions`
- 完整路径：`src/commands/permissions/permissions.tsx`
- 行数：9
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/permissions` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/plan`
- 完整路径：`src/commands/plan/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/plan` 的职责见上文分类。

#### `plan.tsx`

- 所在目录：`src/commands/plan`
- 完整路径：`src/commands/plan/plan.tsx`
- 行数：121
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plan` 的职责见上文分类。

#### `AddMarketplace.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/AddMarketplace.tsx`
- 行数：161
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `BrowseMarketplace.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/BrowseMarketplace.tsx`
- 行数：803
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 803 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `DiscoverPlugins.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/DiscoverPlugins.tsx`
- 行数：780
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `ManageMarketplaces.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/ManageMarketplaces.tsx`
- 行数：837
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 837 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `ManagePlugins.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/ManagePlugins.tsx`
- 行数：2220
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 2220 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `PluginErrors.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/PluginErrors.tsx`
- 行数：123
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `PluginOptionsDialog.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/PluginOptionsDialog.tsx`
- 行数：356
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `PluginOptionsFlow.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/PluginOptionsFlow.tsx`
- 行数：134
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `PluginSettings.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/PluginSettings.tsx`
- 行数：1082
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1082 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `PluginTrustWarning.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/PluginTrustWarning.tsx`
- 行数：31
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `UnifiedInstalledCell.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/UnifiedInstalledCell.tsx`
- 行数：564
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `ValidatePlugin.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/ValidatePlugin.tsx`
- 行数：97
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `index.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/index.tsx`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `parseArgs.ts`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/parseArgs.ts`
- 行数：103
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/plugin` 的职责见上文分类。

#### `plugin.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/plugin.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `pluginDetailsHelpers.tsx`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/pluginDetailsHelpers.tsx`
- 行数：116
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/types.ts`
- 行数：35
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/plugin` 的职责见上文分类。

#### `unifiedTypes.ts`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/unifiedTypes.ts`
- 行数：58
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/plugin` 的职责见上文分类。

#### `usePagination.ts`

- 所在目录：`src/commands/plugin`
- 完整路径：`src/commands/plugin/usePagination.ts`
- 行数：171
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/plugin` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/pr_comments`
- 完整路径：`src/commands/pr_comments/index.ts`
- 行数：50
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/pr_comments` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/privacy-settings`
- 完整路径：`src/commands/privacy-settings/index.ts`
- 行数：14
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/privacy-settings` 的职责见上文分类。

#### `privacy-settings.tsx`

- 所在目录：`src/commands/privacy-settings`
- 完整路径：`src/commands/privacy-settings/privacy-settings.tsx`
- 行数：57
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/privacy-settings` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/provider`
- 完整路径：`src/commands/provider/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/provider` 的职责见上文分类。

#### `provider.test.tsx`

- 所在目录：`src/commands/provider`
- 完整路径：`src/commands/provider/provider.test.tsx`
- 行数：811
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/provider/provider.test.tsx`。体量较大（约 811 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/provider` 的职责见上文分类。

#### `provider.tsx`

- 所在目录：`src/commands/provider`
- 完整路径：`src/commands/provider/provider.tsx`
- 行数：1821
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1821 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/commands/provider` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/rate-limit-options`
- 完整路径：`src/commands/rate-limit-options/index.ts`
- 行数：19
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/rate-limit-options` 的职责见上文分类。

#### `rate-limit-options.tsx`

- 所在目录：`src/commands/rate-limit-options`
- 完整路径：`src/commands/rate-limit-options/rate-limit-options.tsx`
- 行数：209
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/rate-limit-options` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/release-notes`
- 完整路径：`src/commands/release-notes/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/release-notes` 的职责见上文分类。

#### `release-notes.ts`

- 所在目录：`src/commands/release-notes`
- 完整路径：`src/commands/release-notes/release-notes.ts`
- 行数：52
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/release-notes` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/reload-plugins`
- 完整路径：`src/commands/reload-plugins/index.ts`
- 行数：18
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/reload-plugins` 的职责见上文分类。

#### `reload-plugins.ts`

- 所在目录：`src/commands/reload-plugins`
- 完整路径：`src/commands/reload-plugins/reload-plugins.ts`
- 行数：61
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/reload-plugins` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/remote-env`
- 完整路径：`src/commands/remote-env/index.ts`
- 行数：15
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/remote-env` 的职责见上文分类。

#### `remote-env.tsx`

- 所在目录：`src/commands/remote-env`
- 完整路径：`src/commands/remote-env/remote-env.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/remote-env` 的职责见上文分类。

#### `api.ts`

- 所在目录：`src/commands/remote-setup`
- 完整路径：`src/commands/remote-setup/api.ts`
- 行数：182
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/remote-setup` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/remote-setup`
- 完整路径：`src/commands/remote-setup/index.ts`
- 行数：20
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/remote-setup` 的职责见上文分类。

#### `remote-setup.tsx`

- 所在目录：`src/commands/remote-setup`
- 完整路径：`src/commands/remote-setup/remote-setup.tsx`
- 行数：186
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/remote-setup` 的职责见上文分类。

#### `generateSessionName.ts`

- 所在目录：`src/commands/rename`
- 完整路径：`src/commands/rename/generateSessionName.ts`
- 行数：67
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/rename` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/rename`
- 完整路径：`src/commands/rename/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/rename` 的职责见上文分类。

#### `rename.ts`

- 所在目录：`src/commands/rename`
- 完整路径：`src/commands/rename/rename.ts`
- 行数：87
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/rename` 的职责见上文分类。

#### `ReplayTimeline.tsx`

- 所在目录：`src/commands/replay`
- 完整路径：`src/commands/replay/ReplayTimeline.tsx`
- 行数：175
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/replay` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/replay`
- 完整路径：`src/commands/replay/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/replay` 的职责见上文分类。

#### `replay.test.tsx`

- 所在目录：`src/commands/replay`
- 完整路径：`src/commands/replay/replay.test.tsx`
- 行数：378
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/replay/replay.test.tsx`。所在目录 `src/commands/replay` 的职责见上文分类。

#### `replay.tsx`

- 所在目录：`src/commands/replay`
- 完整路径：`src/commands/replay/replay.tsx`
- 行数：188
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/replay` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/repomap`
- 完整路径：`src/commands/repomap/index.ts`
- 行数：17
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/repomap` 的职责见上文分类。

#### `repomap.test.ts`

- 所在目录：`src/commands/repomap`
- 完整路径：`src/commands/repomap/repomap.test.ts`
- 行数：299
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/repomap/repomap.test.ts`。所在目录 `src/commands/repomap` 的职责见上文分类。

#### `repomap.ts`

- 所在目录：`src/commands/repomap`
- 完整路径：`src/commands/repomap/repomap.ts`
- 行数：169
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/repomap` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/request-size`
- 完整路径：`src/commands/request-size/index.ts`
- 行数：24
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/request-size` 的职责见上文分类。

#### `request-size-noninteractive.ts`

- 所在目录：`src/commands/request-size`
- 完整路径：`src/commands/request-size/request-size-noninteractive.ts`
- 行数：27
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/request-size` 的职责见上文分类。

#### `request-size.test.ts`

- 所在目录：`src/commands/request-size`
- 完整路径：`src/commands/request-size/request-size.test.ts`
- 行数：263
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/request-size/request-size.test.ts`。所在目录 `src/commands/request-size` 的职责见上文分类。

#### `request-size.tsx`

- 所在目录：`src/commands/request-size`
- 完整路径：`src/commands/request-size/request-size.tsx`
- 行数：56
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/request-size` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/reset-limits`
- 完整路径：`src/commands/reset-limits/index.js`
- 行数：4
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/reset-limits` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/resume`
- 完整路径：`src/commands/resume/index.ts`
- 行数：22
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/resume` 的职责见上文分类。

#### `resume.test.tsx`

- 所在目录：`src/commands/resume`
- 完整路径：`src/commands/resume/resume.test.tsx`
- 行数：556
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/commands/resume/resume.test.tsx`。所在目录 `src/commands/resume` 的职责见上文分类。

#### `resume.tsx`

- 所在目录：`src/commands/resume`
- 完整路径：`src/commands/resume/resume.tsx`
- 行数：555
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/resume` 的职责见上文分类。

#### `review.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/review.ts`
- 行数：57
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `UltrareviewOverageDialog.tsx`

- 所在目录：`src/commands/review`
- 完整路径：`src/commands/review/UltrareviewOverageDialog.tsx`
- 行数：95
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/review` 的职责见上文分类。

#### `reviewRemote.ts`

- 所在目录：`src/commands/review`
- 完整路径：`src/commands/review/reviewRemote.ts`
- 行数：316
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/review` 的职责见上文分类。

#### `ultrareviewCommand.tsx`

- 所在目录：`src/commands/review`
- 完整路径：`src/commands/review/ultrareviewCommand.tsx`
- 行数：57
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/review` 的职责见上文分类。

#### `ultrareviewEnabled.ts`

- 所在目录：`src/commands/review`
- 完整路径：`src/commands/review/ultrareviewEnabled.ts`
- 行数：14
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/review` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/rewind`
- 完整路径：`src/commands/rewind/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/rewind` 的职责见上文分类。

#### `rewind.ts`

- 所在目录：`src/commands/rewind`
- 完整路径：`src/commands/rewind/rewind.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/rewind` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/commands/sandbox-toggle`
- 完整路径：`src/commands/sandbox-toggle/index.test.ts`
- 行数：64
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/commands/sandbox-toggle/index.test.ts`。所在目录 `src/commands/sandbox-toggle` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/sandbox-toggle`
- 完整路径：`src/commands/sandbox-toggle/index.ts`
- 行数：56
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/sandbox-toggle` 的职责见上文分类。

#### `sandbox-toggle.tsx`

- 所在目录：`src/commands/sandbox-toggle`
- 完整路径：`src/commands/sandbox-toggle/sandbox-toggle.tsx`
- 行数：82
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/sandbox-toggle` 的职责见上文分类。

#### `security-review.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/security-review.ts`
- 行数：243
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/session`
- 完整路径：`src/commands/session/index.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/session` 的职责见上文分类。

#### `session.tsx`

- 所在目录：`src/commands/session`
- 完整路径：`src/commands/session/session.tsx`
- 行数：139
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/session` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/set-context-window`
- 完整路径：`src/commands/set-context-window/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/set-context-window` 的职责见上文分类。

#### `set-context-window.ts`

- 所在目录：`src/commands/set-context-window`
- 完整路径：`src/commands/set-context-window/set-context-window.ts`
- 行数：79
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/set-context-window` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/share`
- 完整路径：`src/commands/share/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/share` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/skills`
- 完整路径：`src/commands/skills/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/skills` 的职责见上文分类。

#### `skills.tsx`

- 所在目录：`src/commands/skills`
- 完整路径：`src/commands/skills/skills.tsx`
- 行数：7
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/skills` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/commands/smartroute`
- 完整路径：`src/commands/smartroute/index.test.ts`
- 行数：182
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/commands/smartroute/index.test.ts`。所在目录 `src/commands/smartroute` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/smartroute`
- 完整路径：`src/commands/smartroute/index.ts`
- 行数：134
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/smartroute` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/stats`
- 完整路径：`src/commands/stats/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/stats` 的职责见上文分类。

#### `stats.tsx`

- 所在目录：`src/commands/stats`
- 完整路径：`src/commands/stats/stats.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/stats` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/status`
- 完整路径：`src/commands/status/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/status` 的职责见上文分类。

#### `status.tsx`

- 所在目录：`src/commands/status`
- 完整路径：`src/commands/status/status.tsx`
- 行数：7
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/status` 的职责见上文分类。

#### `statusline.tsx`

- 所在目录：`src/commands`
- 完整路径：`src/commands/statusline.tsx`
- 行数：31
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/stickers`
- 完整路径：`src/commands/stickers/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/stickers` 的职责见上文分类。

#### `stickers.ts`

- 所在目录：`src/commands/stickers`
- 完整路径：`src/commands/stickers/stickers.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/stickers` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/summary`
- 完整路径：`src/commands/summary/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/summary` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/tag`
- 完整路径：`src/commands/tag/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/tag` 的职责见上文分类。

#### `tag.tsx`

- 所在目录：`src/commands/tag`
- 完整路径：`src/commands/tag/tag.tsx`
- 行数：215
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/tag` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/tasks`
- 完整路径：`src/commands/tasks/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/tasks` 的职责见上文分类。

#### `tasks.tsx`

- 所在目录：`src/commands/tasks`
- 完整路径：`src/commands/tasks/tasks.tsx`
- 行数：7
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/tasks` 的职责见上文分类。

#### `index.js`

- 所在目录：`src/commands/teleport`
- 完整路径：`src/commands/teleport/index.js`
- 行数：1
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/teleport` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/terminalSetup`
- 完整路径：`src/commands/terminalSetup/index.ts`
- 行数：23
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/terminalSetup` 的职责见上文分类。

#### `terminalSetup.tsx`

- 所在目录：`src/commands/terminalSetup`
- 完整路径：`src/commands/terminalSetup/terminalSetup.tsx`
- 行数：525
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/terminalSetup` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/theme`
- 完整路径：`src/commands/theme/index.ts`
- 行数：10
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/theme` 的职责见上文分类。

#### `theme.tsx`

- 所在目录：`src/commands/theme`
- 完整路径：`src/commands/theme/theme.tsx`
- 行数：56
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/theme` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/thinkback-play`
- 完整路径：`src/commands/thinkback-play/index.ts`
- 行数：17
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/thinkback-play` 的职责见上文分类。

#### `thinkback-play.ts`

- 所在目录：`src/commands/thinkback-play`
- 完整路径：`src/commands/thinkback-play/thinkback-play.ts`
- 行数：43
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/thinkback-play` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/thinkback`
- 完整路径：`src/commands/thinkback/index.ts`
- 行数：13
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/thinkback` 的职责见上文分类。

#### `thinkback.tsx`

- 所在目录：`src/commands/thinkback`
- 完整路径：`src/commands/thinkback/thinkback.tsx`
- 行数：550
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/thinkback` 的职责见上文分类。

#### `ultraplan.tsx`

- 所在目录：`src/commands`
- 完整路径：`src/commands/ultraplan.tsx`
- 行数：462
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/update`
- 完整路径：`src/commands/update/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/update` 的职责见上文分类。

#### `update.test.ts`

- 所在目录：`src/commands/update`
- 完整路径：`src/commands/update/update.test.ts`
- 行数：68
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：运行方式：`bun test ./src/commands/update/update.test.ts`。所在目录 `src/commands/update` 的职责见上文分类。

#### `update.tsx`

- 所在目录：`src/commands/update`
- 完整路径：`src/commands/update/update.tsx`
- 行数：377
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/update` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/upgrade`
- 完整路径：`src/commands/upgrade/index.ts`
- 行数：16
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/upgrade` 的职责见上文分类。

#### `upgrade.tsx`

- 所在目录：`src/commands/upgrade`
- 完整路径：`src/commands/upgrade/upgrade.tsx`
- 行数：37
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/upgrade` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/commands/usage`
- 完整路径：`src/commands/usage/index.test.ts`
- 行数：175
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/commands/usage/index.test.ts`。所在目录 `src/commands/usage` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/usage`
- 完整路径：`src/commands/usage/index.ts`
- 行数：179
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/commands/usage` 的职责见上文分类。

#### `usage.tsx`

- 所在目录：`src/commands/usage`
- 完整路径：`src/commands/usage/usage.tsx`
- 行数：6
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/usage` 的职责见上文分类。

#### `version.ts`

- 所在目录：`src/commands`
- 完整路径：`src/commands/version.ts`
- 行数：22
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/vim`
- 完整路径：`src/commands/vim/index.ts`
- 行数：11
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/vim` 的职责见上文分类。

#### `vim.ts`

- 所在目录：`src/commands/vim`
- 完整路径：`src/commands/vim/vim.ts`
- 行数：38
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/commands/vim` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/voice`
- 完整路径：`src/commands/voice/index.ts`
- 行数：20
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/voice` 的职责见上文分类。

#### `voice.ts`

- 所在目录：`src/commands/voice`
- 完整路径：`src/commands/voice/voice.ts`
- 行数：150
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：所在目录 `src/commands/voice` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/wiki`
- 完整路径：`src/commands/wiki/index.ts`
- 行数：12
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/wiki` 的职责见上文分类。

#### `wiki.tsx`

- 所在目录：`src/commands/wiki`
- 完整路径：`src/commands/wiki/wiki.tsx`
- 行数：155
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/commands/wiki` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/commands/workflows`
- 完整路径：`src/commands/workflows/index.ts`
- 行数：30
- 主要功能简介：斜杠命令或 CLI 命令实现。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/commands/workflows` 的职责见上文分类。

### 2.57 目录组 `src/commands.test.ts`（1 个文件，约 862 行）

该组位于仓库相对路径 `src/commands.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `commands.test.ts`

- 所在目录：`src`
- 完整路径：`src/commands.test.ts`
- 行数：862
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/commands.test.ts`。体量较大（约 862 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.58 目录组 `src/commands.ts`（1 个文件，约 825 行）

该组位于仓库相对路径 `src/commands.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `commands.ts`

- 所在目录：`src`
- 完整路径：`src/commands.ts`
- 行数：825
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：体量较大（约 825 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.59 目录组 `src/components`（482 个文件，约 107878 行）

该组位于仓库相对路径 `src/components`。Ink/React 终端 UI 组件。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `AgentProgressLine.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/AgentProgressLine.tsx`
- 行数：134
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `App.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/App.tsx`
- 行数：55
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ApproveApiKey.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ApproveApiKey.tsx`
- 行数：121
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AutoModeOptInDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/AutoModeOptInDialog.tsx`
- 行数：141
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AutoUpdater.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/AutoUpdater.test.ts`
- 行数：48
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/AutoUpdater.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `AutoUpdater.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/AutoUpdater.tsx`
- 行数：199
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AutoUpdaterWrapper.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/AutoUpdaterWrapper.tsx`
- 行数：94
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AwsAuthStatusBox.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/AwsAuthStatusBox.tsx`
- 行数：81
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `BaseTextInput.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BaseTextInput.tsx`
- 行数：137
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `BashModeProgress.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BashModeProgress.tsx`
- 行数：55
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `BridgeDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BridgeDialog.tsx`
- 行数：400
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `BuiltinStatusLine.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BuiltinStatusLine.test.tsx`
- 行数：214
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/BuiltinStatusLine.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `BuiltinStatusLine.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BuiltinStatusLine.tsx`
- 行数：244
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `BypassPermissionsModeDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/BypassPermissionsModeDialog.tsx`
- 行数：89
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ChannelDowngradeDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ChannelDowngradeDialog.tsx`
- 行数：100
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `PluginHintMenu.tsx`

- 所在目录：`src/components/ClaudeCodeHint`
- 完整路径：`src/components/ClaudeCodeHint/PluginHintMenu.tsx`
- 行数：77
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/ClaudeCodeHint` 的职责见上文分类。

#### `ClaudeInChromeOnboarding.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ClaudeInChromeOnboarding.tsx`
- 行数：120
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ClaudeMdExternalIncludesDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ClaudeMdExternalIncludesDialog.tsx`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ClickableImageRef.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ClickableImageRef.tsx`
- 行数：71
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `CompactProgressBar.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/CompactProgressBar.tsx`
- 行数：24
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `CompactSummary.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/CompactSummary.tsx`
- 行数：117
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ConfigurableShortcutHint.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ConfigurableShortcutHint.tsx`
- 行数：56
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ConsoleOAuthFlow.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ConsoleOAuthFlow.test.tsx`
- 行数：145
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/ConsoleOAuthFlow.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `ConsoleOAuthFlow.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ConsoleOAuthFlow.tsx`
- 行数：677
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ContextSuggestions.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ContextSuggestions.tsx`
- 行数：45
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ContextVisualization.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ContextVisualization.tsx`
- 行数：488
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `CoordinatorAgentStatus.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/CoordinatorAgentStatus.tsx`
- 行数：272
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `CostThresholdDialog.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/CostThresholdDialog.test.ts`
- 行数：20
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/CostThresholdDialog.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `CostThresholdDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/CostThresholdDialog.tsx`
- 行数：40
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `CostThresholdProviderLabel.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/CostThresholdProviderLabel.ts`
- 行数：16
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `CtrlOToExpand.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/CtrlOToExpand.tsx`
- 行数：50
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `SelectMulti.tsx`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/SelectMulti.tsx`
- 行数：212
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/index.ts`
- 行数：3
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `option-map.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/option-map.ts`
- 行数：50
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `select-input-option.tsx`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/select-input-option.tsx`
- 行数：489
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `select-option.tsx`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/select-option.tsx`
- 行数：67
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `select.tsx`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/select.tsx`
- 行数：707
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `use-multi-select-state.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/use-multi-select-state.ts`
- 行数：414
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `use-select-input.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/use-select-input.ts`
- 行数：287
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `use-select-navigation.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/use-select-navigation.ts`
- 行数：676
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `use-select-state.ts`

- 所在目录：`src/components/CustomSelect`
- 完整路径：`src/components/CustomSelect/use-select-state.ts`
- 行数：163
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/CustomSelect` 的职责见上文分类。

#### `DesktopHandoff.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/DesktopHandoff.tsx`
- 行数：192
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `DesktopUpsellStartup.tsx`

- 所在目录：`src/components/DesktopUpsell`
- 完整路径：`src/components/DesktopUpsell/DesktopUpsellStartup.tsx`
- 行数：170
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/DesktopUpsell` 的职责见上文分类。

#### `DevChannelsDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/DevChannelsDialog.tsx`
- 行数：103
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `DiagnosticsDisplay.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/DiagnosticsDisplay.tsx`
- 行数：94
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `EffortCallout.modelGate.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/EffortCallout.modelGate.test.ts`
- 行数：22
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/EffortCallout.modelGate.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `EffortCallout.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/EffortCallout.tsx`
- 行数：275
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `EffortIndicator.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/EffortIndicator.ts`
- 行数：45
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `EffortPicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/EffortPicker.tsx`
- 行数：159
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ExitFlow.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ExitFlow.tsx`
- 行数：47
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ExportDialog.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ExportDialog.test.tsx`
- 行数：697
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/ExportDialog.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `ExportDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ExportDialog.tsx`
- 行数：198
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FallbackToolUseErrorMessage.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FallbackToolUseErrorMessage.tsx`
- 行数：115
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FallbackToolUseRejectedMessage.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FallbackToolUseRejectedMessage.tsx`
- 行数：14
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `FastIcon.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FastIcon.tsx`
- 行数：44
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Feedback.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/Feedback.test.ts`
- 行数：44
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/Feedback.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `Feedback.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Feedback.tsx`
- 行数：563
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FeedbackSurvey.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/FeedbackSurvey.tsx`
- 行数：172
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `FeedbackSurveyView.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/FeedbackSurveyView.tsx`
- 行数：107
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `TranscriptSharePrompt.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/TranscriptSharePrompt.tsx`
- 行数：87
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `submitTranscriptShare.ts`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/submitTranscriptShare.ts`
- 行数：133
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `useDebouncedDigitInput.ts`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/useDebouncedDigitInput.ts`
- 行数：82
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `useFeedbackSurvey.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/useFeedbackSurvey.tsx`
- 行数：278
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `useMemorySurvey.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/useMemorySurvey.tsx`
- 行数：181
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `usePostCompactSurvey.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/usePostCompactSurvey.tsx`
- 行数：193
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `useSurveyState.tsx`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/useSurveyState.tsx`
- 行数：99
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/FeedbackSurvey`
- 完整路径：`src/components/FeedbackSurvey/utils.ts`
- 行数：3
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/FeedbackSurvey` 的职责见上文分类。

#### `FileEditToolDiff.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FileEditToolDiff.tsx`
- 行数：186
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FileEditToolUpdatedMessage.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FileEditToolUpdatedMessage.tsx`
- 行数：123
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FileEditToolUseRejectedMessage.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FileEditToolUseRejectedMessage.tsx`
- 行数：169
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FilePathLink.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FilePathLink.tsx`
- 行数：42
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `FullscreenLayout.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/FullscreenLayout.tsx`
- 行数：636
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `GlobalSearchDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/GlobalSearchDialog.tsx`
- 行数：353
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Commands.tsx`

- 所在目录：`src/components/HelpV2`
- 完整路径：`src/components/HelpV2/Commands.tsx`
- 行数：81
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/HelpV2` 的职责见上文分类。

#### `General.tsx`

- 所在目录：`src/components/HelpV2`
- 完整路径：`src/components/HelpV2/General.tsx`
- 行数：22
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/HelpV2` 的职责见上文分类。

#### `HelpV2.tsx`

- 所在目录：`src/components/HelpV2`
- 完整路径：`src/components/HelpV2/HelpV2.tsx`
- 行数：178
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/HelpV2` 的职责见上文分类。

#### `HighlightedCode.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/HighlightedCode.tsx`
- 行数：192
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Fallback.tsx`

- 所在目录：`src/components/HighlightedCode`
- 完整路径：`src/components/HighlightedCode/Fallback.tsx`
- 行数：195
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/HighlightedCode` 的职责见上文分类。

#### `HistorySearchDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/HistorySearchDialog.tsx`
- 行数：118
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `IdeAutoConnectDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/IdeAutoConnectDialog.tsx`
- 行数：153
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `IdeOnboardingDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/IdeOnboardingDialog.tsx`
- 行数：166
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `IdeStatusIndicator.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/IdeStatusIndicator.tsx`
- 行数：62
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `IdleReturnDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/IdleReturnDialog.tsx`
- 行数：117
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `InterruptedByUser.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/InterruptedByUser.tsx`
- 行数：14
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `InvalidConfigDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/InvalidConfigDialog.tsx`
- 行数：155
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `InvalidSettingsDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/InvalidSettingsDialog.tsx`
- 行数：88
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `KeybindingWarnings.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/KeybindingWarnings.tsx`
- 行数：54
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `LanguagePicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/LanguagePicker.tsx`
- 行数：85
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `LogSelector.resumeBranches.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/LogSelector.resumeBranches.test.ts`
- 行数：425
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/LogSelector.resumeBranches.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `LogSelector.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/LogSelector.tsx`
- 行数：1687
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1687 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `LogoPicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/LogoPicker.tsx`
- 行数：59
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AnimatedAsterisk.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/AnimatedAsterisk.tsx`
- 行数：51
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `AnimatedClawd.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/AnimatedClawd.tsx`
- 行数：122
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `ChannelsNotice.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/ChannelsNotice.tsx`
- 行数：265
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `Clawd.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/Clawd.tsx`
- 行数：239
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `CondensedLogo.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/CondensedLogo.tsx`
- 行数：162
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `EmergencyTip.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/EmergencyTip.tsx`
- 行数：57
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `Feed.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/Feed.tsx`
- 行数：111
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `FeedColumn.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/FeedColumn.tsx`
- 行数：58
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `GuestPassesUpsell.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/GuestPassesUpsell.tsx`
- 行数：68
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `LogoV2.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/LogoV2.tsx`
- 行数：555
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `Opus1mMergeNotice.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/Opus1mMergeNotice.tsx`
- 行数：53
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `OverageCreditUpsell.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/OverageCreditUpsell.tsx`
- 行数：165
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `VoiceModeNotice.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/VoiceModeNotice.tsx`
- 行数：66
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `WelcomeV2.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/WelcomeV2.tsx`
- 行数：432
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `WordmarkRow.test.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/WordmarkRow.test.tsx`
- 行数：55
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/LogoV2/WordmarkRow.test.tsx`。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `WordmarkRow.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/WordmarkRow.tsx`
- 行数：29
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `feedConfigs.tsx`

- 所在目录：`src/components/LogoV2`
- 完整路径：`src/components/LogoV2/feedConfigs.tsx`
- 行数：87
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LogoV2` 的职责见上文分类。

#### `LspRecommendationMenu.tsx`

- 所在目录：`src/components/LspRecommendation`
- 完整路径：`src/components/LspRecommendation/LspRecommendationMenu.tsx`
- 行数：87
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/LspRecommendation` 的职责见上文分类。

#### `MCPServerApprovalDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MCPServerApprovalDialog.tsx`
- 行数：114
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MCPServerDesktopImportDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MCPServerDesktopImportDialog.tsx`
- 行数：202
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MCPServerDialogCopy.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MCPServerDialogCopy.tsx`
- 行数：13
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `MCPServerMultiselectDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MCPServerMultiselectDialog.tsx`
- 行数：132
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ManagedSettingsSecurityDialog.tsx`

- 所在目录：`src/components/ManagedSettingsSecurityDialog`
- 完整路径：`src/components/ManagedSettingsSecurityDialog/ManagedSettingsSecurityDialog.tsx`
- 行数：148
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/ManagedSettingsSecurityDialog` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/ManagedSettingsSecurityDialog`
- 完整路径：`src/components/ManagedSettingsSecurityDialog/utils.ts`
- 行数：144
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/ManagedSettingsSecurityDialog` 的职责见上文分类。

#### `Markdown.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Markdown.tsx`
- 行数：235
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MarkdownTable.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MarkdownTable.tsx`
- 行数：321
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MemoryUsageIndicator.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MemoryUsageIndicator.tsx`
- 行数：4
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `Message.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Message.tsx`
- 行数：628
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MessageModel.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MessageModel.tsx`
- 行数：42
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MessageResponse.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MessageResponse.tsx`
- 行数：77
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MessageRow.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MessageRow.tsx`
- 行数：382
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MessageSelector.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MessageSelector.tsx`
- 行数：759
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `MessageTimestamp.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/MessageTimestamp.tsx`
- 行数：62
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Messages.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Messages.tsx`
- 行数：838
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 838 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `ModelPicker.switchProfile.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/ModelPicker.switchProfile.test.ts`
- 行数：276
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/ModelPicker.switchProfile.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `ModelPicker.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ModelPicker.test.tsx`
- 行数：338
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/ModelPicker.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `ModelPicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ModelPicker.tsx`
- 行数：567
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `NativeAutoUpdater.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/NativeAutoUpdater.test.ts`
- 行数：32
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/NativeAutoUpdater.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `NativeAutoUpdater.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/NativeAutoUpdater.tsx`
- 行数：181
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `NotebookEditToolUseRejectedMessage.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/NotebookEditToolUseRejectedMessage.tsx`
- 行数：91
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `OffscreenFreeze.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/OffscreenFreeze.tsx`
- 行数：43
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Onboarding.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Onboarding.tsx`
- 行数：244
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `OutputStylePicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/OutputStylePicker.tsx`
- 行数：112
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `PackageManagerAutoUpdater.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/PackageManagerAutoUpdater.tsx`
- 行数：107
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `PackageManagerUpdateGuidance.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/PackageManagerUpdateGuidance.test.tsx`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PackageManagerUpdateGuidance.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `Passes.tsx`

- 所在目录：`src/components/Passes`
- 完整路径：`src/components/Passes/Passes.tsx`
- 行数：183
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Passes` 的职责见上文分类。

#### `PrBadge.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/PrBadge.tsx`
- 行数：96
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `PressEnterToContinue.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/PressEnterToContinue.tsx`
- 行数：13
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `HistorySearchInput.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/HistorySearchInput.test.tsx`
- 行数：107
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/HistorySearchInput.test.tsx`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `HistorySearchInput.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/HistorySearchInput.tsx`
- 行数：53
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `IssueFlagBanner.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/IssueFlagBanner.tsx`
- 行数：7
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `KeepMounted.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/KeepMounted.test.tsx`
- 行数：98
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/KeepMounted.test.tsx`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `KeepMounted.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/KeepMounted.tsx`
- 行数：16
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `Notifications.effort.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/Notifications.effort.test.tsx`
- 行数：312
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/Notifications.effort.test.tsx`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `Notifications.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/Notifications.tsx`
- 行数：356
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInput.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInput.tsx`
- 行数：2419
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 2419 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooter.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooter.test.tsx`
- 行数：130
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/PromptInputFooter.test.tsx`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooter.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooter.tsx`
- 行数：263
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooterLeftSide.test.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooterLeftSide.test.ts`
- 行数：118
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/PromptInput/PromptInputFooterLeftSide.test.ts`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooterLeftSide.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooterLeftSide.tsx`
- 行数：532
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooterSuggestions.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooterSuggestions.test.tsx`
- 行数：35
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/PromptInputFooterSuggestions.test.tsx`。短文件，多为常量、再导出或薄包装。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputFooterSuggestions.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputFooterSuggestions.tsx`
- 行数：212
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputHelpMenu.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputHelpMenu.tsx`
- 行数：357
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputModeIndicator.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputModeIndicator.tsx`
- 行数：92
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputQueuedCommands.test.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputQueuedCommands.test.tsx`
- 行数：47
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/PromptInput/PromptInputQueuedCommands.test.tsx`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputQueuedCommands.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputQueuedCommands.tsx`
- 行数：130
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `PromptInputStashNotice.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/PromptInputStashNotice.tsx`
- 行数：24
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `SandboxPromptFooterHint.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/SandboxPromptFooterHint.tsx`
- 行数：63
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `ShimmeredInput.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/ShimmeredInput.tsx`
- 行数：142
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `VoiceIndicator.tsx`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/VoiceIndicator.tsx`
- 行数：136
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `footerVisibility.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/footerVisibility.ts`
- 行数：52
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `inputModes.test.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/inputModes.test.ts`
- 行数：104
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/PromptInput/inputModes.test.ts`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `inputModes.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/inputModes.ts`
- 行数：60
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `inputPaste.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/inputPaste.ts`
- 行数：90
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `useMaybeTruncateInput.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/useMaybeTruncateInput.ts`
- 行数：58
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `usePromptInputPlaceholder.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/usePromptInputPlaceholder.ts`
- 行数：76
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `useShowFastIconHint.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/useShowFastIconHint.ts`
- 行数：31
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `useSwarmBanner.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/useSwarmBanner.ts`
- 行数：155
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `utils.test.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/utils.test.ts`
- 行数：92
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/PromptInput/utils.test.ts`。所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/PromptInput`
- 完整路径：`src/components/PromptInput/utils.ts`
- 行数：128
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/PromptInput` 的职责见上文分类。

#### `ProviderManager.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ProviderManager.test.tsx`
- 行数：6304
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/ProviderManager.test.tsx`。体量较大（约 6304 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `ProviderManager.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ProviderManager.tsx`
- 行数：4536
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 4536 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `QuickOpenDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/QuickOpenDialog.tsx`
- 行数：243
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `RemoteCallout.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/RemoteCallout.tsx`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `RemoteEnvironmentDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/RemoteEnvironmentDialog.tsx`
- 行数：339
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ResumeCompactPrompt.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ResumeCompactPrompt.tsx`
- 行数：57
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ResumeTask.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ResumeTask.tsx`
- 行数：267
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `SandboxViolationExpandedView.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/SandboxViolationExpandedView.tsx`
- 行数：104
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ScrollKeybindingHandler.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ScrollKeybindingHandler.tsx`
- 行数：1011
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1011 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `SearchBox.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/SearchBox.tsx`
- 行数：71
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `SentryErrorBoundary.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/SentryErrorBoundary.ts`
- 行数：28
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `SessionBackgroundHint.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/SessionBackgroundHint.tsx`
- 行数：109
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `SessionPreview.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/SessionPreview.tsx`
- 行数：193
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `SessionSummary.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/SessionSummary.tsx`
- 行数：72
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ClinePassUsage.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/ClinePassUsage.tsx`
- 行数：219
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `CodexUsage.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/CodexUsage.tsx`
- 行数：211
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `Config.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/Config.tsx`
- 行数：2048
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 2048 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/Settings` 的职责见上文分类。

#### `MiniMaxUsage.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/MiniMaxUsage.tsx`
- 行数：249
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `Settings.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/Settings.tsx`
- 行数：143
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `Status.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/Status.tsx`
- 行数：240
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `UnsupportedUsage.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/UnsupportedUsage.tsx`
- 行数：28
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/Settings` 的职责见上文分类。

#### `Usage.tsx`

- 所在目录：`src/components/Settings`
- 完整路径：`src/components/Settings/Usage.tsx`
- 行数：408
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Settings` 的职责见上文分类。

#### `ShowInIDEPrompt.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ShowInIDEPrompt.tsx`
- 行数：177
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Spinner.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Spinner.tsx`
- 行数：554
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `CompletionFlash.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/CompletionFlash.tsx`
- 行数：73
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `FlashingChar.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/FlashingChar.tsx`
- 行数：60
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `GlimmerMessage.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/GlimmerMessage.tsx`
- 行数：327
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `ShimmerChar.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/ShimmerChar.tsx`
- 行数：35
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `SpinnerAnimationRow.test.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/SpinnerAnimationRow.test.tsx`
- 行数：1093
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/Spinner/SpinnerAnimationRow.test.tsx`。体量较大（约 1093 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `SpinnerAnimationRow.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/SpinnerAnimationRow.tsx`
- 行数：716
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `SpinnerGlyph.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/SpinnerGlyph.tsx`
- 行数：79
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `TeammateSpinnerLine.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/TeammateSpinnerLine.tsx`
- 行数：232
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `TeammateSpinnerTree.tsx`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/TeammateSpinnerTree.tsx`
- 行数：271
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/index.ts`
- 行数：10
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `teammateSelectHint.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/teammateSelectHint.ts`
- 行数：1
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/types.ts`
- 行数：13
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `useShimmerAnimation.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/useShimmerAnimation.ts`
- 行数：31
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/Spinner` 的职责见上文分类。

#### `useStalledAnimation.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/useStalledAnimation.ts`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/Spinner` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/Spinner`
- 完整路径：`src/components/Spinner/utils.ts`
- 行数：82
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/Spinner` 的职责见上文分类。

#### `StartupScreen.palettes.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/StartupScreen.palettes.test.ts`
- 行数：31
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/StartupScreen.palettes.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `StartupScreen.palettes.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/StartupScreen.palettes.ts`
- 行数：119
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `StartupScreen.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/StartupScreen.test.ts`
- 行数：453
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/StartupScreen.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `StartupScreen.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/StartupScreen.ts`
- 行数：269
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `Stats.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/Stats.tsx`
- 行数：1221
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1221 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `StatusLine.active.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/StatusLine.active.test.tsx`
- 行数：255
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/StatusLine.active.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `StatusLine.test.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/StatusLine.test.ts`
- 行数：131
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/StatusLine.test.ts`。所在目录 `src/components` 的职责见上文分类。

#### `StatusLine.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/StatusLine.tsx`
- 行数：437
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `StatusNotices.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/StatusNotices.tsx`
- 行数：120
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `StructuredDiff.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/StructuredDiff.tsx`
- 行数：189
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `Fallback.tsx`

- 所在目录：`src/components/StructuredDiff`
- 完整路径：`src/components/StructuredDiff/Fallback.tsx`
- 行数：488
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/StructuredDiff` 的职责见上文分类。

#### `colorDiff.ts`

- 所在目录：`src/components/StructuredDiff`
- 完整路径：`src/components/StructuredDiff/colorDiff.ts`
- 行数：37
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/StructuredDiff` 的职责见上文分类。

#### `StructuredDiffList.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/StructuredDiffList.tsx`
- 行数：29
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components` 的职责见上文分类。

#### `TagTabs.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TagTabs.tsx`
- 行数：138
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TaskListV2.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TaskListV2.tsx`
- 行数：378
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeammateViewHeader.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeammateViewHeader.tsx`
- 行数：81
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeleportError.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeleportError.tsx`
- 行数：188
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeleportProgress.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeleportProgress.tsx`
- 行数：139
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeleportRepoMismatchDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeleportRepoMismatchDialog.tsx`
- 行数：103
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeleportResumeWrapper.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeleportResumeWrapper.tsx`
- 行数：166
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TeleportStash.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TeleportStash.tsx`
- 行数：115
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TextInput.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TextInput.test.tsx`
- 行数：1405
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/TextInput.test.tsx`。体量较大（约 1405 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `TextInput.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TextInput.tsx`
- 行数：123
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ThemePicker.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ThemePicker.test.tsx`
- 行数：171
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/ThemePicker.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `ThemePicker.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ThemePicker.tsx`
- 行数：261
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ThinkingToggle.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ThinkingToggle.tsx`
- 行数：153
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TokenWarning.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/TokenWarning.tsx`
- 行数：178
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `ToolUseLoader.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ToolUseLoader.tsx`
- 行数：41
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `TrustDialog.tsx`

- 所在目录：`src/components/TrustDialog`
- 完整路径：`src/components/TrustDialog/TrustDialog.tsx`
- 行数：289
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/TrustDialog` 的职责见上文分类。

#### `utils.test.ts`

- 所在目录：`src/components/TrustDialog`
- 完整路径：`src/components/TrustDialog/utils.test.ts`
- 行数：292
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/TrustDialog/utils.test.ts`。所在目录 `src/components/TrustDialog` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/TrustDialog`
- 完整路径：`src/components/TrustDialog/utils.ts`
- 行数：253
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/TrustDialog` 的职责见上文分类。

#### `ValidationErrorsList.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/ValidationErrorsList.tsx`
- 行数：147
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `VimTextInput.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/VimTextInput.tsx`
- 行数：139
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `VirtualMessageList.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/VirtualMessageList.tsx`
- 行数：1081
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1081 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components` 的职责见上文分类。

#### `WorkflowMultiselectDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/WorkflowMultiselectDialog.tsx`
- 行数：127
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `WorktreeExitDialog.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/WorktreeExitDialog.tsx`
- 行数：230
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AgentDetail.test.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentDetail.test.tsx`
- 行数：139
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/AgentDetail.test.tsx`。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentDetail.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentDetail.tsx`
- 行数：239
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentDetailDialog.test.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentDetailDialog.test.tsx`
- 行数：141
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/AgentDetailDialog.test.tsx`。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentDetailDialog.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentDetailDialog.tsx`
- 行数：49
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentEditor.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentEditor.tsx`
- 行数：177
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentNavigationFooter.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentNavigationFooter.tsx`
- 行数：25
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentRouteSelector.test.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentRouteSelector.test.tsx`
- 行数：298
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/AgentRouteSelector.test.tsx`。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentRouteSelector.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentRouteSelector.tsx`
- 行数：116
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentsList.test.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentsList.test.tsx`
- 行数：192
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/AgentsList.test.tsx`。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentsList.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentsList.tsx`
- 行数：421
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentsMenu.test.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentsMenu.test.tsx`
- 行数：496
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/AgentsMenu.test.tsx`。所在目录 `src/components/agents` 的职责见上文分类。

#### `AgentsMenu.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/AgentsMenu.tsx`
- 行数：770
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `ColorPicker.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/ColorPicker.tsx`
- 行数：111
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `ModelSelector.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/ModelSelector.tsx`
- 行数：67
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `SnapshotUpdateDialog.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/SnapshotUpdateDialog.tsx`
- 行数：15
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/agents` 的职责见上文分类。

#### `ToolSelector.tsx`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/ToolSelector.tsx`
- 行数：561
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents` 的职责见上文分类。

#### `agentFileUtils.test.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/agentFileUtils.test.ts`
- 行数：46
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/agents/agentFileUtils.test.ts`。所在目录 `src/components/agents` 的职责见上文分类。

#### `agentFileUtils.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/agentFileUtils.ts`
- 行数：286
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/agents` 的职责见上文分类。

#### `generateAgent.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/generateAgent.ts`
- 行数：206
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/agents` 的职责见上文分类。

#### `CreateAgentWizard.tsx`

- 所在目录：`src/components/agents/new-agent-creation`
- 完整路径：`src/components/agents/new-agent-creation/CreateAgentWizard.tsx`
- 行数：96
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/components/agents/new-agent-creation`
- 完整路径：`src/components/agents/new-agent-creation/types.ts`
- 行数：27
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/agents/new-agent-creation` 的职责见上文分类。

#### `ColorStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/ColorStep.tsx`
- 行数：87
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `ConfirmStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/ConfirmStep.tsx`
- 行数：381
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `ConfirmStepWrapper.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/ConfirmStepWrapper.tsx`
- 行数：73
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `DescriptionStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/DescriptionStep.tsx`
- 行数：123
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `GenerateStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/GenerateStep.tsx`
- 行数：142
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `LocationStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/LocationStep.tsx`
- 行数：74
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `MemoryStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/MemoryStep.tsx`
- 行数：113
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `MethodStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/MethodStep.tsx`
- 行数：80
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `ModelStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/ModelStep.tsx`
- 行数：51
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `PromptStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/PromptStep.tsx`
- 行数：127
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `ToolsStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/ToolsStep.tsx`
- 行数：60
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `TypeStep.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/TypeStep.tsx`
- 行数：102
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `wizardSteps.test.tsx`

- 所在目录：`src/components/agents/new-agent-creation/wizard-steps`
- 完整路径：`src/components/agents/new-agent-creation/wizard-steps/wizardSteps.test.tsx`
- 行数：254
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/agents/new-agent-creation/wizard-steps/wizardSteps.test.tsx`。所在目录 `src/components/agents/new-agent-creation/wizard-steps` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/types.ts`
- 行数：27
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/agents` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/utils.ts`
- 行数：21
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/agents` 的职责见上文分类。

#### `validateAgent.ts`

- 所在目录：`src/components/agents`
- 完整路径：`src/components/agents/validateAgent.ts`
- 行数：109
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/agents` 的职责见上文分类。

#### `Byline.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Byline.tsx`
- 行数：76
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `Dialog.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Dialog.tsx`
- 行数：72
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `Divider.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Divider.tsx`
- 行数：148
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `FullWidthRow.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/FullWidthRow.tsx`
- 行数：15
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/design-system` 的职责见上文分类。

#### `FuzzyPicker.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/FuzzyPicker.tsx`
- 行数：313
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `KeyboardShortcutHint.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/KeyboardShortcutHint.tsx`
- 行数：80
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ListItem.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ListItem.tsx`
- 行数：243
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `LoadingState.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/LoadingState.tsx`
- 行数：93
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `Pane.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Pane.tsx`
- 行数：76
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ProgressBar.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ProgressBar.tsx`
- 行数：85
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `Ratchet.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Ratchet.tsx`
- 行数：79
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `StatusIcon.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/StatusIcon.tsx`
- 行数：94
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `Tabs.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/Tabs.tsx`
- 行数：339
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ThemeProvider.test.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ThemeProvider.test.tsx`
- 行数：229
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/design-system/ThemeProvider.test.tsx`。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ThemeProvider.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ThemeProvider.tsx`
- 行数：140
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ThemedBox.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ThemedBox.tsx`
- 行数：152
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `ThemedText.tsx`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/ThemedText.tsx`
- 行数：123
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/design-system` 的职责见上文分类。

#### `color.ts`

- 所在目录：`src/components/design-system`
- 完整路径：`src/components/design-system/color.ts`
- 行数：30
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/design-system` 的职责见上文分类。

#### `DiffDetailView.tsx`

- 所在目录：`src/components/diff`
- 完整路径：`src/components/diff/DiffDetailView.tsx`
- 行数：280
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/diff` 的职责见上文分类。

#### `DiffDialog.tsx`

- 所在目录：`src/components/diff`
- 完整路径：`src/components/diff/DiffDialog.tsx`
- 行数：382
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/diff` 的职责见上文分类。

#### `DiffFileList.tsx`

- 所在目录：`src/components/diff`
- 完整路径：`src/components/diff/DiffFileList.tsx`
- 行数：291
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/diff` 的职责见上文分类。

#### `Grove.tsx`

- 所在目录：`src/components/grove`
- 完整路径：`src/components/grove/Grove.tsx`
- 行数：462
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/grove` 的职责见上文分类。

#### `HooksConfigMenu.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/HooksConfigMenu.tsx`
- 行数：582
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `PromptDialog.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/PromptDialog.tsx`
- 行数：89
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `SelectEventMode.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/SelectEventMode.tsx`
- 行数：137
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `SelectHookMode.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/SelectHookMode.tsx`
- 行数：112
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `SelectMatcherMode.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/SelectMatcherMode.tsx`
- 行数：144
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `ViewHookMode.tsx`

- 所在目录：`src/components/hooks`
- 完整路径：`src/components/hooks/ViewHookMode.tsx`
- 行数：199
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/hooks` 的职责见上文分类。

#### `CapabilitiesSection.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/CapabilitiesSection.tsx`
- 行数：60
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `ElicitationDialog.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/ElicitationDialog.tsx`
- 行数：1168
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1168 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPAgentServerMenu.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPAgentServerMenu.tsx`
- 行数：182
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPListPanel.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPListPanel.tsx`
- 行数：503
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPReconnect.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPReconnect.tsx`
- 行数：166
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPRemoteServerMenu.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPRemoteServerMenu.tsx`
- 行数：648
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPSettings.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPSettings.tsx`
- 行数：397
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPStdioServerMenu.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPStdioServerMenu.tsx`
- 行数：176
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPToolDetailView.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPToolDetailView.tsx`
- 行数：211
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `MCPToolListView.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/MCPToolListView.tsx`
- 行数：140
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `McpParsingWarnings.tsx`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/McpParsingWarnings.tsx`
- 行数：212
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/index.ts`
- 行数：9
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/components/mcp` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/components/mcp`
- 完整路径：`src/components/mcp/types.ts`
- 行数：76
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/mcp` 的职责见上文分类。

#### `reconnectHelpers.tsx`

- 所在目录：`src/components/mcp/utils`
- 完整路径：`src/components/mcp/utils/reconnectHelpers.tsx`
- 行数：48
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/mcp/utils` 的职责见上文分类。

#### `MemoryFileSelector.tsx`

- 所在目录：`src/components/memory`
- 完整路径：`src/components/memory/MemoryFileSelector.tsx`
- 行数：440
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/memory` 的职责见上文分类。

#### `MemoryUpdateNotification.tsx`

- 所在目录：`src/components/memory`
- 完整路径：`src/components/memory/MemoryUpdateNotification.tsx`
- 行数：44
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/memory` 的职责见上文分类。

#### `memoryFileSelectorPaths.test.ts`

- 所在目录：`src/components/memory`
- 完整路径：`src/components/memory/memoryFileSelectorPaths.test.ts`
- 行数：72
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/memory/memoryFileSelectorPaths.test.ts`。所在目录 `src/components/memory` 的职责见上文分类。

#### `memoryFileSelectorPaths.ts`

- 所在目录：`src/components/memory`
- 完整路径：`src/components/memory/memoryFileSelectorPaths.ts`
- 行数：34
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/memory` 的职责见上文分类。

#### `messageActions.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/messageActions.tsx`
- 行数：451
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components` 的职责见上文分类。

#### `AdvisorMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AdvisorMessage.tsx`
- 行数：157
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `AssistantRedactedThinkingMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AssistantRedactedThinkingMessage.tsx`
- 行数：30
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `AssistantTextMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AssistantTextMessage.tsx`
- 行数：269
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `AssistantThinkingMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AssistantThinkingMessage.tsx`
- 行数：85
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `AssistantToolUseMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AssistantToolUseMessage.tsx`
- 行数：367
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `AttachmentMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/AttachmentMessage.tsx`
- 行数：536
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `CollapsedReadSearchContent.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/CollapsedReadSearchContent.tsx`
- 行数：483
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `CompactBoundaryMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/CompactBoundaryMessage.tsx`
- 行数：16
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `GroupedToolUseContent.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/GroupedToolUseContent.tsx`
- 行数：57
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `HighlightedThinkingText.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/HighlightedThinkingText.tsx`
- 行数：161
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `HookProgressMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/HookProgressMessage.tsx`
- 行数：115
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `PlanApprovalMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/PlanApprovalMessage.tsx`
- 行数：221
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `RateLimitMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/RateLimitMessage.tsx`
- 行数：160
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `ShutdownMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/ShutdownMessage.tsx`
- 行数：131
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `SnipBoundaryMessage.test.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/SnipBoundaryMessage.test.tsx`
- 行数：38
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/messages/SnipBoundaryMessage.test.tsx`。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `SnipBoundaryMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/SnipBoundaryMessage.tsx`
- 行数：26
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `SystemAPIErrorMessage.test.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/SystemAPIErrorMessage.test.tsx`
- 行数：107
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/messages/SystemAPIErrorMessage.test.tsx`。所在目录 `src/components/messages` 的职责见上文分类。

#### `SystemAPIErrorMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/SystemAPIErrorMessage.tsx`
- 行数：66
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `SystemTextMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/SystemTextMessage.tsx`
- 行数：829
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 829 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/messages` 的职责见上文分类。

#### `TaskAssignmentMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/TaskAssignmentMessage.tsx`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserAgentNotificationMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserAgentNotificationMessage.tsx`
- 行数：82
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserBashInputMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserBashInputMessage.tsx`
- 行数：57
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserBashOutputMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserBashOutputMessage.tsx`
- 行数：54
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserChannelMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserChannelMessage.tsx`
- 行数：136
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserCommandMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserCommandMessage.tsx`
- 行数：107
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserCrossSessionMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserCrossSessionMessage.tsx`
- 行数：21
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserForkBoilerplateMessage.test.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserForkBoilerplateMessage.test.tsx`
- 行数：48
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/messages/UserForkBoilerplateMessage.test.tsx`。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserForkBoilerplateMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserForkBoilerplateMessage.tsx`
- 行数：28
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserGitHubWebhookMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserGitHubWebhookMessage.tsx`
- 行数：21
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserImageMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserImageMessage.tsx`
- 行数：58
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserLocalCommandOutputMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserLocalCommandOutputMessage.tsx`
- 行数：167
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserMemoryInputMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserMemoryInputMessage.tsx`
- 行数：74
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserPlanMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserPlanMessage.tsx`
- 行数：41
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserPromptMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserPromptMessage.tsx`
- 行数：79
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserResourceUpdateMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserResourceUpdateMessage.tsx`
- 行数：120
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserTeammateMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserTeammateMessage.tsx`
- 行数：205
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `UserTextMessage.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/UserTextMessage.tsx`
- 行数：274
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `RejectedPlanMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/RejectedPlanMessage.tsx`
- 行数：30
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `RejectedToolUseMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/RejectedToolUseMessage.tsx`
- 行数：14
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `UserToolCanceledMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/UserToolCanceledMessage.tsx`
- 行数：14
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `UserToolErrorMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/UserToolErrorMessage.tsx`
- 行数：102
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `UserToolRejectMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/UserToolRejectMessage.tsx`
- 行数：94
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `UserToolResultMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/UserToolResultMessage.tsx`
- 行数：105
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `UserToolSuccessMessage.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx`
- 行数：146
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `utils.tsx`

- 所在目录：`src/components/messages/UserToolResultMessage`
- 完整路径：`src/components/messages/UserToolResultMessage/utils.tsx`
- 行数：43
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages/UserToolResultMessage` 的职责见上文分类。

#### `nullRenderingAttachments.ts`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/nullRenderingAttachments.ts`
- 行数：78
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/messages` 的职责见上文分类。

#### `teamMemCollapsed.tsx`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/teamMemCollapsed.tsx`
- 行数：139
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/messages` 的职责见上文分类。

#### `teamMemSaved.ts`

- 所在目录：`src/components/messages`
- 完整路径：`src/components/messages/teamMemSaved.ts`
- 行数：19
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/messages` 的职责见上文分类。

#### `AskUserQuestionPermissionRequest.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/AskUserQuestionPermissionRequest.tsx`
- 行数：644
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `PreviewBox.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/PreviewBox.tsx`
- 行数：228
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `PreviewQuestionView.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/PreviewQuestionView.tsx`
- 行数：327
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `QuestionNavigationBar.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/QuestionNavigationBar.tsx`
- 行数：177
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `QuestionView.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/QuestionView.tsx`
- 行数：459
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `SubmitQuestionsView.tsx`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/SubmitQuestionsView.tsx`
- 行数：143
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `use-multiple-choice-state.ts`

- 所在目录：`src/components/permissions/AskUserQuestionPermissionRequest`
- 完整路径：`src/components/permissions/AskUserQuestionPermissionRequest/use-multiple-choice-state.ts`
- 行数：179
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions/AskUserQuestionPermissionRequest` 的职责见上文分类。

#### `BashPermissionRequest.tsx`

- 所在目录：`src/components/permissions/BashPermissionRequest`
- 完整路径：`src/components/permissions/BashPermissionRequest/BashPermissionRequest.tsx`
- 行数：383
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/BashPermissionRequest` 的职责见上文分类。

#### `ComputerUseApproval.tsx`

- 所在目录：`src/components/permissions/ComputerUseApproval`
- 完整路径：`src/components/permissions/ComputerUseApproval/ComputerUseApproval.tsx`
- 行数：441
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/ComputerUseApproval` 的职责见上文分类。

#### `EnterPlanModePermissionRequest.tsx`

- 所在目录：`src/components/permissions/EnterPlanModePermissionRequest`
- 完整路径：`src/components/permissions/EnterPlanModePermissionRequest/EnterPlanModePermissionRequest.tsx`
- 行数：122
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/EnterPlanModePermissionRequest` 的职责见上文分类。

#### `ExitPlanModePermissionRequest.render.test.tsx`

- 所在目录：`src/components/permissions/ExitPlanModePermissionRequest`
- 完整路径：`src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.render.test.tsx`
- 行数：229
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.render.test.tsx`。所在目录 `src/components/permissions/ExitPlanModePermissionRequest` 的职责见上文分类。

#### `ExitPlanModePermissionRequest.test.ts`

- 所在目录：`src/components/permissions/ExitPlanModePermissionRequest`
- 完整路径：`src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.test.ts`
- 行数：143
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.test.ts`。所在目录 `src/components/permissions/ExitPlanModePermissionRequest` 的职责见上文分类。

#### `ExitPlanModePermissionRequest.tsx`

- 所在目录：`src/components/permissions/ExitPlanModePermissionRequest`
- 完整路径：`src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.tsx`
- 行数：904
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 904 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/permissions/ExitPlanModePermissionRequest` 的职责见上文分类。

#### `FallbackPermissionRequest.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/FallbackPermissionRequest.tsx`
- 行数：152
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `FileEditPermissionRequest.tsx`

- 所在目录：`src/components/permissions/FileEditPermissionRequest`
- 完整路径：`src/components/permissions/FileEditPermissionRequest/FileEditPermissionRequest.tsx`
- 行数：181
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FileEditPermissionRequest` 的职责见上文分类。

#### `FilePermissionDialog.tsx`

- 所在目录：`src/components/permissions/FilePermissionDialog`
- 完整路径：`src/components/permissions/FilePermissionDialog/FilePermissionDialog.tsx`
- 行数：226
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FilePermissionDialog` 的职责见上文分类。

#### `ideDiffConfig.ts`

- 所在目录：`src/components/permissions/FilePermissionDialog`
- 完整路径：`src/components/permissions/FilePermissionDialog/ideDiffConfig.ts`
- 行数：42
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions/FilePermissionDialog` 的职责见上文分类。

#### `permissionOptions.tsx`

- 所在目录：`src/components/permissions/FilePermissionDialog`
- 完整路径：`src/components/permissions/FilePermissionDialog/permissionOptions.tsx`
- 行数：203
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FilePermissionDialog` 的职责见上文分类。

#### `useFilePermissionDialog.ts`

- 所在目录：`src/components/permissions/FilePermissionDialog`
- 完整路径：`src/components/permissions/FilePermissionDialog/useFilePermissionDialog.ts`
- 行数：207
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions/FilePermissionDialog` 的职责见上文分类。

#### `usePermissionHandler.ts`

- 所在目录：`src/components/permissions/FilePermissionDialog`
- 完整路径：`src/components/permissions/FilePermissionDialog/usePermissionHandler.ts`
- 行数：167
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions/FilePermissionDialog` 的职责见上文分类。

#### `FileWritePermissionRequest.tsx`

- 所在目录：`src/components/permissions/FileWritePermissionRequest`
- 完整路径：`src/components/permissions/FileWritePermissionRequest/FileWritePermissionRequest.tsx`
- 行数：160
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FileWritePermissionRequest` 的职责见上文分类。

#### `FileWriteToolDiff.tsx`

- 所在目录：`src/components/permissions/FileWritePermissionRequest`
- 完整路径：`src/components/permissions/FileWritePermissionRequest/FileWriteToolDiff.tsx`
- 行数：88
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FileWritePermissionRequest` 的职责见上文分类。

#### `FilesystemPermissionRequest.tsx`

- 所在目录：`src/components/permissions/FilesystemPermissionRequest`
- 完整路径：`src/components/permissions/FilesystemPermissionRequest/FilesystemPermissionRequest.tsx`
- 行数：114
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/FilesystemPermissionRequest` 的职责见上文分类。

#### `MonitorPermissionRequest.test.tsx`

- 所在目录：`src/components/permissions/MonitorPermissionRequest`
- 完整路径：`src/components/permissions/MonitorPermissionRequest/MonitorPermissionRequest.test.tsx`
- 行数：537
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/permissions/MonitorPermissionRequest/MonitorPermissionRequest.test.tsx`。所在目录 `src/components/permissions/MonitorPermissionRequest` 的职责见上文分类。

#### `MonitorPermissionRequest.tsx`

- 所在目录：`src/components/permissions/MonitorPermissionRequest`
- 完整路径：`src/components/permissions/MonitorPermissionRequest/MonitorPermissionRequest.tsx`
- 行数：134
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/MonitorPermissionRequest` 的职责见上文分类。

#### `NotebookEditPermissionRequest.tsx`

- 所在目录：`src/components/permissions/NotebookEditPermissionRequest`
- 完整路径：`src/components/permissions/NotebookEditPermissionRequest/NotebookEditPermissionRequest.tsx`
- 行数：165
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/NotebookEditPermissionRequest` 的职责见上文分类。

#### `NotebookEditToolDiff.tsx`

- 所在目录：`src/components/permissions/NotebookEditPermissionRequest`
- 完整路径：`src/components/permissions/NotebookEditPermissionRequest/NotebookEditToolDiff.tsx`
- 行数：234
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/NotebookEditPermissionRequest` 的职责见上文分类。

#### `PermissionDecisionDebugInfo.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionDecisionDebugInfo.tsx`
- 行数：461
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionDialog.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionDialog.tsx`
- 行数：71
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionExplanation.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionExplanation.tsx`
- 行数：273
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionPrompt.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionPrompt.tsx`
- 行数：308
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionRequest.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionRequest.tsx`
- 行数：217
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionRequestTitle.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionRequestTitle.tsx`
- 行数：65
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionRuleExplanation.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionRuleExplanation.tsx`
- 行数：120
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PermissionScaffold.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/PermissionScaffold.tsx`
- 行数：55
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `PowerShellPermissionRequest.tsx`

- 所在目录：`src/components/permissions/PowerShellPermissionRequest`
- 完整路径：`src/components/permissions/PowerShellPermissionRequest/PowerShellPermissionRequest.tsx`
- 行数：193
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/PowerShellPermissionRequest` 的职责见上文分类。

#### `ReviewArtifactPermissionRequest.tsx`

- 所在目录：`src/components/permissions/ReviewArtifactPermissionRequest`
- 完整路径：`src/components/permissions/ReviewArtifactPermissionRequest/ReviewArtifactPermissionRequest.tsx`
- 行数：11
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/permissions/ReviewArtifactPermissionRequest` 的职责见上文分类。

#### `SandboxPermissionRequest.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/SandboxPermissionRequest.tsx`
- 行数：163
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `SedEditPermissionRequest.tsx`

- 所在目录：`src/components/permissions/SedEditPermissionRequest`
- 完整路径：`src/components/permissions/SedEditPermissionRequest/SedEditPermissionRequest.tsx`
- 行数：229
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/SedEditPermissionRequest` 的职责见上文分类。

#### `SharedShellPermissionRequest.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/SharedShellPermissionRequest.tsx`
- 行数：163
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `SkillPermissionRequest.tsx`

- 所在目录：`src/components/permissions/SkillPermissionRequest`
- 完整路径：`src/components/permissions/SkillPermissionRequest/SkillPermissionRequest.tsx`
- 行数：187
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/SkillPermissionRequest` 的职责见上文分类。

#### `WebFetchPermissionRequest.tsx`

- 所在目录：`src/components/permissions/WebFetchPermissionRequest`
- 完整路径：`src/components/permissions/WebFetchPermissionRequest/WebFetchPermissionRequest.tsx`
- 行数：140
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/WebFetchPermissionRequest` 的职责见上文分类。

#### `WorkerBadge.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/WorkerBadge.tsx`
- 行数：48
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `WorkerPendingPermission.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/WorkerPendingPermission.tsx`
- 行数：104
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `baseShellToolUseOptions.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/baseShellToolUseOptions.tsx`
- 行数：450
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `hooks.ts`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/hooks.ts`
- 行数：209
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions` 的职责见上文分类。

#### `AddPermissionRules.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/AddPermissionRules.tsx`
- 行数：180
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `AddWorkspaceDirectory.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/AddWorkspaceDirectory.tsx`
- 行数：340
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `PermissionModeTab.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/PermissionModeTab.tsx`
- 行数：61
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `PermissionRuleDescription.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/PermissionRuleDescription.tsx`
- 行数：75
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `PermissionRuleInput.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/PermissionRuleInput.tsx`
- 行数：137
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `PermissionRuleList.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/PermissionRuleList.tsx`
- 行数：1258
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1258 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `RecentDenialsTab.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/RecentDenialsTab.tsx`
- 行数：206
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `RemoveWorkspaceDirectory.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/RemoveWorkspaceDirectory.tsx`
- 行数：110
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `WorkspaceTab.tsx`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/WorkspaceTab.tsx`
- 行数：149
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `permissionModeOptions.test.ts`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/permissionModeOptions.test.ts`
- 行数：37
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：运行方式：`bun test ./src/components/permissions/rules/permissionModeOptions.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `permissionModeOptions.ts`

- 所在目录：`src/components/permissions/rules`
- 完整路径：`src/components/permissions/rules/permissionModeOptions.ts`
- 行数：64
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions/rules` 的职责见上文分类。

#### `simplePermissionActions.ts`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/simplePermissionActions.ts`
- 行数：128
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions` 的职责见上文分类。

#### `useDangerousModeConfirmation.test.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/useDangerousModeConfirmation.test.tsx`
- 行数：217
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/permissions/useDangerousModeConfirmation.test.tsx`。所在目录 `src/components/permissions` 的职责见上文分类。

#### `useDangerousModeConfirmation.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/useDangerousModeConfirmation.tsx`
- 行数：69
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `usePermissionModeChangeRequest.tsx`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/usePermissionModeChangeRequest.tsx`
- 行数：61
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/permissions` 的职责见上文分类。

#### `useShellPermissionFeedback.ts`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/useShellPermissionFeedback.ts`
- 行数：155
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components/permissions` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/components/permissions`
- 完整路径：`src/components/permissions/utils.ts`
- 行数：25
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/permissions` 的职责见上文分类。

#### `providerManagerAimlapi.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/providerManagerAimlapi.ts`
- 行数：101
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `SandboxConfigTab.tsx`

- 所在目录：`src/components/sandbox`
- 完整路径：`src/components/sandbox/SandboxConfigTab.tsx`
- 行数：44
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/sandbox` 的职责见上文分类。

#### `SandboxDependenciesTab.tsx`

- 所在目录：`src/components/sandbox`
- 完整路径：`src/components/sandbox/SandboxDependenciesTab.tsx`
- 行数：119
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/sandbox` 的职责见上文分类。

#### `SandboxDoctorSection.tsx`

- 所在目录：`src/components/sandbox`
- 完整路径：`src/components/sandbox/SandboxDoctorSection.tsx`
- 行数：45
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/sandbox` 的职责见上文分类。

#### `SandboxOverridesTab.tsx`

- 所在目录：`src/components/sandbox`
- 完整路径：`src/components/sandbox/SandboxOverridesTab.tsx`
- 行数：192
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/sandbox` 的职责见上文分类。

#### `SandboxSettings.tsx`

- 所在目录：`src/components/sandbox`
- 完整路径：`src/components/sandbox/SandboxSettings.tsx`
- 行数：295
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/sandbox` 的职责见上文分类。

#### `ExpandShellOutputContext.tsx`

- 所在目录：`src/components/shell`
- 完整路径：`src/components/shell/ExpandShellOutputContext.tsx`
- 行数：35
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/shell` 的职责见上文分类。

#### `OutputLine.tsx`

- 所在目录：`src/components/shell`
- 完整路径：`src/components/shell/OutputLine.tsx`
- 行数：117
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/shell` 的职责见上文分类。

#### `ShellProgressMessage.tsx`

- 所在目录：`src/components/shell`
- 完整路径：`src/components/shell/ShellProgressMessage.tsx`
- 行数：149
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/shell` 的职责见上文分类。

#### `ShellTimeDisplay.tsx`

- 所在目录：`src/components/shell`
- 完整路径：`src/components/shell/ShellTimeDisplay.tsx`
- 行数：73
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/shell` 的职责见上文分类。

#### `SkillsMenu.tsx`

- 所在目录：`src/components/skills`
- 完整路径：`src/components/skills/SkillsMenu.tsx`
- 行数：245
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/skills` 的职责见上文分类。

#### `AsyncAgentDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/AsyncAgentDetailDialog.tsx`
- 行数：228
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `BackgroundTask.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/BackgroundTask.tsx`
- 行数：344
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `BackgroundTaskStatus.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/BackgroundTaskStatus.tsx`
- 行数：428
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `BackgroundTasksDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/BackgroundTasksDialog.tsx`
- 行数：652
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `DreamDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/DreamDetailDialog.tsx`
- 行数：250
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `InProcessTeammateDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/InProcessTeammateDetailDialog.tsx`
- 行数：265
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `MonitorMcpDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/MonitorMcpDetailDialog.tsx`
- 行数：22
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/tasks` 的职责见上文分类。

#### `RemoteSessionDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/RemoteSessionDetailDialog.tsx`
- 行数：904
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 904 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/components/tasks` 的职责见上文分类。

#### `RemoteSessionProgress.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/RemoteSessionProgress.tsx`
- 行数：241
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `ShellDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/ShellDetailDialog.tsx`
- 行数：403
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `ShellProgress.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/ShellProgress.tsx`
- 行数：86
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `WorkflowDetailDialog.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/WorkflowDetailDialog.tsx`
- 行数：31
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/tasks` 的职责见上文分类。

#### `renderToolActivity.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/renderToolActivity.tsx`
- 行数：32
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/tasks` 的职责见上文分类。

#### `taskStatusUtils.test.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/taskStatusUtils.test.tsx`
- 行数：61
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/tasks/taskStatusUtils.test.tsx`。所在目录 `src/components/tasks` 的职责见上文分类。

#### `taskStatusUtils.tsx`

- 所在目录：`src/components/tasks`
- 完整路径：`src/components/tasks/taskStatusUtils.tsx`
- 行数：116
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/tasks` 的职责见上文分类。

#### `TeamStatus.tsx`

- 所在目录：`src/components/teams`
- 完整路径：`src/components/teams/TeamStatus.tsx`
- 行数：79
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/teams` 的职责见上文分类。

#### `TeamsDialog.tsx`

- 所在目录：`src/components/teams`
- 完整路径：`src/components/teams/TeamsDialog.tsx`
- 行数：775
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/teams` 的职责见上文分类。

#### `OrderedList.tsx`

- 所在目录：`src/components/ui`
- 完整路径：`src/components/ui/OrderedList.tsx`
- 行数：70
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/ui` 的职责见上文分类。

#### `OrderedListItem.tsx`

- 所在目录：`src/components/ui`
- 完整路径：`src/components/ui/OrderedListItem.tsx`
- 行数：44
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/ui` 的职责见上文分类。

#### `TreeSelect.tsx`

- 所在目录：`src/components/ui`
- 完整路径：`src/components/ui/TreeSelect.tsx`
- 行数：396
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/ui` 的职责见上文分类。

#### `useCodexOAuthFlow.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/useCodexOAuthFlow.test.tsx`
- 行数：486
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/useCodexOAuthFlow.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `useCodexOAuthFlow.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/useCodexOAuthFlow.ts`
- 行数：141
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `useXaiOAuthFlow.test.tsx`

- 所在目录：`src/components`
- 完整路径：`src/components/useXaiOAuthFlow.test.tsx`
- 行数：166
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/components/useXaiOAuthFlow.test.tsx`。所在目录 `src/components` 的职责见上文分类。

#### `useXaiOAuthFlow.ts`

- 所在目录：`src/components`
- 完整路径：`src/components/useXaiOAuthFlow.ts`
- 行数：147
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：所在目录 `src/components` 的职责见上文分类。

#### `WizardDialogLayout.tsx`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/WizardDialogLayout.tsx`
- 行数：64
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/wizard` 的职责见上文分类。

#### `WizardNavigationFooter.tsx`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/WizardNavigationFooter.tsx`
- 行数：23
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/components/wizard` 的职责见上文分类。

#### `WizardProvider.tsx`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/WizardProvider.tsx`
- 行数：212
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/components/wizard` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/index.ts`
- 行数：9
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/components/wizard` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/types.ts`
- 行数：31
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/wizard` 的职责见上文分类。

#### `useWizard.ts`

- 所在目录：`src/components/wizard`
- 完整路径：`src/components/wizard/useWizard.ts`
- 行数：13
- 主要功能简介：Ink/React 终端 UI 组件。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/components/wizard` 的职责见上文分类。

### 2.60 目录组 `src/constants`（27 个文件，约 3247 行）

该组位于仓库相对路径 `src/constants`。产品常量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `apiLimits.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/apiLimits.ts`
- 行数：117
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `betas.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/betas.ts`
- 行数：53
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `brand.test.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/brand.test.ts`
- 行数：44
- 主要功能简介：产品常量。
- 说明：运行方式：`bun test ./src/constants/brand.test.ts`。所在目录 `src/constants` 的职责见上文分类。

#### `brand.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/brand.ts`
- 行数：40
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `common.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/common.ts`
- 行数：33
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `cyberRiskInstruction.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/cyberRiskInstruction.ts`
- 行数：17
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `errorIds.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/errorIds.ts`
- 行数：15
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `figures.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/figures.ts`
- 行数：48
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `files.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/files.ts`
- 行数：156
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `github-app.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/github-app.ts`
- 行数：144
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `messages.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/messages.ts`
- 行数：1
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `oauth.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/oauth.ts`
- 行数：234
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `outputStyles.protoName.test.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/outputStyles.protoName.test.ts`
- 行数：48
- 主要功能简介：产品常量。
- 说明：运行方式：`bun test ./src/constants/outputStyles.protoName.test.ts`。所在目录 `src/constants` 的职责见上文分类。

#### `outputStyles.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/outputStyles.ts`
- 行数：236
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `product.test.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/product.test.ts`
- 行数：118
- 主要功能简介：产品常量。
- 说明：运行方式：`bun test ./src/constants/product.test.ts`。所在目录 `src/constants` 的职责见上文分类。

#### `product.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/product.ts`
- 行数：134
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `promptIdentity.test.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/promptIdentity.test.ts`
- 行数：209
- 主要功能简介：产品常量。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/constants/promptIdentity.test.ts`。所在目录 `src/constants` 的职责见上文分类。

#### `prompts.doingTasks.test.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/prompts.doingTasks.test.ts`
- 行数：35
- 主要功能简介：产品常量。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/constants/prompts.doingTasks.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `prompts.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/prompts.ts`
- 行数：918
- 主要功能简介：产品常量。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。体量较大（约 918 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/constants` 的职责见上文分类。

#### `querySource.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/querySource.ts`
- 行数：7
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `spinnerVerbs.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/spinnerVerbs.ts`
- 行数：204
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `system.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/system.ts`
- 行数：104
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `systemPromptSections.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/systemPromptSections.ts`
- 行数：68
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `toolLimits.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/toolLimits.ts`
- 行数：56
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `tools.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/tools.ts`
- 行数：110
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

#### `turnCompletionVerbs.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/turnCompletionVerbs.ts`
- 行数：12
- 主要功能简介：产品常量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/constants` 的职责见上文分类。

#### `xml.ts`

- 所在目录：`src/constants`
- 完整路径：`src/constants/xml.ts`
- 行数：86
- 主要功能简介：产品常量。
- 说明：所在目录 `src/constants` 的职责见上文分类。

### 2.61 目录组 `src/context`（28 个文件，约 3530 行）

该组位于仓库相对路径 `src/context`。系统上下文拼装。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `QueuedMessageContext.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/QueuedMessageContext.tsx`
- 行数：62
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `fpsMetrics.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/fpsMetrics.tsx`
- 行数：29
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/context` 的职责见上文分类。

#### `mailbox.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/mailbox.tsx`
- 行数：37
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/context` 的职责见上文分类。

#### `modalContext.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/modalContext.tsx`
- 行数：57
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `notifications.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/notifications.tsx`
- 行数：252
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `overlayContext.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/overlayContext.tsx`
- 行数：150
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `promptOverlayContext.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/promptOverlayContext.tsx`
- 行数：158
- 主要功能简介：系统上下文拼装。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `fileA.ts`

- 所在目录：`src/context/repoMap/__fixtures__/mini-repo`
- 完整路径：`src/context/repoMap/__fixtures__/mini-repo/fileA.ts`
- 行数：29
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap/__fixtures__/mini-repo` 的职责见上文分类。

#### `fileB.ts`

- 所在目录：`src/context/repoMap/__fixtures__/mini-repo`
- 完整路径：`src/context/repoMap/__fixtures__/mini-repo/fileB.ts`
- 行数：23
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap/__fixtures__/mini-repo` 的职责见上文分类。

#### `fileC.ts`

- 所在目录：`src/context/repoMap/__fixtures__/mini-repo`
- 完整路径：`src/context/repoMap/__fixtures__/mini-repo/fileC.ts`
- 行数：22
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap/__fixtures__/mini-repo` 的职责见上文分类。

#### `fileD.ts`

- 所在目录：`src/context/repoMap/__fixtures__/mini-repo`
- 完整路径：`src/context/repoMap/__fixtures__/mini-repo/fileD.ts`
- 行数：9
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap/__fixtures__/mini-repo` 的职责见上文分类。

#### `fileE.ts`

- 所在目录：`src/context/repoMap/__fixtures__/mini-repo`
- 完整路径：`src/context/repoMap/__fixtures__/mini-repo/fileE.ts`
- 行数：25
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap/__fixtures__/mini-repo` 的职责见上文分类。

#### `cache.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/cache.ts`
- 行数：235
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `gitFiles.test.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/gitFiles.test.ts`
- 行数：42
- 主要功能简介：系统上下文拼装。
- 说明：运行方式：`bun test ./src/context/repoMap/gitFiles.test.ts`。所在目录 `src/context/repoMap` 的职责见上文分类。

#### `gitFiles.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/gitFiles.ts`
- 行数：133
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `graph.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/graph.ts`
- 行数：92
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/index.ts`
- 行数：217
- 主要功能简介：系统上下文拼装。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/context/repoMap` 的职责见上文分类。

#### `pagerank.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/pagerank.ts`
- 行数：84
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `parser.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/parser.ts`
- 行数：168
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `queries.test.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/queries.test.ts`
- 行数：34
- 主要功能简介：系统上下文拼装。
- 说明：运行方式：`bun test ./src/context/repoMap/queries.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap` 的职责见上文分类。

#### `queries.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/queries.ts`
- 行数：186
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `renderer.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/renderer.ts`
- 行数：72
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `repoMap.test.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/repoMap.test.ts`
- 行数：866
- 主要功能简介：系统上下文拼装。
- 说明：运行方式：`bun test ./src/context/repoMap/repoMap.test.ts`。体量较大（约 866 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/context/repoMap` 的职责见上文分类。

#### `symbolExtractor.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/symbolExtractor.ts`
- 行数：145
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `tokenize.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/tokenize.ts`
- 行数：15
- 主要功能简介：系统上下文拼装。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/context/repoMap` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/context/repoMap`
- 完整路径：`src/context/repoMap/types.ts`
- 行数：82
- 主要功能简介：系统上下文拼装。
- 说明：所在目录 `src/context/repoMap` 的职责见上文分类。

#### `stats.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/stats.tsx`
- 行数：219
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

#### `voice.tsx`

- 所在目录：`src/context`
- 完整路径：`src/context/voice.tsx`
- 行数：87
- 主要功能简介：系统上下文拼装。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/context` 的职责见上文分类。

### 2.62 目录组 `src/context.repoMap.test.ts`（1 个文件，约 236 行）

该组位于仓库相对路径 `src/context.repoMap.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `context.repoMap.test.ts`

- 所在目录：`src`
- 完整路径：`src/context.repoMap.test.ts`
- 行数：236
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/context.repoMap.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.63 目录组 `src/context.ts`（1 个文件，约 299 行）

该组位于仓库相对路径 `src/context.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `context.ts`

- 所在目录：`src`
- 完整路径：`src/context.ts`
- 行数：299
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.64 目录组 `src/coordinator`（2 个文件，约 387 行）

该组位于仓库相对路径 `src/coordinator`。多 Agent 协调器。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `coordinatorMode.ts`

- 所在目录：`src/coordinator`
- 完整路径：`src/coordinator/coordinatorMode.ts`
- 行数：369
- 主要功能简介：多 Agent 协调器。
- 说明：所在目录 `src/coordinator` 的职责见上文分类。

#### `workerAgent.ts`

- 所在目录：`src/coordinator`
- 完整路径：`src/coordinator/workerAgent.ts`
- 行数：18
- 主要功能简介：多 Agent 协调器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/coordinator` 的职责见上文分类。

### 2.65 目录组 `src/cost-tracker.cacheIntegration.test.ts`（1 个文件，约 141 行）

该组位于仓库相对路径 `src/cost-tracker.cacheIntegration.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `cost-tracker.cacheIntegration.test.ts`

- 所在目录：`src`
- 完整路径：`src/cost-tracker.cacheIntegration.test.ts`
- 行数：141
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/cost-tracker.cacheIntegration.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.66 目录组 `src/cost-tracker.customPricing.test.ts`（1 个文件，约 143 行）

该组位于仓库相对路径 `src/cost-tracker.customPricing.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `cost-tracker.customPricing.test.ts`

- 所在目录：`src`
- 完整路径：`src/cost-tracker.customPricing.test.ts`
- 行数：143
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/cost-tracker.customPricing.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.67 目录组 `src/cost-tracker.format.test.ts`（1 个文件，约 209 行）

该组位于仓库相对路径 `src/cost-tracker.format.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `cost-tracker.format.test.ts`

- 所在目录：`src`
- 完整路径：`src/cost-tracker.format.test.ts`
- 行数：209
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/cost-tracker.format.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.68 目录组 `src/cost-tracker.ts`（1 个文件，约 421 行）

该组位于仓库相对路径 `src/cost-tracker.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `cost-tracker.ts`

- 所在目录：`src`
- 完整路径：`src/cost-tracker.ts`
- 行数：421
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.69 目录组 `src/costHook.ts`（1 个文件，约 22 行）

该组位于仓库相对路径 `src/costHook.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `costHook.ts`

- 所在目录：`src`
- 完整路径：`src/costHook.ts`
- 行数：22
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.70 目录组 `src/daemon`（2 个文件，约 24 行）

该组位于仓库相对路径 `src/daemon`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `main.ts`

- 所在目录：`src/daemon`
- 完整路径：`src/daemon/main.ts`
- 行数：11
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/daemon` 的职责见上文分类。

#### `workerRegistry.ts`

- 所在目录：`src/daemon`
- 完整路径：`src/daemon/workerRegistry.ts`
- 行数：13
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/daemon` 的职责见上文分类。

### 2.71 目录组 `src/dialogLaunchers.tsx`（1 个文件，约 144 行）

该组位于仓库相对路径 `src/dialogLaunchers.tsx`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `dialogLaunchers.tsx`

- 所在目录：`src`
- 完整路径：`src/dialogLaunchers.tsx`
- 行数：144
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src` 的职责见上文分类。

### 2.72 目录组 `src/entrypoints`（31 个文件，约 13738 行）

该组位于仓库相对路径 `src/entrypoints`。CLI/MCP/SDK 入口。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `agentSdkTypes.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/agentSdkTypes.ts`
- 行数：243
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `applyChildProcessHeapOptions.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/applyChildProcessHeapOptions.ts`
- 行数：46
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `cli.skills.test.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/cli.skills.test.ts`
- 行数：306
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：运行方式：`bun test ./src/entrypoints/cli.skills.test.ts`。所在目录 `src/entrypoints` 的职责见上文分类。

#### `cli.test.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/cli.test.ts`
- 行数：1259
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：运行方式：`bun test ./src/entrypoints/cli.test.ts`。体量较大（约 1259 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints` 的职责见上文分类。

#### `cli.tsx`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/cli.tsx`
- 行数：823
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 823 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints` 的职责见上文分类。

#### `init.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/init.ts`
- 行数：226
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `mcp.test.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/mcp.test.ts`
- 行数：103
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：运行方式：`bun test ./src/entrypoints/mcp.test.ts`。所在目录 `src/entrypoints` 的职责见上文分类。

#### `mcp.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/mcp.ts`
- 行数：267
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `sandboxTypes.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/sandboxTypes.ts`
- 行数：156
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `sdk.d.ts`

- 所在目录：`src/entrypoints`
- 完整路径：`src/entrypoints/sdk.d.ts`
- 行数：601
- 主要功能简介：CLI/MCP/SDK 入口。
- 说明：所在目录 `src/entrypoints` 的职责见上文分类。

#### `agentDefinitions.test.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/agentDefinitions.test.ts`
- 行数：196
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：运行方式：`bun test ./src/entrypoints/sdk/agentDefinitions.test.ts`。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `agentDefinitions.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/agentDefinitions.ts`
- 行数：118
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `casing.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/casing.ts`
- 行数：53
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `controlSchemas.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/controlSchemas.ts`
- 行数：663
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `controlTypes.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/controlTypes.ts`
- 行数：43
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `coreSchemas.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/coreSchemas.ts`
- 行数：1974
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：体量较大（约 1974 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `coreTypes.generated.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/coreTypes.generated.ts`
- 行数：2385
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。体量较大（约 2385 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `coreTypes.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/coreTypes.ts`
- 行数：62
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/index.ts`
- 行数：288
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `interruption.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/interruption.ts`
- 行数：14
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `permissions.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/permissions.ts`
- 行数：706
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `query.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/query.ts`
- 行数：1181
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：体量较大（约 1181 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `runtimeTypes.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/runtimeTypes.ts`
- 行数：69
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `sdkUtilityTypes.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/sdkUtilityTypes.ts`
- 行数：20
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `sessions.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/sessions.ts`
- 行数：442
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `settingsTypes.generated.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/settingsTypes.generated.ts`
- 行数：13
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。短文件，多为常量、再导出或薄包装。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `shared.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/shared.ts`
- 行数：417
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `stubLeakDetection.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/stubLeakDetection.ts`
- 行数：51
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `toolTypes.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/toolTypes.ts`
- 行数：2
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `transcript.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/transcript.ts`
- 行数：172
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：所在目录 `src/entrypoints/sdk` 的职责见上文分类。

#### `v2.ts`

- 所在目录：`src/entrypoints/sdk`
- 完整路径：`src/entrypoints/sdk/v2.ts`
- 行数：839
- 主要功能简介：对外 SDK 类型与运行时。
- 说明：体量较大（约 839 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/entrypoints/sdk` 的职责见上文分类。

### 2.73 目录组 `src/environment-runner`（1 个文件，约 13 行）

该组位于仓库相对路径 `src/environment-runner`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `main.ts`

- 所在目录：`src/environment-runner`
- 完整路径：`src/environment-runner/main.ts`
- 行数：13
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/environment-runner` 的职责见上文分类。

### 2.74 目录组 `src/global.d.ts`（1 个文件，约 23 行）

该组位于仓库相对路径 `src/global.d.ts`。TypeScript 类型声明。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `global.d.ts`

- 所在目录：`src`
- 完整路径：`src/global.d.ts`
- 行数：23
- 主要功能简介：TypeScript 类型声明。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.75 目录组 `src/grpc`（2 个文件，约 410 行）

该组位于仓库相对路径 `src/grpc`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `server.interruptionTrace.test.ts`

- 所在目录：`src/grpc`
- 完整路径：`src/grpc/server.interruptionTrace.test.ts`
- 行数：89
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/grpc/server.interruptionTrace.test.ts`。所在目录 `src/grpc` 的职责见上文分类。

#### `server.ts`

- 所在目录：`src/grpc`
- 完整路径：`src/grpc/server.ts`
- 行数：321
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/grpc` 的职责见上文分类。

### 2.76 目录组 `src/history.ts`（1 个文件，约 464 行）

该组位于仓库相对路径 `src/history.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `history.ts`

- 所在目录：`src`
- 完整路径：`src/history.ts`
- 行数：464
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.77 目录组 `src/hooks`（113 个文件，约 22161 行）

该组位于仓库相对路径 `src/hooks`。React 钩子，连接输入、权限、IDE、队列。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `fileSuggestions.test.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/fileSuggestions.test.ts`
- 行数：430
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/fileSuggestions.test.ts`。所在目录 `src/hooks` 的职责见上文分类。

#### `fileSuggestions.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/fileSuggestions.ts`
- 行数：1180
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：体量较大（约 1180 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/hooks` 的职责见上文分类。

#### `npmDeprecationNotification.ts`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/npmDeprecationNotification.ts`
- 行数：41
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useAutoModeUnavailableNotification.ts`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useAutoModeUnavailableNotification.ts`
- 行数：56
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useCanSwitchToExistingSubscription.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useCanSwitchToExistingSubscription.tsx`
- 行数：59
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useDeprecationWarningNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useDeprecationWarningNotification.tsx`
- 行数：43
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useFastModeNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useFastModeNotification.tsx`
- 行数：161
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useIDEStatusIndicator.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useIDEStatusIndicator.tsx`
- 行数：185
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useInstallMessages.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useInstallMessages.tsx`
- 行数：25
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useLspInitializationNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useLspInitializationNotification.tsx`
- 行数：141
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useMcpConnectivityStatus.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useMcpConnectivityStatus.tsx`
- 行数：92
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useModelMigrationNotifications.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useModelMigrationNotifications.tsx`
- 行数：51
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useNpmDeprecationNotification.test.ts`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useNpmDeprecationNotification.test.ts`
- 行数：60
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/notifs/useNpmDeprecationNotification.test.ts`。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useNpmDeprecationNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useNpmDeprecationNotification.tsx`
- 行数：6
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `usePluginAutoupdateNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/usePluginAutoupdateNotification.tsx`
- 行数：82
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `usePluginInstallationStatus.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/usePluginInstallationStatus.tsx`
- 行数：127
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useRateLimitWarningNotification.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useRateLimitWarningNotification.tsx`
- 行数：113
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useSettingsErrors.tsx`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useSettingsErrors.tsx`
- 行数：68
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useStartupNotification.ts`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useStartupNotification.ts`
- 行数：41
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `useTeammateShutdownNotification.ts`

- 所在目录：`src/hooks/notifs`
- 完整路径：`src/hooks/notifs/useTeammateShutdownNotification.ts`
- 行数：78
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/notifs` 的职责见上文分类。

#### `renderPlaceholder.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/renderPlaceholder.ts`
- 行数：51
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `PermissionContext.ts`

- 所在目录：`src/hooks/toolPermission`
- 完整路径：`src/hooks/toolPermission/PermissionContext.ts`
- 行数：580
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/toolPermission` 的职责见上文分类。

#### `coordinatorHandler.ts`

- 所在目录：`src/hooks/toolPermission/handlers`
- 完整路径：`src/hooks/toolPermission/handlers/coordinatorHandler.ts`
- 行数：65
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/toolPermission/handlers` 的职责见上文分类。

#### `interactiveHandler.test.ts`

- 所在目录：`src/hooks/toolPermission/handlers`
- 完整路径：`src/hooks/toolPermission/handlers/interactiveHandler.test.ts`
- 行数：320
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/toolPermission/handlers/interactiveHandler.test.ts`。所在目录 `src/hooks/toolPermission/handlers` 的职责见上文分类。

#### `interactiveHandler.ts`

- 所在目录：`src/hooks/toolPermission/handlers`
- 完整路径：`src/hooks/toolPermission/handlers/interactiveHandler.ts`
- 行数：630
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/toolPermission/handlers` 的职责见上文分类。

#### `swarmWorkerHandler.ts`

- 所在目录：`src/hooks/toolPermission/handlers`
- 完整路径：`src/hooks/toolPermission/handlers/swarmWorkerHandler.ts`
- 行数：159
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/toolPermission/handlers` 的职责见上文分类。

#### `permissionLogging.ts`

- 所在目录：`src/hooks/toolPermission`
- 完整路径：`src/hooks/toolPermission/permissionLogging.ts`
- 行数：223
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks/toolPermission` 的职责见上文分类。

#### `unifiedSuggestions.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/unifiedSuggestions.ts`
- 行数：202
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useAfterFirstRender.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useAfterFirstRender.ts`
- 行数：17
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useApiKeyVerification.test.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useApiKeyVerification.test.tsx`
- 行数：141
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/hooks/useApiKeyVerification.test.tsx`。所在目录 `src/hooks` 的职责见上文分类。

#### `useApiKeyVerification.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useApiKeyVerification.ts`
- 行数：103
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useArrowKeyHistory.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useArrowKeyHistory.tsx`
- 行数：228
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useAssistantHistory.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useAssistantHistory.ts`
- 行数：250
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useAwaySummary.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useAwaySummary.ts`
- 行数：125
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useBackgroundTaskNavigation.interruptionTrace.test.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useBackgroundTaskNavigation.interruptionTrace.test.tsx`
- 行数：223
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/hooks/useBackgroundTaskNavigation.interruptionTrace.test.tsx`。所在目录 `src/hooks` 的职责见上文分类。

#### `useBackgroundTaskNavigation.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useBackgroundTaskNavigation.ts`
- 行数：277
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useBlink.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useBlink.ts`
- 行数：34
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useCanUseTool.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCanUseTool.tsx`
- 行数：204
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useCancelRequest.test.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCancelRequest.test.tsx`
- 行数：304
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/hooks/useCancelRequest.test.tsx`。所在目录 `src/hooks` 的职责见上文分类。

#### `useCancelRequest.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCancelRequest.ts`
- 行数：305
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useChromeExtensionNotification.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useChromeExtensionNotification.tsx`
- 行数：49
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useClaudeCodeHintRecommendation.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useClaudeCodeHintRecommendation.tsx`
- 行数：128
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useClipboardImageHint.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useClipboardImageHint.ts`
- 行数：77
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useCommandKeybindings.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCommandKeybindings.tsx`
- 行数：107
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useCommandQueue.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCommandQueue.ts`
- 行数：15
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useCopyOnSelect.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useCopyOnSelect.ts`
- 行数：98
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDeferredHookMessages.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDeferredHookMessages.ts`
- 行数：46
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDiffData.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDiffData.ts`
- 行数：110
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDiffInIDE.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDiffInIDE.ts`
- 行数：379
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDirectConnect.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDirectConnect.ts`
- 行数：184
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDoublePress.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDoublePress.ts`
- 行数：62
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useDynamicConfig.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useDynamicConfig.ts`
- 行数：22
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useEffectEventCompat.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useEffectEventCompat.ts`
- 行数：16
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useElapsedTime.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useElapsedTime.ts`
- 行数：37
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useExitOnCtrlCD.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useExitOnCtrlCD.ts`
- 行数：95
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useExitOnCtrlCDWithKeybindings.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useExitOnCtrlCDWithKeybindings.ts`
- 行数：24
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useFileHistorySnapshotInit.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useFileHistorySnapshotInit.ts`
- 行数：25
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useGlobalKeybindings.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useGlobalKeybindings.tsx`
- 行数：248
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useHistorySearch.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useHistorySearch.ts`
- 行数：303
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useIDEIntegration.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIDEIntegration.tsx`
- 行数：69
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useIdeAtMentioned.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIdeAtMentioned.ts`
- 行数：76
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useIdeConnectionStatus.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIdeConnectionStatus.ts`
- 行数：33
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useIdeLogging.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIdeLogging.ts`
- 行数：41
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useIdeSelection.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIdeSelection.ts`
- 行数：150
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useInboxPoller.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useInboxPoller.ts`
- 行数：1009
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：体量较大（约 1009 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/hooks` 的职责见上文分类。

#### `useInputBuffer.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useInputBuffer.ts`
- 行数：132
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useIssueFlagBanner.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useIssueFlagBanner.ts`
- 行数：133
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useLogMessages.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useLogMessages.ts`
- 行数：119
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useLspPluginRecommendation.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useLspPluginRecommendation.tsx`
- 行数：193
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useMailboxBridge.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMailboxBridge.ts`
- 行数：21
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useMainLoopModel.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMainLoopModel.ts`
- 行数：34
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useManagePlugins.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useManagePlugins.ts`
- 行数：329
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useMergedClients.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMergedClients.ts`
- 行数：23
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useMergedCommands.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMergedCommands.ts`
- 行数：15
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useMergedTools.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMergedTools.ts`
- 行数：44
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useMinDisplayTime.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useMinDisplayTime.ts`
- 行数：35
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useNotifyAfterTimeout.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useNotifyAfterTimeout.ts`
- 行数：65
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useOfficialMarketplaceNotification.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useOfficialMarketplaceNotification.tsx`
- 行数：47
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `usePasteHandler.image-path-error.test.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePasteHandler.image-path-error.test.ts`
- 行数：150
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/usePasteHandler.image-path-error.test.ts`。所在目录 `src/hooks` 的职责见上文分类。

#### `usePasteHandler.test.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePasteHandler.test.ts`
- 行数：64
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/usePasteHandler.test.ts`。所在目录 `src/hooks` 的职责见上文分类。

#### `usePasteHandler.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePasteHandler.ts`
- 行数：345
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `usePluginRecommendationBase.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePluginRecommendationBase.tsx`
- 行数：104
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `usePrStatus.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePrStatus.ts`
- 行数：106
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `usePromptSuggestion.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePromptSuggestion.ts`
- 行数：177
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `usePromptsFromClaudeInChrome.test.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePromptsFromClaudeInChrome.test.ts`
- 行数：19
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/usePromptsFromClaudeInChrome.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `usePromptsFromClaudeInChrome.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/usePromptsFromClaudeInChrome.tsx`
- 行数：76
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useQueueProcessor.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useQueueProcessor.ts`
- 行数：68
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useRemoteSession.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useRemoteSession.ts`
- 行数：569
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useReplBridge.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useReplBridge.tsx`
- 行数：718
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useSSHSession.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSSHSession.ts`
- 行数：247
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useScheduledTasks.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useScheduledTasks.ts`
- 行数：139
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useSearchInput.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSearchInput.ts`
- 行数：364
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useSessionBackgrounding.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSessionBackgrounding.ts`
- 行数：158
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useSettings.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSettings.ts`
- 行数：17
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useSettingsChange.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSettingsChange.ts`
- 行数：25
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useSkillsChange.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSkillsChange.ts`
- 行数：62
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useSwarmInitialization.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSwarmInitialization.ts`
- 行数：81
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useSwarmPermissionPoller.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useSwarmPermissionPoller.ts`
- 行数：222
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useTasksV2.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTasksV2.ts`
- 行数：250
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useTeammateViewAutoExit.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTeammateViewAutoExit.ts`
- 行数：63
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useTeleportResume.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTeleportResume.tsx`
- 行数：88
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

#### `useTerminalSize.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTerminalSize.ts`
- 行数：15
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useTextInput.test.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTextInput.test.ts`
- 行数：439
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：运行方式：`bun test ./src/hooks/useTextInput.test.ts`。所在目录 `src/hooks` 的职责见上文分类。

#### `useTextInput.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTextInput.ts`
- 行数：930
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：体量较大（约 930 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/hooks` 的职责见上文分类。

#### `useTimeout.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTimeout.ts`
- 行数：14
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useTurnDiffs.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTurnDiffs.ts`
- 行数：213
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useTypeahead.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useTypeahead.tsx`
- 行数：1388
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1388 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/hooks` 的职责见上文分类。

#### `useUpdateNotification.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useUpdateNotification.ts`
- 行数：34
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useVimInput.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useVimInput.ts`
- 行数：377
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useVirtualScroll.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useVirtualScroll.ts`
- 行数：721
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：所在目录 `src/hooks` 的职责见上文分类。

#### `useVoice.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useVoice.ts`
- 行数：1144
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：体量较大（约 1144 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/hooks` 的职责见上文分类。

#### `useVoiceEnabled.ts`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useVoiceEnabled.ts`
- 行数：25
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/hooks` 的职责见上文分类。

#### `useVoiceIntegration.tsx`

- 所在目录：`src/hooks`
- 完整路径：`src/hooks/useVoiceIntegration.tsx`
- 行数：676
- 主要功能简介：React 钩子，连接输入、权限、IDE、队列。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/hooks` 的职责见上文分类。

### 2.78 目录组 `src/i18n`（6 个文件，约 352 行）

该组位于仓库相对路径 `src/i18n`。文案与语言包。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `commandDescriptions.ts`

- 所在目录：`src/i18n`
- 完整路径：`src/i18n/commandDescriptions.ts`
- 行数：70
- 主要功能简介：文案与语言包。
- 说明：所在目录 `src/i18n` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/i18n`
- 完整路径：`src/i18n/index.ts`
- 行数：41
- 主要功能简介：文案与语言包。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/i18n` 的职责见上文分类。

#### `en.ts`

- 所在目录：`src/i18n/languages`
- 完整路径：`src/i18n/languages/en.ts`
- 行数：107
- 主要功能简介：文案与语言包。
- 说明：所在目录 `src/i18n/languages` 的职责见上文分类。

#### `vi.ts`

- 所在目录：`src/i18n/languages`
- 完整路径：`src/i18n/languages/vi.ts`
- 行数：108
- 主要功能简介：文案与语言包。
- 说明：所在目录 `src/i18n/languages` 的职责见上文分类。

#### `locale.ts`

- 所在目录：`src/i18n`
- 完整路径：`src/i18n/locale.ts`
- 行数：19
- 主要功能简介：文案与语言包。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/i18n` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/i18n`
- 完整路径：`src/i18n/types.ts`
- 行数：7
- 主要功能简介：文案与语言包。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/i18n` 的职责见上文分类。

### 2.79 目录组 `src/ink`（110 个文件，约 22209 行）

该组位于仓库相对路径 `src/ink`。终端渲染器。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `Ansi.tsx`

- 所在目录：`src/ink`
- 完整路径：`src/ink/Ansi.tsx`
- 行数：291
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink` 的职责见上文分类。

#### `bidi.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/bidi.ts`
- 行数：139
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `clearTerminal.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/clearTerminal.ts`
- 行数：74
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `colorize.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/colorize.ts`
- 行数：231
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `AlternateScreen.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/AlternateScreen.tsx`
- 行数：79
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `App.test.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/App.test.tsx`
- 行数：221
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/ink/components/App.test.tsx`。所在目录 `src/ink/components` 的职责见上文分类。

#### `App.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/App.tsx`
- 行数：709
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `AppContext.ts`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/AppContext.ts`
- 行数：21
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `Box.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Box.tsx`
- 行数：209
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `Button.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Button.tsx`
- 行数：191
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `ClockContext.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/ClockContext.tsx`
- 行数：111
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `CursorDeclarationContext.ts`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/CursorDeclarationContext.ts`
- 行数：32
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `ErrorOverview.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/ErrorOverview.tsx`
- 行数：27
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `Link.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Link.tsx`
- 行数：41
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `Newline.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Newline.tsx`
- 行数：38
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `NoSelect.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/NoSelect.tsx`
- 行数：67
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `RawAnsi.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/RawAnsi.tsx`
- 行数：56
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `ScrollBox.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/ScrollBox.tsx`
- 行数：236
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `Spacer.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Spacer.tsx`
- 行数：19
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `StdinContext.ts`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/StdinContext.ts`
- 行数：49
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/components` 的职责见上文分类。

#### `TerminalFocusContext.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/TerminalFocusContext.tsx`
- 行数：51
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `TerminalSizeContext.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/TerminalSizeContext.tsx`
- 行数：6
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/ink/components` 的职责见上文分类。

#### `Text.tsx`

- 所在目录：`src/ink/components`
- 完整路径：`src/ink/components/Text.tsx`
- 行数：253
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/ink/components` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/constants.ts`
- 行数：2
- 主要功能简介：终端渲染器。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `cursor.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/cursor.ts`
- 行数：9
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `devtools.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/devtools.ts`
- 行数：2
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `dom.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/dom.ts`
- 行数：487
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `click-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/click-event.ts`
- 行数：38
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `dispatcher.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/dispatcher.ts`
- 行数：234
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/events` 的职责见上文分类。

#### `emitter.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/emitter.ts`
- 行数：39
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `event-handlers.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/event-handlers.ts`
- 行数：73
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/events` 的职责见上文分类。

#### `event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/event.ts`
- 行数：11
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `focus-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/focus-event.ts`
- 行数：21
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `input-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/input-event.ts`
- 行数：222
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/events` 的职责见上文分类。

#### `keyboard-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/keyboard-event.ts`
- 行数：51
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/events` 的职责见上文分类。

#### `paste-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/paste-event.ts`
- 行数：17
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `resize-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/resize-event.ts`
- 行数：21
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `terminal-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/terminal-event.ts`
- 行数：107
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/events` 的职责见上文分类。

#### `terminal-focus-event.ts`

- 所在目录：`src/ink/events`
- 完整路径：`src/ink/events/terminal-focus-event.ts`
- 行数：19
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/events` 的职责见上文分类。

#### `focus.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/focus.ts`
- 行数：181
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `frame.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/frame.ts`
- 行数：124
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `get-max-width.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/get-max-width.ts`
- 行数：27
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `global.d.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/global.d.ts`
- 行数：56
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `hit-test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/hit-test.ts`
- 行数：130
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `use-animation-frame.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-animation-frame.ts`
- 行数：57
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-app.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-app.ts`
- 行数：8
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-declared-cursor.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-declared-cursor.ts`
- 行数：73
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-input.test.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-input.test.ts`
- 行数：166
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/hooks/use-input.test.ts`。所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-input.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-input.ts`
- 行数：129
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-interval.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-interval.ts`
- 行数：67
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-search-highlight.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-search-highlight.ts`
- 行数：53
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-selection.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-selection.ts`
- 行数：104
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-stdin.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-stdin.ts`
- 行数：8
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-tab-status.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-tab-status.ts`
- 行数：72
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-terminal-focus.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-terminal-focus.ts`
- 行数：16
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-terminal-title.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-terminal-title.ts`
- 行数：31
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/hooks` 的职责见上文分类。

#### `use-terminal-viewport.ts`

- 所在目录：`src/ink/hooks`
- 完整路径：`src/ink/hooks/use-terminal-viewport.ts`
- 行数：96
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/hooks` 的职责见上文分类。

#### `ink.tsx`

- 所在目录：`src/ink`
- 完整路径：`src/ink/ink.tsx`
- 行数：1747
- 主要功能简介：终端渲染器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1747 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `instances.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/instances.ts`
- 行数：10
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `engine.ts`

- 所在目录：`src/ink/layout`
- 完整路径：`src/ink/layout/engine.ts`
- 行数：6
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink/layout` 的职责见上文分类。

#### `geometry.ts`

- 所在目录：`src/ink/layout`
- 完整路径：`src/ink/layout/geometry.ts`
- 行数：97
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/layout` 的职责见上文分类。

#### `node.ts`

- 所在目录：`src/ink/layout`
- 完整路径：`src/ink/layout/node.ts`
- 行数：152
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/layout` 的职责见上文分类。

#### `yoga.ts`

- 所在目录：`src/ink/layout`
- 完整路径：`src/ink/layout/yoga.ts`
- 行数：308
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/layout` 的职责见上文分类。

#### `line-width-cache.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/line-width-cache.ts`
- 行数：24
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `log-update.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/log-update.test.ts`
- 行数：126
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/log-update.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `log-update.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/log-update.ts`
- 行数：857
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 857 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `measure-element.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/measure-element.ts`
- 行数：23
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `measure-text.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/measure-text.ts`
- 行数：47
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `node-cache.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/node-cache.ts`
- 行数：54
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `optimizer.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/optimizer.ts`
- 行数：93
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `output.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/output.test.ts`
- 行数：258
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/output.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `output.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/output.ts`
- 行数：930
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 930 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `parse-keypress.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/parse-keypress.test.ts`
- 行数：151
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/parse-keypress.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `parse-keypress.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/parse-keypress.ts`
- 行数：914
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 914 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `reconciler.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/reconciler.test.ts`
- 行数：369
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/reconciler.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `reconciler.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/reconciler.ts`
- 行数：548
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `render-border.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/render-border.ts`
- 行数：231
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `render-node-to-output.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/render-node-to-output.ts`
- 行数：1537
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 1537 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `render-to-screen.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/render-to-screen.ts`
- 行数：226
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `renderer.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/renderer.test.ts`
- 行数：151
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/renderer.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `renderer.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/renderer.ts`
- 行数：219
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `root.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/root.ts`
- 行数：184
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `screen.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/screen.ts`
- 行数：1486
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 1486 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `searchHighlight.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/searchHighlight.ts`
- 行数：93
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `selection.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/selection.ts`
- 行数：917
- 主要功能简介：终端渲染器。
- 说明：体量较大（约 917 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/ink` 的职责见上文分类。

#### `squash-text-nodes.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/squash-text-nodes.ts`
- 行数：92
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `stringWidth.test.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/stringWidth.test.ts`
- 行数：56
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/stringWidth.test.ts`。所在目录 `src/ink` 的职责见上文分类。

#### `stringWidth.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/stringWidth.ts`
- 行数：235
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `styles.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/styles.ts`
- 行数：771
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `supports-hyperlinks.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/supports-hyperlinks.ts`
- 行数：57
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `tabstops.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/tabstops.ts`
- 行数：46
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `terminal-focus-state.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/terminal-focus-state.ts`
- 行数：47
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `terminal-querier.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/terminal-querier.ts`
- 行数：212
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `terminal.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/terminal.ts`
- 行数：275
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `termio.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/termio.ts`
- 行数：42
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `ansi.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/ansi.ts`
- 行数：75
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `csi.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/csi.ts`
- 行数：319
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `dec.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/dec.ts`
- 行数：60
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `esc.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/esc.ts`
- 行数：67
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `osc.test.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/osc.test.ts`
- 行数：172
- 主要功能简介：终端渲染器。
- 说明：运行方式：`bun test ./src/ink/termio/osc.test.ts`。所在目录 `src/ink/termio` 的职责见上文分类。

#### `osc.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/osc.ts`
- 行数：518
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `parser.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/parser.ts`
- 行数：394
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `sgr.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/sgr.ts`
- 行数：308
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `tokenize.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/tokenize.ts`
- 行数：319
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/ink/termio`
- 完整路径：`src/ink/termio/types.ts`
- 行数：236
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink/termio` 的职责见上文分类。

#### `useTerminalNotification.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/useTerminalNotification.ts`
- 行数：126
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `warn.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/warn.ts`
- 行数：9
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `widest-line.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/widest-line.ts`
- 行数：19
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

#### `wrap-text.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/wrap-text.ts`
- 行数：74
- 主要功能简介：终端渲染器。
- 说明：所在目录 `src/ink` 的职责见上文分类。

#### `wrapAnsi.ts`

- 所在目录：`src/ink`
- 完整路径：`src/ink/wrapAnsi.ts`
- 行数：20
- 主要功能简介：终端渲染器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/ink` 的职责见上文分类。

### 2.80 目录组 `src/ink.ts`（1 个文件，约 85 行）

该组位于仓库相对路径 `src/ink.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `ink.ts`

- 所在目录：`src`
- 完整路径：`src/ink.ts`
- 行数：85
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.81 目录组 `src/integrations`（146 个文件，约 35915 行）

该组位于仓库相对路径 `src/integrations`。多模型集成系统。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `client.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/client.test.ts`
- 行数：649
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/client.test.ts`。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `client.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/client.ts`
- 行数：631
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `config.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/config.test.ts`
- 行数：266
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/config.test.ts`。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/config.ts`
- 行数：340
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/index.ts`
- 行数：25
- 主要功能简介：多模型集成系统。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `messages.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/messages.ts`
- 行数：30
- 主要功能简介：多模型集成系统。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `onboarding.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/onboarding.test.ts`
- 行数：632
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/onboarding.test.ts`。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `onboarding.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/onboarding.ts`
- 行数：276
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/prompt.ts`
- 行数：64
- 主要功能简介：多模型集成系统。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `topup.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/topup.test.ts`
- 行数：2026
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/topup.test.ts`。体量较大（约 2026 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `topup.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/topup.ts`
- 行数：1175
- 主要功能简介：多模型集成系统。
- 说明：体量较大（约 1175 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `topupState.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/topupState.test.ts`
- 行数：1463
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/topupState.test.ts`。体量较大（约 1463 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `topupState.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/topupState.ts`
- 行数：1563
- 主要功能简介：多模型集成系统。
- 说明：体量较大（约 1563 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `transport.test.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/transport.test.ts`
- 行数：30
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/aimlapi/transport.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `transport.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/transport.ts`
- 行数：66
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `validation.ts`

- 所在目录：`src/integrations/aimlapi`
- 完整路径：`src/integrations/aimlapi/validation.ts`
- 行数：62
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/aimlapi` 的职责见上文分类。

#### `custom.ts`

- 所在目录：`src/integrations/anthropicProxies`
- 完整路径：`src/integrations/anthropicProxies/custom.ts`
- 行数：45
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations/anthropicProxies` 的职责见上文分类。

#### `artifactGenerator.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/artifactGenerator.test.ts`
- 行数：277
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/artifactGenerator.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `artifactGenerator.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/artifactGenerator.ts`
- 行数：517
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `claude.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/claude.ts`
- 行数：20
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `deepseek.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/deepseek.ts`
- 行数：21
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `fireworks.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/fireworks.ts`
- 行数：293
- 主要功能简介：品牌展示元数据。
- 说明：所在目录 `src/integrations/brands` 的职责见上文分类。

#### `gemini.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/gemini.ts`
- 行数：27
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `glm.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/glm.ts`
- 行数：30
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `gpt.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/gpt.ts`
- 行数：38
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `kimi.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/kimi.ts`
- 行数：26
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `ling.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/ling.ts`
- 行数：16
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `llama.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/llama.ts`
- 行数：32
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `longcat.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/longcat.ts`
- 行数：15
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `macaron.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/macaron.ts`
- 行数：16
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `minimax.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/minimax.ts`
- 行数：28
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `mistral.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/mistral.ts`
- 行数：25
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `nearai.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/nearai.ts`
- 行数：39
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `nemotron.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/nemotron.ts`
- 行数：20
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `openai-compatible-alias.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/openai-compatible-alias.ts`
- 行数：15
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `qwen.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/qwen.ts`
- 行数：23
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `tencent.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/tencent.ts`
- 行数：16
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `xai.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/xai.ts`
- 行数：23
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `xiaomi-mimo.ts`

- 所在目录：`src/integrations/brands`
- 完整路径：`src/integrations/brands/xiaomi-mimo.ts`
- 行数：19
- 主要功能简介：品牌展示元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/brands` 的职责见上文分类。

#### `compatibility.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/compatibility.test.ts`
- 行数：144
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/compatibility.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `compatibility.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/compatibility.ts`
- 行数：60
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `define.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/define.ts`
- 行数：36
- 主要功能简介：多模型集成系统。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations` 的职责见上文分类。

#### `descriptors.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/descriptors.ts`
- 行数：370
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `discoveryCache.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/discoveryCache.test.ts`
- 行数：244
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/discoveryCache.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `discoveryCache.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/discoveryCache.ts`
- 行数：430
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `discoveryService.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/discoveryService.test.ts`
- 行数：1394
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/discoveryService.test.ts`。体量较大（约 1394 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations` 的职责见上文分类。

#### `discoveryService.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/discoveryService.ts`
- 行数：687
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `aimlapi.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/aimlapi.ts`
- 行数：127
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `apismart.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/apismart.test.ts`
- 行数：44
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/apismart.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `apismart.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/apismart.ts`
- 行数：227
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `atlas-cloud.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/atlas-cloud.ts`
- 行数：97
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `atomic-chat.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/atomic-chat.ts`
- 行数：39
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `azure-openai.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/azure-openai.ts`
- 行数：34
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `bedrock.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/bedrock.ts`
- 行数：30
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `clinepass.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/clinepass.ts`
- 行数：230
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `cloudflare.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/cloudflare.ts`
- 行数：95
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `commandcode.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/commandcode.test.ts`
- 行数：263
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/commandcode.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `commandcode.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/commandcode.ts`
- 行数：204
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `concentrate.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/concentrate.test.ts`
- 行数：79
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/concentrate.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `concentrate.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/concentrate.ts`
- 行数：111
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `custom.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/custom.test.ts`
- 行数：186
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/custom.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `custom.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/custom.ts`
- 行数：91
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `dashscope-cn.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/dashscope-cn.ts`
- 行数：34
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `dashscope-intl.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/dashscope-intl.ts`
- 行数：34
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `github-enterprise.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/github-enterprise.ts`
- 行数：197
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `github.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/github.ts`
- 行数：239
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `gitlawb-opengateway.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/gitlawb-opengateway.test.ts`
- 行数：76
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/gitlawb-opengateway.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `gitlawb-opengateway.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/gitlawb-opengateway.ts`
- 行数：265
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `groq.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/groq.ts`
- 行数：56
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `hicap.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/hicap.ts`
- 行数：74
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `kimi-code.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/kimi-code.ts`
- 行数：46
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `llmtr.models.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/llmtr.models.ts`
- 行数：129
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `llmtr.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/llmtr.test.ts`
- 行数：187
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/llmtr.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `llmtr.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/llmtr.ts`
- 行数：49
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `lmstudio.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/lmstudio.ts`
- 行数：39
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `mistral.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/mistral.ts`
- 行数：59
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `nvidia-nim.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/nvidia-nim.test.ts`
- 行数：149
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/nvidia-nim.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `nvidia-nim.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/nvidia-nim.ts`
- 行数：104
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `ollama.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/ollama.ts`
- 行数：58
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `opencode-go.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/opencode-go.ts`
- 行数：105
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `opencode.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/opencode.test.ts`
- 行数：532
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/opencode.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `opencode.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/opencode.ts`
- 行数：131
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `openrouter.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/openrouter.test.ts`
- 行数：127
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/openrouter.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `openrouter.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/openrouter.ts`
- 行数：156
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `together.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/together.ts`
- 行数：34
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `vertex.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/vertex.ts`
- 行数：30
- 主要功能简介：网关描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `xiaomi-mimo-token.test.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/xiaomi-mimo-token.test.ts`
- 行数：56
- 主要功能简介：网关描述符。
- 说明：运行方式：`bun test ./src/integrations/gateways/xiaomi-mimo-token.test.ts`。所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `xiaomi-mimo-token.ts`

- 所在目录：`src/integrations/gateways`
- 完整路径：`src/integrations/gateways/xiaomi-mimo-token.ts`
- 行数：60
- 主要功能简介：网关描述符。
- 说明：所在目录 `src/integrations/gateways` 的职责见上文分类。

#### `integrationArtifacts.generated.ts`

- 所在目录：`src/integrations/generated`
- 完整路径：`src/integrations/generated/integrationArtifacts.generated.ts`
- 行数：103
- 主要功能简介：由脚本生成的集成清单，勿手改。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `src/integrations/generated` 的职责见上文分类。

#### `integrationManifest.generated.ts`

- 所在目录：`src/integrations/generated`
- 完整路径：`src/integrations/generated/integrationManifest.generated.ts`
- 行数：630
- 主要功能简介：由脚本生成的集成清单，勿手改。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `src/integrations/generated` 的职责见上文分类。

#### `gpt56Catalog.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/gpt56Catalog.test.ts`
- 行数：32
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/gpt56Catalog.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/integrations` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/index.test.ts`
- 行数：123
- 主要功能简介：多模型集成系统。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/integrations/index.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/index.ts`
- 行数：159
- 主要功能简介：多模型集成系统。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/integrations` 的职责见上文分类。

#### `ling-tiny.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/ling-tiny.test.ts`
- 行数：77
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/ling-tiny.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `macaron.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/macaron.test.ts`
- 行数：47
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/macaron.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `modelMapping.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/modelMapping.ts`
- 行数：58
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `claude.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/claude.ts`
- 行数：94
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `deepseek.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/deepseek.ts`
- 行数：76
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `fireworks-merged.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/fireworks-merged.ts`
- 行数：4467
- 主要功能简介：模型目录与能力元数据。
- 说明：体量较大（约 4467 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/models` 的职责见上文分类。

#### `gemini.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/gemini.ts`
- 行数：54
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `glm.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/glm.ts`
- 行数：77
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `gpt.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/gpt.ts`
- 行数：67
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `kimi.test.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/kimi.test.ts`
- 行数：8
- 主要功能简介：模型目录与能力元数据。
- 说明：运行方式：`bun test ./src/integrations/models/kimi.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/models` 的职责见上文分类。

#### `kimi.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/kimi.ts`
- 行数：43
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `ling.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/ling.ts`
- 行数：40
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `llama.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/llama.ts`
- 行数：108
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `longcat.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/longcat.ts`
- 行数：21
- 主要功能简介：模型目录与能力元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/models` 的职责见上文分类。

#### `macaron.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/macaron.ts`
- 行数：40
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `minimax.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/minimax.ts`
- 行数：167
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `mistral.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/mistral.ts`
- 行数：40
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `nearai.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/nearai.ts`
- 行数：264
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `nemotron.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/nemotron.ts`
- 行数：58
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `openai-compatible-alias.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/openai-compatible-alias.ts`
- 行数：138
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `opencode.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/opencode.ts`
- 行数：114
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `qwen.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/qwen.ts`
- 行数：112
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `tencent.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/tencent.ts`
- 行数：22
- 主要功能简介：模型目录与能力元数据。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/models` 的职责见上文分类。

#### `xai.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/xai.ts`
- 行数：109
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `xiaomi-mimo.ts`

- 所在目录：`src/integrations/models`
- 完整路径：`src/integrations/models/xiaomi-mimo.ts`
- 行数：54
- 主要功能简介：模型目录与能力元数据。
- 说明：所在目录 `src/integrations/models` 的职责见上文分类。

#### `nearai.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/nearai.test.ts`
- 行数：62
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/nearai.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `profileResolver.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/profileResolver.ts`
- 行数：54
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `providerUiMetadata.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/providerUiMetadata.ts`
- 行数：120
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `registry.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/registry.test.ts`
- 行数：533
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/registry.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `registry.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/registry.ts`
- 行数：480
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `routeMetadata.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/routeMetadata.test.ts`
- 行数：1263
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/routeMetadata.test.ts`。体量较大（约 1263 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations` 的职责见上文分类。

#### `routeMetadata.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/routeMetadata.ts`
- 行数：1527
- 主要功能简介：多模型集成系统。
- 说明：体量较大（约 1527 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations` 的职责见上文分类。

#### `runtimeMetadata.modelLimits.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/runtimeMetadata.modelLimits.test.ts`
- 行数：237
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/runtimeMetadata.modelLimits.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `runtimeMetadata.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/runtimeMetadata.test.ts`
- 行数：1525
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/runtimeMetadata.test.ts`。体量较大（约 1525 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations` 的职责见上文分类。

#### `runtimeMetadata.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/runtimeMetadata.ts`
- 行数：622
- 主要功能简介：多模型集成系统。
- 说明：所在目录 `src/integrations` 的职责见上文分类。

#### `tencent.test.ts`

- 所在目录：`src/integrations`
- 完整路径：`src/integrations/tencent.test.ts`
- 行数：41
- 主要功能简介：多模型集成系统。
- 说明：运行方式：`bun test ./src/integrations/tencent.test.ts`。所在目录 `src/integrations` 的职责见上文分类。

#### `zaiGlmShim.ts`

- 所在目录：`src/integrations/transport`
- 完整路径：`src/integrations/transport/zaiGlmShim.ts`
- 行数：24
- 主要功能简介：多模型集成系统。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/transport` 的职责见上文分类。

#### `anthropic.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/anthropic.ts`
- 行数：27
- 主要功能简介：厂商描述符。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `bankr.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/bankr.ts`
- 行数：40
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `deepseek.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/deepseek.ts`
- 行数：55
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `fireworks.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/fireworks.ts`
- 行数：1701
- 主要功能简介：厂商描述符。
- 说明：体量较大（约 1701 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `gemini.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/gemini.ts`
- 行数：44
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `longcat.test.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/longcat.test.ts`
- 行数：88
- 主要功能简介：厂商描述符。
- 说明：运行方式：`bun test ./src/integrations/vendors/longcat.test.ts`。所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `longcat.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/longcat.ts`
- 行数：67
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `minimax.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/minimax.ts`
- 行数：51
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `moonshot.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/moonshot.ts`
- 行数：44
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `nearai.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/nearai.ts`
- 行数：180
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `openai.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/openai.ts`
- 行数：73
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `venice.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/venice.ts`
- 行数：49
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `xai.test.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/xai.test.ts`
- 行数：110
- 主要功能简介：厂商描述符。
- 说明：运行方式：`bun test ./src/integrations/vendors/xai.test.ts`。所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `xai.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/xai.ts`
- 行数：196
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `xiaomi-mimo.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/xiaomi-mimo.ts`
- 行数：57
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

#### `zai.ts`

- 所在目录：`src/integrations/vendors`
- 完整路径：`src/integrations/vendors/zai.ts`
- 行数：116
- 主要功能简介：厂商描述符。
- 说明：所在目录 `src/integrations/vendors` 的职责见上文分类。

### 2.82 目录组 `src/interactiveHelpers.tsx`（1 个文件，约 404 行）

该组位于仓库相对路径 `src/interactiveHelpers.tsx`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `interactiveHelpers.tsx`

- 所在目录：`src`
- 完整路径：`src/interactiveHelpers.tsx`
- 行数：404
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src` 的职责见上文分类。

### 2.83 目录组 `src/jobs`（1 个文件，约 21 行）

该组位于仓库相对路径 `src/jobs`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `classifier.ts`

- 所在目录：`src/jobs`
- 完整路径：`src/jobs/classifier.ts`
- 行数：21
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/jobs` 的职责见上文分类。

### 2.84 目录组 `src/keybindings`（15 个文件，约 3294 行）

该组位于仓库相对路径 `src/keybindings`。键位默认值与加载。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `KeybindingContext.tsx`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/KeybindingContext.tsx`
- 行数：242
- 主要功能简介：键位默认值与加载。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/keybindings` 的职责见上文分类。

#### `KeybindingProviderSetup.tsx`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/KeybindingProviderSetup.tsx`
- 行数：307
- 主要功能简介：键位默认值与加载。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/keybindings` 的职责见上文分类。

#### `defaultBindings.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/defaultBindings.ts`
- 行数：341
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `loadUserBindings.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/loadUserBindings.ts`
- 行数：472
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `match.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/match.ts`
- 行数：120
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `parser.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/parser.ts`
- 行数：203
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `reservedShortcuts.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/reservedShortcuts.ts`
- 行数：127
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `resolver.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/resolver.ts`
- 行数：244
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `schema.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/schema.ts`
- 行数：237
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `shortcutFormat.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/shortcutFormat.ts`
- 行数：63
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `template.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/template.ts`
- 行数：52
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/types.ts`
- 行数：133
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `useKeybinding.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/useKeybinding.ts`
- 行数：196
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `useShortcutDisplay.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/useShortcutDisplay.ts`
- 行数：59
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

#### `validate.ts`

- 所在目录：`src/keybindings`
- 完整路径：`src/keybindings/validate.ts`
- 行数：498
- 主要功能简介：键位默认值与加载。
- 说明：所在目录 `src/keybindings` 的职责见上文分类。

### 2.85 目录组 `src/main.tsx`（1 个文件，约 4482 行）

该组位于仓库相对路径 `src/main.tsx`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `main.tsx`

- 所在目录：`src`
- 完整路径：`src/main.tsx`
- 行数：4482
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 4482 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.86 目录组 `src/memdir`（17 个文件，约 4445 行）

该组位于仓库相对路径 `src/memdir`。记忆目录。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `autoExtractFacts.test.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/autoExtractFacts.test.ts`
- 行数：343
- 主要功能简介：记忆目录。
- 说明：运行方式：`bun test ./src/memdir/autoExtractFacts.test.ts`。所在目录 `src/memdir` 的职责见上文分类。

#### `autoExtractFacts.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/autoExtractFacts.ts`
- 行数：388
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `findRelevantMemories.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/findRelevantMemories.ts`
- 行数：140
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `memdir.entrypointBytes.test.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memdir.entrypointBytes.test.ts`
- 行数：80
- 主要功能简介：记忆目录。
- 说明：运行方式：`bun test ./src/memdir/memdir.entrypointBytes.test.ts`。所在目录 `src/memdir` 的职责见上文分类。

#### `memdir.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memdir.ts`
- 行数：537
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `memoryAge.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memoryAge.ts`
- 行数：53
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `memoryScan.test.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memoryScan.test.ts`
- 行数：507
- 主要功能简介：记忆目录。
- 说明：运行方式：`bun test ./src/memdir/memoryScan.test.ts`。所在目录 `src/memdir` 的职责见上文分类。

#### `memoryScan.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memoryScan.ts`
- 行数：255
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `memorySecurity.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memorySecurity.ts`
- 行数：162
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `memoryShapeTelemetry.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memoryShapeTelemetry.ts`
- 行数：35
- 主要功能简介：记忆目录。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/memdir` 的职责见上文分类。

#### `memoryTypes.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/memoryTypes.ts`
- 行数：271
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `paths.test.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/paths.test.ts`
- 行数：151
- 主要功能简介：记忆目录。
- 说明：运行方式：`bun test ./src/memdir/paths.test.ts`。所在目录 `src/memdir` 的职责见上文分类。

#### `paths.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/paths.ts`
- 行数：314
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `teamMemPaths.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/teamMemPaths.ts`
- 行数：292
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `teamMemPrompts.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/teamMemPrompts.ts`
- 行数：113
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

#### `vectorIndex.test.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/vectorIndex.test.ts`
- 行数：416
- 主要功能简介：记忆目录。
- 说明：运行方式：`bun test ./src/memdir/vectorIndex.test.ts`。所在目录 `src/memdir` 的职责见上文分类。

#### `vectorIndex.ts`

- 所在目录：`src/memdir`
- 完整路径：`src/memdir/vectorIndex.ts`
- 行数：388
- 主要功能简介：记忆目录。
- 说明：所在目录 `src/memdir` 的职责见上文分类。

### 2.87 目录组 `src/migrations`（10 个文件，约 577 行）

该组位于仓库相对路径 `src/migrations`。配置迁移。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `migrateAutoUpdatesToSettings.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateAutoUpdatesToSettings.ts`
- 行数：61
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateBypassPermissionsAcceptedToSettings.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateBypassPermissionsAcceptedToSettings.ts`
- 行数：40
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateEnableAllProjectMcpServersToSettings.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateEnableAllProjectMcpServersToSettings.ts`
- 行数：118
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateLegacyOpusToCurrent.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateLegacyOpusToCurrent.ts`
- 行数：57
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateOpusToOpus1m.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateOpusToOpus1m.ts`
- 行数：43
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateReplBridgeEnabledToRemoteControlAtStartup.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateReplBridgeEnabledToRemoteControlAtStartup.ts`
- 行数：22
- 主要功能简介：配置迁移。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/migrations` 的职责见上文分类。

#### `migrateSonnet1mToSonnet45.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateSonnet1mToSonnet45.ts`
- 行数：55
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `migrateSonnet45ToSonnet46.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/migrateSonnet45ToSonnet46.ts`
- 行数：67
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `resetAutoModeOptInForDefaultOffer.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/resetAutoModeOptInForDefaultOffer.ts`
- 行数：51
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

#### `resetProToOpusDefault.ts`

- 所在目录：`src/migrations`
- 完整路径：`src/migrations/resetProToOpusDefault.ts`
- 行数：63
- 主要功能简介：配置迁移。
- 说明：所在目录 `src/migrations` 的职责见上文分类。

### 2.88 目录组 `src/native-ts`（5 个文件，约 4140 行）

该组位于仓库相对路径 `src/native-ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `detectLanguage.test.ts`

- 所在目录：`src/native-ts/color-diff`
- 完整路径：`src/native-ts/color-diff/detectLanguage.test.ts`
- 行数：45
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/native-ts/color-diff/detectLanguage.test.ts`。所在目录 `src/native-ts/color-diff` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/native-ts/color-diff`
- 完整路径：`src/native-ts/color-diff/index.ts`
- 行数：1013
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。体量较大（约 1013 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/native-ts/color-diff` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/native-ts/file-index`
- 完整路径：`src/native-ts/file-index/index.ts`
- 行数：370
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/native-ts/file-index` 的职责见上文分类。

#### `enums.ts`

- 所在目录：`src/native-ts/yoga-layout`
- 完整路径：`src/native-ts/yoga-layout/enums.ts`
- 行数：134
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/native-ts/yoga-layout` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/native-ts/yoga-layout`
- 完整路径：`src/native-ts/yoga-layout/index.ts`
- 行数：2578
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。体量较大（约 2578 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/native-ts/yoga-layout` 的职责见上文分类。

### 2.89 目录组 `src/optionalModules.d.ts`（1 个文件，约 619 行）

该组位于仓库相对路径 `src/optionalModules.d.ts`。TypeScript 类型声明。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `optionalModules.d.ts`

- 所在目录：`src`
- 完整路径：`src/optionalModules.d.ts`
- 行数：619
- 主要功能简介：TypeScript 类型声明。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.90 目录组 `src/optionalModules.types.test.ts`（1 个文件，约 30 行）

该组位于仓库相对路径 `src/optionalModules.types.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `optionalModules.types.test.ts`

- 所在目录：`src`
- 完整路径：`src/optionalModules.types.test.ts`
- 行数：30
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/optionalModules.types.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.91 目录组 `src/outputStyles`（1 个文件，约 98 行）

该组位于仓库相对路径 `src/outputStyles`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `loadOutputStylesDir.ts`

- 所在目录：`src/outputStyles`
- 完整路径：`src/outputStyles/loadOutputStylesDir.ts`
- 行数：98
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/outputStyles` 的职责见上文分类。

### 2.92 目录组 `src/plugins`（4 个文件，约 331 行）

该组位于仓库相对路径 `src/plugins`。插件加载。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `builtinPlugins.ts`

- 所在目录：`src/plugins`
- 完整路径：`src/plugins/builtinPlugins.ts`
- 行数：170
- 主要功能简介：插件加载。
- 说明：所在目录 `src/plugins` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/plugins/bundled`
- 完整路径：`src/plugins/bundled/index.ts`
- 行数：24
- 主要功能简介：插件加载。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/plugins/bundled` 的职责见上文分类。

#### `karpathyGuidelines.test.ts`

- 所在目录：`src/plugins/bundled`
- 完整路径：`src/plugins/bundled/karpathyGuidelines.test.ts`
- 行数：41
- 主要功能简介：插件加载。
- 说明：运行方式：`bun test ./src/plugins/bundled/karpathyGuidelines.test.ts`。所在目录 `src/plugins/bundled` 的职责见上文分类。

#### `karpathyGuidelines.ts`

- 所在目录：`src/plugins/bundled`
- 完整路径：`src/plugins/bundled/karpathyGuidelines.ts`
- 行数：96
- 主要功能简介：插件加载。
- 说明：所在目录 `src/plugins/bundled` 的职责见上文分类。

### 2.93 目录组 `src/proactive`（2 个文件，约 135 行）

该组位于仓库相对路径 `src/proactive`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `index.test.ts`

- 所在目录：`src/proactive`
- 完整路径：`src/proactive/index.test.ts`
- 行数：58
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/proactive/index.test.ts`。所在目录 `src/proactive` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/proactive`
- 完整路径：`src/proactive/index.ts`
- 行数：77
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/proactive` 的职责见上文分类。

### 2.94 目录组 `src/projectOnboardingState.test.ts`（1 个文件，约 62 行）

该组位于仓库相对路径 `src/projectOnboardingState.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `projectOnboardingState.test.ts`

- 所在目录：`src`
- 完整路径：`src/projectOnboardingState.test.ts`
- 行数：62
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/projectOnboardingState.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.95 目录组 `src/projectOnboardingState.ts`（1 个文件，约 47 行）

该组位于仓库相对路径 `src/projectOnboardingState.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `projectOnboardingState.ts`

- 所在目录：`src`
- 完整路径：`src/projectOnboardingState.ts`
- 行数：47
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.96 目录组 `src/projectOnboardingSteps.ts`（1 个文件，约 44 行）

该组位于仓库相对路径 `src/projectOnboardingSteps.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `projectOnboardingSteps.ts`

- 所在目录：`src`
- 完整路径：`src/projectOnboardingSteps.ts`
- 行数：44
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.97 目录组 `src/query`（15 个文件，约 6419 行）

该组位于仓库相对路径 `src/query`。查询循环拆出的辅助模块。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `agentStepLimit.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/agentStepLimit.test.ts`
- 行数：1124
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/agentStepLimit.test.ts`。体量较大（约 1124 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/query` 的职责见上文分类。

#### `agentStepLimit.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/agentStepLimit.ts`
- 行数：2
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/query` 的职责见上文分类。

#### `autoCompactCooldown.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/autoCompactCooldown.test.ts`
- 行数：985
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/autoCompactCooldown.test.ts`。体量较大（约 985 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/query` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/config.ts`
- 行数：46
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：所在目录 `src/query` 的职责见上文分类。

#### `deps.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/deps.ts`
- 行数：46
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：所在目录 `src/query` 的职责见上文分类。

#### `goalContinuation.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/goalContinuation.test.ts`
- 行数：245
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/goalContinuation.test.ts`。所在目录 `src/query` 的职责见上文分类。

#### `providerMaxTokensCapRetry.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/providerMaxTokensCapRetry.test.ts`
- 行数：445
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/providerMaxTokensCapRetry.test.ts`。所在目录 `src/query` 的职责见上文分类。

#### `requestOnlyMessages.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/requestOnlyMessages.test.ts`
- 行数：468
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/requestOnlyMessages.test.ts`。所在目录 `src/query` 的职责见上文分类。

#### `stopHooks.goal.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/stopHooks.goal.test.ts`
- 行数：382
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/stopHooks.goal.test.ts`。所在目录 `src/query` 的职责见上文分类。

#### `stopHooks.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/stopHooks.test.ts`
- 行数：24
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/stopHooks.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/query` 的职责见上文分类。

#### `stopHooks.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/stopHooks.ts`
- 行数：588
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：所在目录 `src/query` 的职责见上文分类。

#### `tokenBudget.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/tokenBudget.ts`
- 行数：93
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：所在目录 `src/query` 的职责见上文分类。

#### `toolFailureLoopGuard.test.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/toolFailureLoopGuard.test.ts`
- 行数：1386
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：运行方式：`bun test ./src/query/toolFailureLoopGuard.test.ts`。体量较大（约 1386 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/query` 的职责见上文分类。

#### `toolFailureLoopGuard.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/toolFailureLoopGuard.ts`
- 行数：554
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：所在目录 `src/query` 的职责见上文分类。

#### `transitions.ts`

- 所在目录：`src/query`
- 完整路径：`src/query/transitions.ts`
- 行数：31
- 主要功能简介：查询循环拆出的辅助模块。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/query` 的职责见上文分类。

### 2.98 目录组 `src/query.abortClassification.test.ts`（1 个文件，约 285 行）

该组位于仓库相对路径 `src/query.abortClassification.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `query.abortClassification.test.ts`

- 所在目录：`src`
- 完整路径：`src/query.abortClassification.test.ts`
- 行数：285
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/query.abortClassification.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.99 目录组 `src/query.conversationArc.test.ts`（1 个文件，约 169 行）

该组位于仓库相对路径 `src/query.conversationArc.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `query.conversationArc.test.ts`

- 所在目录：`src`
- 完整路径：`src/query.conversationArc.test.ts`
- 行数：169
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/query.conversationArc.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.100 目录组 `src/query.ts`（1 个文件，约 3204 行）

该组位于仓库相对路径 `src/query.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `query.ts`

- 所在目录：`src`
- 完整路径：`src/query.ts`
- 行数：3204
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：体量较大（约 3204 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src` 的职责见上文分类。

### 2.101 目录组 `src/queryEngine.goal.test.ts`（1 个文件，约 83 行）

该组位于仓库相对路径 `src/queryEngine.goal.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `queryEngine.goal.test.ts`

- 所在目录：`src`
- 完整路径：`src/queryEngine.goal.test.ts`
- 行数：83
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/queryEngine.goal.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.102 目录组 `src/remote`（4 个文件，约 1219 行）

该组位于仓库相对路径 `src/remote`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `RemoteSessionManager.ts`

- 所在目录：`src/remote`
- 完整路径：`src/remote/RemoteSessionManager.ts`
- 行数：343
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/remote` 的职责见上文分类。

#### `SessionsWebSocket.ts`

- 所在目录：`src/remote`
- 完整路径：`src/remote/SessionsWebSocket.ts`
- 行数：408
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/remote` 的职责见上文分类。

#### `remotePermissionBridge.ts`

- 所在目录：`src/remote`
- 完整路径：`src/remote/remotePermissionBridge.ts`
- 行数：158
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/remote` 的职责见上文分类。

#### `sdkMessageAdapter.ts`

- 所在目录：`src/remote`
- 完整路径：`src/remote/sdkMessageAdapter.ts`
- 行数：310
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/remote` 的职责见上文分类。

### 2.103 目录组 `src/replLauncher.tsx`（1 个文件，约 22 行）

该组位于仓库相对路径 `src/replLauncher.tsx`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `replLauncher.tsx`

- 所在目录：`src`
- 完整路径：`src/replLauncher.tsx`
- 行数：22
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.104 目录组 `src/schemas`（1 个文件，约 222 行）

该组位于仓库相对路径 `src/schemas`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `hooks.ts`

- 所在目录：`src/schemas`
- 完整路径：`src/schemas/hooks.ts`
- 行数：222
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/schemas` 的职责见上文分类。

### 2.105 目录组 `src/screens`（24 个文件，约 8485 行）

该组位于仓库相对路径 `src/screens`。全屏界面。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `Doctor.tsx`

- 所在目录：`src/screens`
- 完整路径：`src/screens/Doctor.tsx`
- 行数：561
- 主要功能简介：全屏界面。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/screens` 的职责见上文分类。

#### `REPL.queryLifecycle.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/REPL.queryLifecycle.test.ts`
- 行数：182
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/REPL.queryLifecycle.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `REPL.tsx`

- 所在目录：`src/screens`
- 完整路径：`src/screens/REPL.tsx`
- 行数：5633
- 主要功能简介：全屏界面。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 5633 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/screens` 的职责见上文分类。

#### `ResumeConversation.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/ResumeConversation.test.ts`
- 行数：65
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/ResumeConversation.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `ResumeConversation.tsx`

- 所在目录：`src/screens`
- 完整路径：`src/screens/ResumeConversation.tsx`
- 行数：400
- 主要功能简介：全屏界面。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/screens` 的职责见上文分类。

#### `doctorDiagnosticLoad.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/doctorDiagnosticLoad.test.ts`
- 行数：59
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/doctorDiagnosticLoad.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `doctorDiagnosticLoad.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/doctorDiagnosticLoad.ts`
- 行数：23
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `doctorDistTags.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/doctorDistTags.test.ts`
- 行数：88
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/doctorDistTags.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `doctorDistTags.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/doctorDistTags.ts`
- 行数：31
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `replActiveAgentModel.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replActiveAgentModel.test.ts`
- 行数：64
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replActiveAgentModel.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `replActiveAgentModel.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replActiveAgentModel.ts`
- 行数：49
- 主要功能简介：全屏界面。
- 说明：所在目录 `src/screens` 的职责见上文分类。

#### `replFallbackModelProp.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replFallbackModelProp.test.ts`
- 行数：231
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replFallbackModelProp.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `replFocusedInputDialog.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replFocusedInputDialog.test.ts`
- 行数：63
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replFocusedInputDialog.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `replFocusedInputDialog.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replFocusedInputDialog.ts`
- 行数：38
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `replInputSuppression.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replInputSuppression.test.ts`
- 行数：18
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replInputSuppression.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `replInputSuppression.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replInputSuppression.ts`
- 行数：6
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `replMaxTurns.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replMaxTurns.ts`
- 行数：189
- 主要功能简介：全屏界面。
- 说明：所在目录 `src/screens` 的职责见上文分类。

#### `replMaxTurnsProp.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replMaxTurnsProp.test.ts`
- 行数：481
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replMaxTurnsProp.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `replStartupGates.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replStartupGates.test.ts`
- 行数：53
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replStartupGates.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `replStartupGates.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replStartupGates.ts`
- 行数：35
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `replStreamingTextClear.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/replStreamingTextClear.test.ts`
- 行数：33
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/replStreamingTextClear.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `resumeFilters.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/resumeFilters.ts`
- 行数：36
- 主要功能简介：全屏界面。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/screens` 的职责见上文分类。

#### `streamingTextPublish.test.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/streamingTextPublish.test.ts`
- 行数：102
- 主要功能简介：全屏界面。
- 说明：运行方式：`bun test ./src/screens/streamingTextPublish.test.ts`。所在目录 `src/screens` 的职责见上文分类。

#### `streamingTextPublish.ts`

- 所在目录：`src/screens`
- 完整路径：`src/screens/streamingTextPublish.ts`
- 行数：45
- 主要功能简介：全屏界面。
- 说明：所在目录 `src/screens` 的职责见上文分类。

### 2.106 目录组 `src/self-hosted-runner`（1 个文件，约 13 行）

该组位于仓库相对路径 `src/self-hosted-runner`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `main.ts`

- 所在目录：`src/self-hosted-runner`
- 完整路径：`src/self-hosted-runner/main.ts`
- 行数：13
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/self-hosted-runner` 的职责见上文分类。

### 2.107 目录组 `src/server`（11 个文件，约 553 行）

该组位于仓库相对路径 `src/server`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `dangerousBackend.ts`

- 所在目录：`src/server/backends`
- 完整路径：`src/server/backends/dangerousBackend.ts`
- 行数：9
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server/backends` 的职责见上文分类。

#### `connectHeadless.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/connectHeadless.ts`
- 行数：18
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server` 的职责见上文分类。

#### `createDirectConnectSession.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/createDirectConnectSession.ts`
- 行数：88
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/server` 的职责见上文分类。

#### `directConnectManager.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/directConnectManager.ts`
- 行数：213
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/server` 的职责见上文分类。

#### `lockfile.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/lockfile.ts`
- 行数：25
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server` 的职责见上文分类。

#### `parseConnectUrl.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/parseConnectUrl.ts`
- 行数：51
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/server` 的职责见上文分类。

#### `server.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/server.ts`
- 行数：41
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/server` 的职责见上文分类。

#### `serverBanner.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/serverBanner.ts`
- 行数：16
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server` 的职责见上文分类。

#### `serverLog.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/serverLog.ts`
- 行数：14
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server` 的职责见上文分类。

#### `sessionManager.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/sessionManager.ts`
- 行数：21
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/server` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/server`
- 完整路径：`src/server/types.ts`
- 行数：57
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/server` 的职责见上文分类。

### 2.108 目录组 `src/services`（369 个文件，约 140340 行）

该组位于仓库相对路径 `src/services`。领域服务（记忆、分析、LSP、插件、wiki 等）。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `agentSummary.test.ts`

- 所在目录：`src/services/AgentSummary`
- 完整路径：`src/services/AgentSummary/agentSummary.test.ts`
- 行数：98
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/AgentSummary/agentSummary.test.ts`。所在目录 `src/services/AgentSummary` 的职责见上文分类。

#### `agentSummary.ts`

- 所在目录：`src/services/AgentSummary`
- 完整路径：`src/services/AgentSummary/agentSummary.ts`
- 行数：182
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/AgentSummary` 的职责见上文分类。

#### `magicDocs.ts`

- 所在目录：`src/services/MagicDocs`
- 完整路径：`src/services/MagicDocs/magicDocs.ts`
- 行数：254
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/MagicDocs` 的职责见上文分类。

#### `prompts.ts`

- 所在目录：`src/services/MagicDocs`
- 完整路径：`src/services/MagicDocs/prompts.ts`
- 行数：127
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/services/MagicDocs` 的职责见上文分类。

#### `promptSuggestion.ts`

- 所在目录：`src/services/PromptSuggestion`
- 完整路径：`src/services/PromptSuggestion/promptSuggestion.ts`
- 行数：523
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/services/PromptSuggestion` 的职责见上文分类。

#### `speculation.test.ts`

- 所在目录：`src/services/PromptSuggestion`
- 完整路径：`src/services/PromptSuggestion/speculation.test.ts`
- 行数：144
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/PromptSuggestion/speculation.test.ts`。所在目录 `src/services/PromptSuggestion` 的职责见上文分类。

#### `speculation.ts`

- 所在目录：`src/services/PromptSuggestion`
- 完整路径：`src/services/PromptSuggestion/speculation.ts`
- 行数：1036
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 1036 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/PromptSuggestion` 的职责见上文分类。

#### `prompts.ts`

- 所在目录：`src/services/SessionMemory`
- 完整路径：`src/services/SessionMemory/prompts.ts`
- 行数：324
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/services/SessionMemory` 的职责见上文分类。

#### `sessionMemory.ts`

- 所在目录：`src/services/SessionMemory`
- 完整路径：`src/services/SessionMemory/sessionMemory.ts`
- 行数：495
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/SessionMemory` 的职责见上文分类。

#### `sessionMemoryUtils.ts`

- 所在目录：`src/services/SessionMemory`
- 完整路径：`src/services/SessionMemory/sessionMemoryUtils.ts`
- 行数：207
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/SessionMemory` 的职责见上文分类。

#### `ads.test.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/ads.test.ts`
- 行数：127
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/ads.test.ts`。所在目录 `src/services` 的职责见上文分类。

#### `ads.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/ads.ts`
- 行数：177
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/config.ts`
- 行数：33
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/analytics` 的职责见上文分类。

#### `datadog.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/datadog.ts`
- 行数：8
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/analytics` 的职责见上文分类。

#### `growthbook.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/growthbook.ts`
- 行数：190
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/analytics` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/index.ts`
- 行数：74
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/analytics` 的职责见上文分类。

#### `metadata.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/metadata.ts`
- 行数：973
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 973 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/analytics` 的职责见上文分类。

#### `sink.ts`

- 所在目录：`src/services/analytics`
- 完整路径：`src/services/analytics/sink.ts`
- 行数：12
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/analytics` 的职责见上文分类。

#### `adminRequests.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/adminRequests.ts`
- 行数：119
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `agentRouteSettings.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/agentRouteSettings.test.ts`
- 行数：477
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/agentRouteSettings.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `agentRouteSettings.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/agentRouteSettings.ts`
- 行数：378
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `agentRouting.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/agentRouting.test.ts`
- 行数：875
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/agentRouting.test.ts`。体量较大（约 875 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `agentRouting.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/agentRouting.ts`
- 行数：374
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `authRouting.attribution.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/authRouting.attribution.test.ts`
- 行数：218
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/authRouting.attribution.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `authRouting.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/authRouting.ts`
- 行数：248
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `bootstrap.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/bootstrap.test.ts`
- 行数：301
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/bootstrap.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `bootstrap.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/bootstrap.ts`
- 行数：355
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `cacheMetrics.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/cacheMetrics.test.ts`
- 行数：810
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/cacheMetrics.test.ts`。体量较大（约 810 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `cacheMetrics.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/cacheMetrics.ts`
- 行数：555
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `cacheMetricsIntegration.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/cacheMetricsIntegration.test.ts`
- 行数：339
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/cacheMetricsIntegration.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `cacheStatsTracker.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/cacheStatsTracker.test.ts`
- 行数：224
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/cacheStatsTracker.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `cacheStatsTracker.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/cacheStatsTracker.ts`
- 行数：179
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `claude.abortClassification.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/claude.abortClassification.test.ts`
- 行数：131
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/claude.abortClassification.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `claude.lifecycle.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/claude.lifecycle.test.ts`
- 行数：1392
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/claude.lifecycle.test.ts`。体量较大（约 1392 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `claude.streamWatchdog.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/claude.streamWatchdog.test.ts`
- 行数：663
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/claude.streamWatchdog.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `claude.toolHistoryRouting.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/claude.toolHistoryRouting.test.ts`
- 行数：87
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/claude.toolHistoryRouting.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `claude.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/claude.ts`
- 行数：3979
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 3979 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `client.oauthRouting.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/client.oauthRouting.test.ts`
- 行数：79
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/client.oauthRouting.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `client.optionalRuntime.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/client.optionalRuntime.test.ts`
- 行数：148
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/client.optionalRuntime.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `client.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/client.test.ts`
- 行数：3092
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/client.test.ts`。体量较大（约 3092 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `client.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/client.ts`
- 行数：1031
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1031 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `clinepassUsage.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/clinepassUsage.test.ts`
- 行数：190
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/clinepassUsage.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `clinepassUsage.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/clinepassUsage.ts`
- 行数：16
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `fetch.ts`

- 所在目录：`src/services/api/clinepassUsage`
- 完整路径：`src/services/api/clinepassUsage/fetch.ts`
- 行数：80
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/clinepassUsage` 的职责见上文分类。

#### `parse.ts`

- 所在目录：`src/services/api/clinepassUsage`
- 完整路径：`src/services/api/clinepassUsage/parse.ts`
- 行数：130
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/clinepassUsage` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/api/clinepassUsage`
- 完整路径：`src/services/api/clinepassUsage/types.ts`
- 行数：30
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api/clinepassUsage` 的职责见上文分类。

#### `codexOAuth.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexOAuth.test.ts`
- 行数：434
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/codexOAuth.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `codexOAuth.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexOAuth.ts`
- 行数：456
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `codexOAuthShared.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexOAuthShared.test.ts`
- 行数：38
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/codexOAuthShared.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `codexOAuthShared.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexOAuthShared.ts`
- 行数：174
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `codexShim.interruption.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexShim.interruption.test.ts`
- 行数：841
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/codexShim.interruption.test.ts`。体量较大（约 841 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `codexShim.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexShim.test.ts`
- 行数：1484
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/codexShim.test.ts`。体量较大（约 1484 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `codexShim.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexShim.ts`
- 行数：1378
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1378 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `codexUsage.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexUsage.test.ts`
- 行数：204
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/codexUsage.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `codexUsage.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/codexUsage.ts`
- 行数：463
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `compressToolHistory.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/compressToolHistory.test.ts`
- 行数：925
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/compressToolHistory.test.ts`。体量较大（约 925 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `compressToolHistory.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/compressToolHistory.ts`
- 行数：486
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `credentialPool.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/credentialPool.test.ts`
- 行数：94
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/credentialPool.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `credentialPool.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/credentialPool.ts`
- 行数：170
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `dumpPrompts.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/dumpPrompts.ts`
- 行数：26
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `emptyUsage.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/emptyUsage.ts`
- 行数：22
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `errorUtils.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/errorUtils.ts`
- 行数：290
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `errors.openaiCompatibility.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/errors.openaiCompatibility.test.ts`
- 行数：148
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/errors.openaiCompatibility.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `errors.opencodeGo.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/errors.opencodeGo.test.ts`
- 行数：324
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/errors.opencodeGo.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `errors.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/errors.ts`
- 行数：1654
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1654 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `fetchWithProxyRetry.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/fetchWithProxyRetry.test.ts`
- 行数：262
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/fetchWithProxyRetry.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `fetchWithProxyRetry.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/fetchWithProxyRetry.ts`
- 行数：84
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `filesApi.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/filesApi.ts`
- 行数：748
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `firstTokenDate.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/firstTokenDate.ts`
- 行数：60
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `geminiVertexClient.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/geminiVertexClient.test.ts`
- 行数：551
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/geminiVertexClient.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `geminiVertexClient.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/geminiVertexClient.ts`
- 行数：803
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 803 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `grove.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/grove.test.ts`
- 行数：106
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/grove.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `grove.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/grove.ts`
- 行数：379
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `logging.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/logging.ts`
- 行数：700
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `minimaxUsage.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/minimaxUsage.test.ts`
- 行数：334
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/minimaxUsage.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `minimaxUsage.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/minimaxUsage.ts`
- 行数：17
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `fetch.ts`

- 所在目录：`src/services/api/minimaxUsage`
- 完整路径：`src/services/api/minimaxUsage/fetch.ts`
- 行数：150
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/minimaxUsage` 的职责见上文分类。

#### `parse.ts`

- 所在目录：`src/services/api/minimaxUsage`
- 完整路径：`src/services/api/minimaxUsage/parse.ts`
- 行数：540
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/minimaxUsage` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/api/minimaxUsage`
- 完整路径：`src/services/api/minimaxUsage/types.ts`
- 行数：43
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/minimaxUsage` 的职责见上文分类。

#### `openaiErrorClassification.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiErrorClassification.test.ts`
- 行数：535
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiErrorClassification.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiErrorClassification.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiErrorClassification.ts`
- 行数：634
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `openaiSchemaSanitizer.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiSchemaSanitizer.ts`
- 行数：1
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.architecture.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.architecture.test.ts`
- 行数：52
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim.architecture.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.compression.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.compression.test.ts`
- 行数：1468
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim.compression.test.ts`。体量较大（约 1468 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.diagnostics.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.diagnostics.test.ts`
- 行数：329
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim.diagnostics.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.ollamaTextToolCalls.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.ollamaTextToolCalls.test.ts`
- 行数：657
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim.ollamaTextToolCalls.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.test.ts`
- 行数：6419
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim.test.ts`。体量较大（约 6419 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `openaiShim.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/openaiShim.ts`
- 行数：586
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `clientDispatch.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/clientDispatch.test.ts`
- 行数：389
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/clientDispatch.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `clientDispatch.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/clientDispatch.ts`
- 行数：365
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `codexDispatch.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/codexDispatch.test.ts`
- 行数：289
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/codexDispatch.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `codexDispatch.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/codexDispatch.ts`
- 行数：214
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `geminiStreamConversion.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/geminiStreamConversion.test.ts`
- 行数：146
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/geminiStreamConversion.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `geminiStreamConversion.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/geminiStreamConversion.ts`
- 行数：314
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `markerEchoGuard.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/markerEchoGuard.test.ts`
- 行数：119
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/markerEchoGuard.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `markerEchoGuard.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/markerEchoGuard.ts`
- 行数：126
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `messageConversion.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/messageConversion.test.ts`
- 行数：350
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/messageConversion.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `messageConversion.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/messageConversion.ts`
- 行数：290
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `ollamaAdapter.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/ollamaAdapter.test.ts`
- 行数：321
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/ollamaAdapter.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `ollamaAdapter.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/ollamaAdapter.ts`
- 行数：263
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `providerCompatibility.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/providerCompatibility.test.ts`
- 行数：610
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/providerCompatibility.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `providerCompatibility.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/providerCompatibility.ts`
- 行数：149
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `providerStreamInterruptionTrace.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/providerStreamInterruptionTrace.test.ts`
- 行数：366
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/providerStreamInterruptionTrace.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `rawToolCallParsing.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/rawToolCallParsing.test.ts`
- 行数：133
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/rawToolCallParsing.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `rawToolCallParsing.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/rawToolCallParsing.ts`
- 行数：247
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestExecutor.integration.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestExecutor.integration.test.ts`
- 行数：4798
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/requestExecutor.integration.test.ts`。体量较大（约 4798 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestExecutor.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestExecutor.test.ts`
- 行数：2322
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/requestExecutor.test.ts`。体量较大（约 2322 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestExecutor.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestExecutor.ts`
- 行数：1188
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1188 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestPlanner.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestPlanner.test.ts`
- 行数：425
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/requestPlanner.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestPlanner.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestPlanner.ts`
- 行数：461
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestPreparation.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestPreparation.test.ts`
- 行数：128
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/requestPreparation.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `requestPreparation.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/requestPreparation.ts`
- 行数：404
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `responseAdapters.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/responseAdapters.test.ts`
- 行数：195
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/responseAdapters.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `responseAdapters.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/responseAdapters.ts`
- 行数：214
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `responseConversion.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/responseConversion.test.ts`
- 行数：228
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/responseConversion.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `responseConversion.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/responseConversion.ts`
- 行数：119
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `streamControl.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/streamControl.test.ts`
- 行数：373
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/streamControl.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `streamControl.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/streamControl.ts`
- 行数：312
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `streamConversion.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/streamConversion.test.ts`
- 行数：612
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/streamConversion.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `streamConversion.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/streamConversion.ts`
- 行数：1150
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1150 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `toolConversion.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/toolConversion.test.ts`
- 行数：116
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/toolConversion.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `toolConversion.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/toolConversion.ts`
- 行数：105
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `transport.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/transport.test.ts`
- 行数：155
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/transport.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `transport.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/transport.ts`
- 行数：443
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `xmlToolCallParsing.test.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/xmlToolCallParsing.test.ts`
- 行数：530
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/openaiShim/xmlToolCallParsing.test.ts`。所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `xmlToolCallParsing.ts`

- 所在目录：`src/services/api/openaiShim`
- 完整路径：`src/services/api/openaiShim/xmlToolCallParsing.ts`
- 行数：251
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/openaiShim` 的职责见上文分类。

#### `overageCreditGrant.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/overageCreditGrant.ts`
- 行数：137
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `promptCacheBreakDetection.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/promptCacheBreakDetection.test.ts`
- 行数：575
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/services/api/promptCacheBreakDetection.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `promptCacheBreakDetection.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/promptCacheBreakDetection.ts`
- 行数：1027
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。体量较大（约 1027 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.codexSecureStorage.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.codexSecureStorage.test.ts`
- 行数：243
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.codexSecureStorage.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.envDiagnostics.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.envDiagnostics.test.ts`
- 行数：161
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.envDiagnostics.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.github.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.github.test.ts`
- 行数：139
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.github.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.local.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.local.test.ts`
- 行数：560
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.local.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.localFastPath.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.localFastPath.test.ts`
- 行数：109
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.localFastPath.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.protoAlias.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.protoAlias.test.ts`
- 行数：134
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.protoAlias.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.runtimeCodexCredentials.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.runtimeCodexCredentials.test.ts`
- 行数：128
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.runtimeCodexCredentials.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.test.ts`
- 行数：528
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerConfig.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `providerConfig.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerConfig.ts`
- 行数：1547
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1547 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `providerMaxTokensCap.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/providerMaxTokensCap.test.ts`
- 行数：143
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/providerMaxTokensCap.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `referral.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/referral.ts`
- 行数：283
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `sessionIngress.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/sessionIngress.ts`
- 行数：514
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `smartModelRouting.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/smartModelRouting.test.ts`
- 行数：191
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/smartModelRouting.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `smartModelRouting.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/smartModelRouting.ts`
- 行数：225
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/index.test.ts`
- 行数：395
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/services/api/smartRouting/index.test.ts`。所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/index.ts`
- 行数：295
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `resolveConfig.test.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/resolveConfig.test.ts`
- 行数：93
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/smartRouting/resolveConfig.test.ts`。所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `resolveConfig.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/resolveConfig.ts`
- 行数：83
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `settings.test.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/settings.test.ts`
- 行数：111
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/smartRouting/settings.test.ts`。所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `settings.ts`

- 所在目录：`src/services/api/smartRouting`
- 完整路径：`src/services/api/smartRouting/settings.ts`
- 行数：83
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api/smartRouting` 的职责见上文分类。

#### `thinkTagSanitizer.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/thinkTagSanitizer.test.ts`
- 行数：183
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/thinkTagSanitizer.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `thinkTagSanitizer.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/thinkTagSanitizer.ts`
- 行数：162
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `toolArgumentNormalization.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/toolArgumentNormalization.test.ts`
- 行数：221
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/toolArgumentNormalization.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `toolArgumentNormalization.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/toolArgumentNormalization.ts`
- 行数：77
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `ultrareviewQuota.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/ultrareviewQuota.ts`
- 行数：38
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/api` 的职责见上文分类。

#### `usage.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/usage.ts`
- 行数：67
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `vertexClient.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/vertexClient.test.ts`
- 行数：248
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/vertexClient.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `vertexClient.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/vertexClient.ts`
- 行数：286
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `withRetry.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/withRetry.test.ts`
- 行数：872
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/withRetry.test.ts`。体量较大（约 872 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `withRetry.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/withRetry.ts`
- 行数：1093
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：体量较大（约 1093 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/api` 的职责见上文分类。

#### `xaiOAuth.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/xaiOAuth.ts`
- 行数：632
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `xaiOAuthCallback.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/xaiOAuthCallback.test.ts`
- 行数：424
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/xaiOAuthCallback.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `xaiOAuthCallback.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/xaiOAuthCallback.ts`
- 行数：249
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `xaiOAuthShared.test.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/xaiOAuthShared.test.ts`
- 行数：145
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：运行方式：`bun test ./src/services/api/xaiOAuthShared.test.ts`。所在目录 `src/services/api` 的职责见上文分类。

#### `xaiOAuthShared.ts`

- 所在目录：`src/services/api`
- 完整路径：`src/services/api/xaiOAuthShared.ts`
- 行数：204
- 主要功能简介：模型运输层、OAuth、错误分类、用量。
- 说明：所在目录 `src/services/api` 的职责见上文分类。

#### `autoDream.ts`

- 所在目录：`src/services/autoDream`
- 完整路径：`src/services/autoDream/autoDream.ts`
- 行数：326
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/autoDream` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/services/autoDream`
- 完整路径：`src/services/autoDream/config.ts`
- 行数：21
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/autoDream` 的职责见上文分类。

#### `consolidationLock.ts`

- 所在目录：`src/services/autoDream`
- 完整路径：`src/services/autoDream/consolidationLock.ts`
- 行数：140
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/autoDream` 的职责见上文分类。

#### `consolidationPrompt.ts`

- 所在目录：`src/services/autoDream`
- 完整路径：`src/services/autoDream/consolidationPrompt.ts`
- 行数：65
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/autoDream` 的职责见上文分类。

#### `autoFixConfig.test.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixConfig.test.ts`
- 行数：106
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/autoFix/autoFixConfig.test.ts`。所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixConfig.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixConfig.ts`
- 行数：52
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixHook.test.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixHook.test.ts`
- 行数：63
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/autoFix/autoFixHook.test.ts`。所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixHook.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixHook.ts`
- 行数：25
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixIntegration.test.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixIntegration.test.ts`
- 行数：50
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/autoFix/autoFixIntegration.test.ts`。所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixRunner.test.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixRunner.test.ts`
- 行数：105
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/autoFix/autoFixRunner.test.ts`。所在目录 `src/services/autoFix` 的职责见上文分类。

#### `autoFixRunner.ts`

- 所在目录：`src/services/autoFix`
- 完整路径：`src/services/autoFix/autoFixRunner.ts`
- 行数：186
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/autoFix` 的职责见上文分类。

#### `awaySummary.test.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/awaySummary.test.ts`
- 行数：182
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/awaySummary.test.ts`。所在目录 `src/services` 的职责见上文分类。

#### `awaySummary.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/awaySummary.ts`
- 行数：88
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `claudeAiLimits.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/claudeAiLimits.ts`
- 行数：520
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `claudeAiLimitsHook.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/claudeAiLimitsHook.ts`
- 行数：23
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services` 的职责见上文分类。

#### `apiMicrocompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/apiMicrocompact.ts`
- 行数：153
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `autoCompact.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/autoCompact.test.ts`
- 行数：980
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/autoCompact.test.ts`。体量较大（约 980 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/compact` 的职责见上文分类。

#### `autoCompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/autoCompact.ts`
- 行数：621
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `cachedMCConfig.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/cachedMCConfig.ts`
- 行数：1
- 主要功能简介：上下文压缩与微压缩。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/compact` 的职责见上文分类。

#### `cachedMicrocompact.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/cachedMicrocompact.test.ts`
- 行数：65
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/cachedMicrocompact.test.ts`。所在目录 `src/services/compact` 的职责见上文分类。

#### `cachedMicrocompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/cachedMicrocompact.ts`
- 行数：80
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `compact.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/compact.test.ts`
- 行数：1058
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/compact.test.ts`。体量较大（约 1058 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/compact` 的职责见上文分类。

#### `compact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/compact.ts`
- 行数：1854
- 主要功能简介：上下文压缩与微压缩。
- 说明：体量较大（约 1854 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/compact` 的职责见上文分类。

#### `compactWarningHook.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/compactWarningHook.ts`
- 行数：16
- 主要功能简介：上下文压缩与微压缩。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/compact` 的职责见上文分类。

#### `compactWarningState.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/compactWarningState.ts`
- 行数：18
- 主要功能简介：上下文压缩与微压缩。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/compact` 的职责见上文分类。

#### `grouping.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/grouping.ts`
- 行数：63
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `microCompact.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/microCompact.test.ts`
- 行数：239
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/microCompact.test.ts`。所在目录 `src/services/compact` 的职责见上文分类。

#### `microCompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/microCompact.ts`
- 行数：552
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `postCompactCleanup.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/postCompactCleanup.ts`
- 行数：75
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/prompt.ts`
- 行数：374
- 主要功能简介：上下文压缩与微压缩。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/services/compact` 的职责见上文分类。

#### `reactiveCompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/reactiveCompact.ts`
- 行数：84
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `resumeCompactPrompt.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/resumeCompactPrompt.test.ts`
- 行数：73
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/resumeCompactPrompt.test.ts`。所在目录 `src/services/compact` 的职责见上文分类。

#### `resumeCompactPrompt.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/resumeCompactPrompt.ts`
- 行数：33
- 主要功能简介：上下文压缩与微压缩。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/compact` 的职责见上文分类。

#### `sessionMemoryCompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/sessionMemoryCompact.ts`
- 行数：630
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `snipCompact.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/snipCompact.test.ts`
- 行数：391
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/snipCompact.test.ts`。所在目录 `src/services/compact` 的职责见上文分类。

#### `snipCompact.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/snipCompact.ts`
- 行数：281
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `snipProjection.test.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/snipProjection.test.ts`
- 行数：82
- 主要功能简介：上下文压缩与微压缩。
- 说明：运行方式：`bun test ./src/services/compact/snipProjection.test.ts`。所在目录 `src/services/compact` 的职责见上文分类。

#### `snipProjection.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/snipProjection.ts`
- 行数：22
- 主要功能简介：上下文压缩与微压缩。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/compact` 的职责见上文分类。

#### `timeBasedMCConfig.ts`

- 所在目录：`src/services/compact`
- 完整路径：`src/services/compact/timeBasedMCConfig.ts`
- 行数：43
- 主要功能简介：上下文压缩与微压缩。
- 说明：所在目录 `src/services/compact` 的职责见上文分类。

#### `collapseUtils.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/collapseUtils.test.ts`
- 行数：163
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/contextCollapse/collapseUtils.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `collapseUtils.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/collapseUtils.ts`
- 行数：70
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `ctxAgentPrompt.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/ctxAgentPrompt.ts`
- 行数：19
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/index.test.ts`
- 行数：641
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/services/contextCollapse/index.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/index.ts`
- 行数：618
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `operations.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/operations.test.ts`
- 行数：281
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/contextCollapse/operations.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `operations.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/operations.ts`
- 行数：55
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `persist.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/persist.test.ts`
- 行数：72
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/contextCollapse/persist.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `persist.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/persist.ts`
- 行数：16
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `spanSelection.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/spanSelection.test.ts`
- 行数：107
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/contextCollapse/spanSelection.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `spanSelection.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/spanSelection.ts`
- 行数：117
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `spawnCtxAgent.test.ts`

- 所在目录：`src/services/contextCollapse`
- 完整路径：`src/services/contextCollapse/spawnCtxAgent.test.ts`
- 行数：201
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/contextCollapse/spawnCtxAgent.test.ts`。所在目录 `src/services/contextCollapse` 的职责见上文分类。

#### `diagnosticTracking.test.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/diagnosticTracking.test.ts`
- 行数：152
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/diagnosticTracking.test.ts`。所在目录 `src/services` 的职责见上文分类。

#### `diagnosticTracking.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/diagnosticTracking.ts`
- 行数：439
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `extractMemories.abort.test.ts`

- 所在目录：`src/services/extractMemories`
- 完整路径：`src/services/extractMemories/extractMemories.abort.test.ts`
- 行数：383
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/extractMemories/extractMemories.abort.test.ts`。所在目录 `src/services/extractMemories` 的职责见上文分类。

#### `extractMemories.ts`

- 所在目录：`src/services/extractMemories`
- 完整路径：`src/services/extractMemories/extractMemories.ts`
- 行数：668
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/extractMemories` 的职责见上文分类。

#### `prompts.ts`

- 所在目录：`src/services/extractMemories`
- 完整路径：`src/services/extractMemories/prompts.ts`
- 行数：160
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/services/extractMemories` 的职责见上文分类。

#### `deviceFlow.test.ts`

- 所在目录：`src/services/github`
- 完整路径：`src/services/github/deviceFlow.test.ts`
- 行数：290
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/github/deviceFlow.test.ts`。所在目录 `src/services/github` 的职责见上文分类。

#### `deviceFlow.ts`

- 所在目录：`src/services/github`
- 完整路径：`src/services/github/deviceFlow.ts`
- 行数：324
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/github` 的职责见上文分类。

#### `controller.test.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/controller.test.ts`
- 行数：598
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/goal/controller.test.ts`。所在目录 `src/services/goal` 的职责见上文分类。

#### `controller.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/controller.ts`
- 行数：273
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/goal` 的职责见上文分类。

#### `evaluator.test.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/evaluator.test.ts`
- 行数：250
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/goal/evaluator.test.ts`。所在目录 `src/services/goal` 的职责见上文分类。

#### `evaluator.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/evaluator.ts`
- 行数：290
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/goal` 的职责见上文分类。

#### `instructions.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/instructions.ts`
- 行数：31
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/goal` 的职责见上文分类。

#### `persistence.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/persistence.ts`
- 行数：14
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/goal` 的职责见上文分类。

#### `sdk.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/sdk.ts`
- 行数：8
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/goal` 的职责见上文分类。

#### `state.test.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/state.test.ts`
- 行数：136
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/goal/state.test.ts`。所在目录 `src/services/goal` 的职责见上文分类。

#### `state.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/state.ts`
- 行数：181
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/goal` 的职责见上文分类。

#### `status.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/status.ts`
- 行数：18
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/goal` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/goal`
- 完整路径：`src/services/goal/types.ts`
- 行数：32
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/goal` 的职责见上文分类。

#### `internalLogging.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/internalLogging.ts`
- 行数：9
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services` 的职责见上文分类。

#### `LSPClient.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/LSPClient.ts`
- 行数：447
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `LSPDiagnosticRegistry.test.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/LSPDiagnosticRegistry.test.ts`
- 行数：668
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/lsp/LSPDiagnosticRegistry.test.ts`。所在目录 `src/services/lsp` 的职责见上文分类。

#### `LSPDiagnosticRegistry.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/LSPDiagnosticRegistry.ts`
- 行数：1031
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 1031 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/lsp` 的职责见上文分类。

#### `LSPServerInstance.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/LSPServerInstance.ts`
- 行数：511
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `LSPServerManager.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/LSPServerManager.ts`
- 行数：429
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/config.ts`
- 行数：79
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `manager.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/manager.ts`
- 行数：289
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `passiveFeedback.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/passiveFeedback.ts`
- 行数：316
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/lsp`
- 完整路径：`src/services/lsp/types.ts`
- 行数：58
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/lsp` 的职责见上文分类。

#### `InProcessTransport.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/InProcessTransport.ts`
- 行数：63
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `MCPConnectionManager.tsx`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/MCPConnectionManager.tsx`
- 行数：72
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/services/mcp` 的职责见上文分类。

#### `SdkControlTransport.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/SdkControlTransport.ts`
- 行数：136
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `auth.refreshLock.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/auth.refreshLock.test.ts`
- 行数：2131
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/auth.refreshLock.test.ts`。体量较大（约 2131 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `auth.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/auth.test.ts`
- 行数：61
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/auth.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `auth.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/auth.ts`
- 行数：2863
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：体量较大（约 2863 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `channelAllowlist.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/channelAllowlist.ts`
- 行数：76
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `channelNotification.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/channelNotification.test.ts`
- 行数：612
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/channelNotification.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `channelNotification.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/channelNotification.ts`
- 行数：351
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `channelPermissions.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/channelPermissions.test.ts`
- 行数：29
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/channelPermissions.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/services/mcp` 的职责见上文分类。

#### `channelPermissions.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/channelPermissions.ts`
- 行数：255
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `claudeai.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/claudeai.ts`
- 行数：193
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `client.activity.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/client.activity.test.ts`
- 行数：703
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/client.activity.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `client.pagination.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/client.pagination.test.ts`
- 行数：950
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/client.pagination.test.ts`。体量较大（约 950 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `client.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/client.test.ts`
- 行数：397
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/client.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `client.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/client.ts`
- 行数：3667
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：体量较大（约 3667 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/config.ts`
- 行数：1578
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：体量较大（约 1578 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `doctor.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/doctor.test.ts`
- 行数：546
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/doctor.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `doctor.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/doctor.ts`
- 行数：703
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `elicitationHandler.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/elicitationHandler.ts`
- 行数：313
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `envExpansion.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/envExpansion.test.ts`
- 行数：49
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/envExpansion.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `envExpansion.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/envExpansion.ts`
- 行数：47
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `headersHelper.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/headersHelper.ts`
- 行数：139
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `mcpStringUtils.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/mcpStringUtils.ts`
- 行数：106
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `normalization.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/normalization.ts`
- 行数：23
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/mcp` 的职责见上文分类。

#### `oauthPort.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/oauthPort.ts`
- 行数：78
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `officialRegistry.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/officialRegistry.test.ts`
- 行数：96
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/officialRegistry.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `officialRegistry.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/officialRegistry.ts`
- 行数：86
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `pagination.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/pagination.ts`
- 行数：115
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `refreshLock.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/refreshLock.ts`
- 行数：241
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/types.ts`
- 行数：258
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `useManageMCPConnections.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/useManageMCPConnections.ts`
- 行数：1164
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：体量较大（约 1164 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/mcp` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/utils.ts`
- 行数：575
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `vscodeSdkMcp.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/vscodeSdkMcp.ts`
- 行数：112
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `xaa.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/xaa.test.ts`
- 行数：123
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/xaa.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `xaa.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/xaa.ts`
- 行数：497
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `xaaIdpCallback.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/xaaIdpCallback.ts`
- 行数：52
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `xaaIdpLogin.test.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/xaaIdpLogin.test.ts`
- 行数：98
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：运行方式：`bun test ./src/services/mcp/xaaIdpLogin.test.ts`。所在目录 `src/services/mcp` 的职责见上文分类。

#### `xaaIdpLogin.ts`

- 所在目录：`src/services/mcp`
- 完整路径：`src/services/mcp/xaaIdpLogin.ts`
- 行数：515
- 主要功能简介：MCP 客户端/宿主集成。
- 说明：所在目录 `src/services/mcp` 的职责见上文分类。

#### `mcpServerApproval.tsx`

- 所在目录：`src/services`
- 完整路径：`src/services/mcpServerApproval.tsx`
- 行数：40
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/services` 的职责见上文分类。

#### `mockRateLimits.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/mockRateLimits.ts`
- 行数：205
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `notifier.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/notifier.ts`
- 行数：156
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `auth-code-listener.analytics.test.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/auth-code-listener.analytics.test.ts`
- 行数：167
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/oauth/auth-code-listener.analytics.test.ts`。所在目录 `src/services/oauth` 的职责见上文分类。

#### `auth-code-listener.test.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/auth-code-listener.test.ts`
- 行数：29
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/oauth/auth-code-listener.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/services/oauth` 的职责见上文分类。

#### `auth-code-listener.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/auth-code-listener.ts`
- 行数：275
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/oauth` 的职责见上文分类。

#### `client.populateAccountInfo.test.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/client.populateAccountInfo.test.ts`
- 行数：42
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/oauth/client.populateAccountInfo.test.ts`。所在目录 `src/services/oauth` 的职责见上文分类。

#### `client.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/client.ts`
- 行数：589
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/oauth` 的职责见上文分类。

#### `crypto.test.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/crypto.test.ts`
- 行数：27
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/oauth/crypto.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/services/oauth` 的职责见上文分类。

#### `crypto.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/crypto.ts`
- 行数：23
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/oauth` 的职责见上文分类。

#### `getOauthProfile.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/getOauthProfile.ts`
- 行数：53
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/oauth` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/index.ts`
- 行数：198
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/oauth` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/oauth`
- 完整路径：`src/services/oauth/types.ts`
- 行数：131
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/oauth` 的职责见上文分类。

#### `PluginInstallationManager.ts`

- 所在目录：`src/services/plugins`
- 完整路径：`src/services/plugins/PluginInstallationManager.ts`
- 行数：184
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/plugins` 的职责见上文分类。

#### `pluginCliCommands.ts`

- 所在目录：`src/services/plugins`
- 完整路径：`src/services/plugins/pluginCliCommands.ts`
- 行数：344
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/plugins` 的职责见上文分类。

#### `pluginOperations.installLocation.test.ts`

- 所在目录：`src/services/plugins`
- 完整路径：`src/services/plugins/pluginOperations.installLocation.test.ts`
- 行数：75
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/plugins/pluginOperations.installLocation.test.ts`。所在目录 `src/services/plugins` 的职责见上文分类。

#### `pluginOperations.ts`

- 所在目录：`src/services/plugins`
- 完整路径：`src/services/plugins/pluginOperations.ts`
- 行数：1092
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 1092 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/plugins` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/policyLimits`
- 完整路径：`src/services/policyLimits/index.ts`
- 行数：663
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/policyLimits` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/policyLimits`
- 完整路径：`src/services/policyLimits/types.ts`
- 行数：27
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/policyLimits` 的职责见上文分类。

#### `preventSleep.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/preventSleep.ts`
- 行数：165
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `rateLimitMessages.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/rateLimitMessages.ts`
- 行数：344
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `rateLimitMocking.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/rateLimitMocking.ts`
- 行数：144
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/remoteManagedSettings`
- 完整路径：`src/services/remoteManagedSettings/index.ts`
- 行数：638
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/remoteManagedSettings` 的职责见上文分类。

#### `securityCheck.tsx`

- 所在目录：`src/services/remoteManagedSettings`
- 完整路径：`src/services/remoteManagedSettings/securityCheck.tsx`
- 行数：73
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/services/remoteManagedSettings` 的职责见上文分类。

#### `syncCache.ts`

- 所在目录：`src/services/remoteManagedSettings`
- 完整路径：`src/services/remoteManagedSettings/syncCache.ts`
- 行数：112
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/remoteManagedSettings` 的职责见上文分类。

#### `syncCacheState.ts`

- 所在目录：`src/services/remoteManagedSettings`
- 完整路径：`src/services/remoteManagedSettings/syncCacheState.ts`
- 行数：96
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/remoteManagedSettings` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/remoteManagedSettings`
- 完整路径：`src/services/remoteManagedSettings/types.ts`
- 行数：31
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/remoteManagedSettings` 的职责见上文分类。

#### `sessionTranscript.ts`

- 所在目录：`src/services/sessionTranscript`
- 完整路径：`src/services/sessionTranscript/sessionTranscript.ts`
- 行数：21
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/sessionTranscript` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/settingsSync`
- 完整路径：`src/services/settingsSync/index.ts`
- 行数：670
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/services/settingsSync` 的职责见上文分类。

#### `settings.transaction.test.ts`

- 所在目录：`src/services/settingsSync`
- 完整路径：`src/services/settingsSync/settings.transaction.test.ts`
- 行数：371
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/settingsSync/settings.transaction.test.ts`。所在目录 `src/services/settingsSync` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/settingsSync`
- 完整路径：`src/services/settingsSync/types.ts`
- 行数：67
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/settingsSync` 的职责见上文分类。

#### `featureCheck.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/featureCheck.ts`
- 行数：7
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `localSearch.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/localSearch.ts`
- 行数：7
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `prefetch.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/prefetch.ts`
- 行数：41
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `remoteSkillLoader.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/remoteSkillLoader.ts`
- 行数：29
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `remoteSkillState.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/remoteSkillState.ts`
- 行数：29
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `signals.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/signals.ts`
- 行数：14
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `telemetry.ts`

- 所在目录：`src/services/skillSearch`
- 完整路径：`src/services/skillSearch/telemetry.ts`
- 行数：15
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/skillSearch` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/index.test.ts`
- 行数：174
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/services/teamMemorySync/index.test.ts`。所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/index.ts`
- 行数：1354
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。体量较大（约 1354 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `secretScanner.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/secretScanner.ts`
- 行数：324
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `teamMemSecretGuard.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/teamMemSecretGuard.ts`
- 行数：44
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/types.ts`
- 行数：156
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `watcher.test.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/watcher.test.ts`
- 行数：270
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/teamMemorySync/watcher.test.ts`。所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `watcher.ts`

- 所在目录：`src/services/teamMemorySync`
- 完整路径：`src/services/teamMemorySync/watcher.ts`
- 行数：446
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/teamMemorySync` 的职责见上文分类。

#### `gitlawbEarn.test.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/gitlawbEarn.test.ts`
- 行数：111
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tips/gitlawbEarn.test.ts`。所在目录 `src/services/tips` 的职责见上文分类。

#### `gitlawbEarn.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/gitlawbEarn.ts`
- 行数：118
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tips` 的职责见上文分类。

#### `sponsoredTips.test.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/sponsoredTips.test.ts`
- 行数：218
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tips/sponsoredTips.test.ts`。所在目录 `src/services/tips` 的职责见上文分类。

#### `sponsoredTips.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/sponsoredTips.ts`
- 行数：173
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tips` 的职责见上文分类。

#### `tipHistory.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipHistory.ts`
- 行数：38
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/tips` 的职责见上文分类。

#### `tipLink.test.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipLink.test.ts`
- 行数：60
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tips/tipLink.test.ts`。所在目录 `src/services/tips` 的职责见上文分类。

#### `tipLink.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipLink.ts`
- 行数：73
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tips` 的职责见上文分类。

#### `tipRegistry.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipRegistry.ts`
- 行数：662
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tips` 的职责见上文分类。

#### `tipScheduler.test.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipScheduler.test.ts`
- 行数：225
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tips/tipScheduler.test.ts`。所在目录 `src/services/tips` 的职责见上文分类。

#### `tipScheduler.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/tipScheduler.ts`
- 行数：96
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tips` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/tips`
- 完整路径：`src/services/tips/types.ts`
- 行数：29
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/tips` 的职责见上文分类。

#### `tokenEstimation.test.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/tokenEstimation.test.ts`
- 行数：80
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tokenEstimation.test.ts`。所在目录 `src/services` 的职责见上文分类。

#### `tokenEstimation.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/tokenEstimation.ts`
- 行数：716
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `tokenModelCompression.test.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/tokenModelCompression.test.ts`
- 行数：100
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tokenModelCompression.test.ts`。所在目录 `src/services` 的职责见上文分类。

#### `toolUseSummaryGenerator.ts`

- 所在目录：`src/services/toolUseSummary`
- 完整路径：`src/services/toolUseSummary/toolUseSummaryGenerator.ts`
- 行数：112
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/toolUseSummary` 的职责见上文分类。

#### `StreamingToolExecutor.test.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/StreamingToolExecutor.test.ts`
- 行数：308
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tools/StreamingToolExecutor.test.ts`。所在目录 `src/services/tools` 的职责见上文分类。

#### `StreamingToolExecutor.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/StreamingToolExecutor.ts`
- 行数：616
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tools` 的职责见上文分类。

#### `queryActivityLease.test.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/queryActivityLease.test.ts`
- 行数：331
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tools/queryActivityLease.test.ts`。所在目录 `src/services/tools` 的职责见上文分类。

#### `queryActivityLease.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/queryActivityLease.ts`
- 行数：42
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tools` 的职责见上文分类。

#### `toolExecution.test.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolExecution.test.ts`
- 行数：797
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tools/toolExecution.test.ts`。所在目录 `src/services/tools` 的职责见上文分类。

#### `toolExecution.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolExecution.ts`
- 行数：1981
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 1981 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/tools` 的职责见上文分类。

#### `toolHooks.test.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolHooks.test.ts`
- 行数：717
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tools/toolHooks.test.ts`。所在目录 `src/services/tools` 的职责见上文分类。

#### `toolHooks.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolHooks.ts`
- 行数：817
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：体量较大（约 817 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/services/tools` 的职责见上文分类。

#### `toolOrchestration.test.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolOrchestration.test.ts`
- 行数：43
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/tools/toolOrchestration.test.ts`。所在目录 `src/services/tools` 的职责见上文分类。

#### `toolOrchestration.ts`

- 所在目录：`src/services/tools`
- 完整路径：`src/services/tools/toolOrchestration.ts`
- 行数：199
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/tools` 的职责见上文分类。

#### `vcr.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/vcr.ts`
- 行数：406
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `voice.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/voice.ts`
- 行数：525
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `voiceKeyterms.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/voiceKeyterms.ts`
- 行数：106
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `voiceStreamSTT.ts`

- 所在目录：`src/services`
- 完整路径：`src/services/voiceStreamSTT.ts`
- 行数：544
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services` 的职责见上文分类。

#### `conventions.test.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/conventions.test.ts`
- 行数：209
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/wiki/conventions.test.ts`。所在目录 `src/services/wiki` 的职责见上文分类。

#### `conventions.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/conventions.ts`
- 行数：462
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `identity.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/identity.ts`
- 行数：131
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `indexBuilder.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/indexBuilder.ts`
- 行数：68
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `ingest.test.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/ingest.test.ts`
- 行数：48
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/wiki/ingest.test.ts`。所在目录 `src/services/wiki` 的职责见上文分类。

#### `ingest.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/ingest.ts`
- 行数：93
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `init.test.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/init.test.ts`
- 行数：58
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/wiki/init.test.ts`。所在目录 `src/services/wiki` 的职责见上文分类。

#### `init.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/init.ts`
- 行数：172
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `paths.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/paths.ts`
- 行数：20
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/wiki` 的职责见上文分类。

#### `status.test.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/status.test.ts`
- 行数：59
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：运行方式：`bun test ./src/services/wiki/status.test.ts`。所在目录 `src/services/wiki` 的职责见上文分类。

#### `status.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/status.ts`
- 行数：96
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/types.ts`
- 行数：48
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：所在目录 `src/services/wiki` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/services/wiki`
- 完整路径：`src/services/wiki/utils.ts`
- 行数：36
- 主要功能简介：领域服务（记忆、分析、LSP、插件、wiki 等）。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/services/wiki` 的职责见上文分类。

### 2.109 目录组 `src/setup.ts`（1 个文件，约 439 行）

该组位于仓库相对路径 `src/setup.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `setup.ts`

- 所在目录：`src`
- 完整路径：`src/setup.ts`
- 行数：439
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.110 目录组 `src/skills`（23 个文件，约 5621 行）

该组位于仓库相对路径 `src/skills`。技能发现与内置技能。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `batch.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/batch.ts`
- 行数：126
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `claudeApi.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/claudeApi.ts`
- 行数：196
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `claudeApiContent.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/claudeApiContent.ts`
- 行数：75
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `claudeInChrome.test.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/claudeInChrome.test.ts`
- 行数：27
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/bundled/claudeInChrome.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `claudeInChrome.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/claudeInChrome.ts`
- 行数：34
- 主要功能简介：技能发现与内置技能。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `claudeInChromeAccess.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/claudeInChromeAccess.ts`
- 行数：22
- 主要功能简介：技能发现与内置技能。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `debug.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/debug.ts`
- 行数：107
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/index.ts`
- 行数：67
- 主要功能简介：技能发现与内置技能。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `keybindings.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/keybindings.ts`
- 行数：353
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `loop.test.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/loop.test.ts`
- 行数：136
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/bundled/loop.test.ts`。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `loop.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/loop.ts`
- 行数：225
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `pdf.test.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/pdf.test.ts`
- 行数：313
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/bundled/pdf.test.ts`。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `pdf.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/pdf.ts`
- 行数：714
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `scheduleRemoteAgents.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/scheduleRemoteAgents.ts`
- 行数：447
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `simplify.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/simplify.ts`
- 行数：70
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `updateConfig.test.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/updateConfig.test.ts`
- 行数：39
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/bundled/updateConfig.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/skills/bundled` 的职责见上文分类。

#### `updateConfig.ts`

- 所在目录：`src/skills/bundled`
- 完整路径：`src/skills/bundled/updateConfig.ts`
- 行数：495
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills/bundled` 的职责见上文分类。

#### `bundledSkills.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/bundledSkills.ts`
- 行数：239
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills` 的职责见上文分类。

#### `loadSkillsDir.test.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/loadSkillsDir.test.ts`
- 行数：382
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/loadSkillsDir.test.ts`。所在目录 `src/skills` 的职责见上文分类。

#### `loadSkillsDir.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/loadSkillsDir.ts`
- 行数：1221
- 主要功能简介：技能发现与内置技能。
- 说明：体量较大（约 1221 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/skills` 的职责见上文分类。

#### `mcpSkillBuilders.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/mcpSkillBuilders.ts`
- 行数：44
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills` 的职责见上文分类。

#### `mcpSkills.test.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/mcpSkills.test.ts`
- 行数：173
- 主要功能简介：技能发现与内置技能。
- 说明：运行方式：`bun test ./src/skills/mcpSkills.test.ts`。所在目录 `src/skills` 的职责见上文分类。

#### `mcpSkills.ts`

- 所在目录：`src/skills`
- 完整路径：`src/skills/mcpSkills.ts`
- 行数：116
- 主要功能简介：技能发现与内置技能。
- 说明：所在目录 `src/skills` 的职责见上文分类。

### 2.111 目录组 `src/ssh`（2 个文件，约 145 行）

该组位于仓库相对路径 `src/ssh`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `SSHSessionManager.ts`

- 所在目录：`src/ssh`
- 完整路径：`src/ssh/SSHSessionManager.ts`
- 行数：58
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/ssh` 的职责见上文分类。

#### `createSSHSession.ts`

- 所在目录：`src/ssh`
- 完整路径：`src/ssh/createSSHSession.ts`
- 行数：87
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/ssh` 的职责见上文分类。

### 2.112 目录组 `src/state`（9 个文件，约 1424 行）

该组位于仓库相对路径 `src/state`。AppState 存储与选择器。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `AppState.tsx`

- 所在目录：`src/state`
- 完整路径：`src/state/AppState.tsx`
- 行数：202
- 主要功能简介：AppState 存储与选择器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/state` 的职责见上文分类。

#### `AppState.types.test.tsx`

- 所在目录：`src/state`
- 完整路径：`src/state/AppState.types.test.tsx`
- 行数：105
- 主要功能简介：AppState 存储与选择器。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/state/AppState.types.test.tsx`。所在目录 `src/state` 的职责见上文分类。

#### `AppStateStore.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/AppStateStore.ts`
- 行数：575
- 主要功能简介：AppState 存储与选择器。
- 说明：所在目录 `src/state` 的职责见上文分类。

#### `onChangeAppState.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/onChangeAppState.ts`
- 行数：179
- 主要功能简介：AppState 存储与选择器。
- 说明：所在目录 `src/state` 的职责见上文分类。

#### `pluginCommandsStore.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/pluginCommandsStore.ts`
- 行数：13
- 主要功能简介：AppState 存储与选择器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/state` 的职责见上文分类。

#### `selectors.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/selectors.ts`
- 行数：76
- 主要功能简介：AppState 存储与选择器。
- 说明：所在目录 `src/state` 的职责见上文分类。

#### `store.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/store.ts`
- 行数：34
- 主要功能简介：AppState 存储与选择器。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/state` 的职责见上文分类。

#### `teammateViewHelpers.interruptionTrace.test.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/teammateViewHelpers.interruptionTrace.test.ts`
- 行数：82
- 主要功能简介：AppState 存储与选择器。
- 说明：运行方式：`bun test ./src/state/teammateViewHelpers.interruptionTrace.test.ts`。所在目录 `src/state` 的职责见上文分类。

#### `teammateViewHelpers.ts`

- 所在目录：`src/state`
- 完整路径：`src/state/teammateViewHelpers.ts`
- 行数：158
- 主要功能简介：AppState 存储与选择器。
- 说明：所在目录 `src/state` 的职责见上文分类。

### 2.113 目录组 `src/tasks`（16 个文件，约 4234 行）

该组位于仓库相对路径 `src/tasks`。后台/本地/远程任务对象。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `DreamTask.ts`

- 所在目录：`src/tasks/DreamTask`
- 完整路径：`src/tasks/DreamTask/DreamTask.ts`
- 行数：157
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/DreamTask` 的职责见上文分类。

#### `InProcessTeammateTask.tsx`

- 所在目录：`src/tasks/InProcessTeammateTask`
- 完整路径：`src/tasks/InProcessTeammateTask/InProcessTeammateTask.tsx`
- 行数：125
- 主要功能简介：后台/本地/远程任务对象。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tasks/InProcessTeammateTask` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/tasks/InProcessTeammateTask`
- 完整路径：`src/tasks/InProcessTeammateTask/types.ts`
- 行数：121
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/InProcessTeammateTask` 的职责见上文分类。

#### `LocalAgentTask.tsx`

- 所在目录：`src/tasks/LocalAgentTask`
- 完整路径：`src/tasks/LocalAgentTask/LocalAgentTask.tsx`
- 行数：702
- 主要功能简介：后台/本地/远程任务对象。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tasks/LocalAgentTask` 的职责见上文分类。

#### `progressTracker.test.ts`

- 所在目录：`src/tasks/LocalAgentTask`
- 完整路径：`src/tasks/LocalAgentTask/progressTracker.test.ts`
- 行数：183
- 主要功能简介：后台/本地/远程任务对象。
- 说明：运行方式：`bun test ./src/tasks/LocalAgentTask/progressTracker.test.ts`。所在目录 `src/tasks/LocalAgentTask` 的职责见上文分类。

#### `LocalMainSessionTask.test.ts`

- 所在目录：`src/tasks`
- 完整路径：`src/tasks/LocalMainSessionTask.test.ts`
- 行数：408
- 主要功能简介：后台/本地/远程任务对象。
- 说明：运行方式：`bun test ./src/tasks/LocalMainSessionTask.test.ts`。所在目录 `src/tasks` 的职责见上文分类。

#### `LocalMainSessionTask.ts`

- 所在目录：`src/tasks`
- 完整路径：`src/tasks/LocalMainSessionTask.ts`
- 行数：609
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks` 的职责见上文分类。

#### `LocalShellTask.tsx`

- 所在目录：`src/tasks/LocalShellTask`
- 完整路径：`src/tasks/LocalShellTask/LocalShellTask.tsx`
- 行数：522
- 主要功能简介：后台/本地/远程任务对象。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tasks/LocalShellTask` 的职责见上文分类。

#### `guards.ts`

- 所在目录：`src/tasks/LocalShellTask`
- 完整路径：`src/tasks/LocalShellTask/guards.ts`
- 行数：41
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/LocalShellTask` 的职责见上文分类。

#### `killShellTasks.ts`

- 所在目录：`src/tasks/LocalShellTask`
- 完整路径：`src/tasks/LocalShellTask/killShellTasks.ts`
- 行数：76
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/LocalShellTask` 的职责见上文分类。

#### `LocalWorkflowTask.ts`

- 所在目录：`src/tasks/LocalWorkflowTask`
- 完整路径：`src/tasks/LocalWorkflowTask/LocalWorkflowTask.ts`
- 行数：70
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/LocalWorkflowTask` 的职责见上文分类。

#### `MonitorMcpTask.ts`

- 所在目录：`src/tasks/MonitorMcpTask`
- 完整路径：`src/tasks/MonitorMcpTask/MonitorMcpTask.ts`
- 行数：113
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks/MonitorMcpTask` 的职责见上文分类。

#### `RemoteAgentTask.tsx`

- 所在目录：`src/tasks/RemoteAgentTask`
- 完整路径：`src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`
- 行数：879
- 主要功能简介：后台/本地/远程任务对象。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 879 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tasks/RemoteAgentTask` 的职责见上文分类。

#### `pillLabel.ts`

- 所在目录：`src/tasks`
- 完整路径：`src/tasks/pillLabel.ts`
- 行数：82
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks` 的职责见上文分类。

#### `stopTask.ts`

- 所在目录：`src/tasks`
- 完整路径：`src/tasks/stopTask.ts`
- 行数：100
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/tasks`
- 完整路径：`src/tasks/types.ts`
- 行数：46
- 主要功能简介：后台/本地/远程任务对象。
- 说明：所在目录 `src/tasks` 的职责见上文分类。

### 2.114 目录组 `src/tasks.ts`（1 个文件，约 39 行）

该组位于仓库相对路径 `src/tasks.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `tasks.ts`

- 所在目录：`src`
- 完整路径：`src/tasks.ts`
- 行数：39
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src` 的职责见上文分类。

### 2.115 目录组 `src/test`（14 个文件，约 1303 行）

该组位于仓库相对路径 `src/test`。测试辅助。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `customPricingDisplay.fixture.tsx`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/customPricingDisplay.fixture.tsx`
- 行数：229
- 主要功能简介：测试辅助。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/test/fixtures` 的职责见上文分类。

#### `gracefulShutdownTrace.fixture.ts`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/gracefulShutdownTrace.fixture.ts`
- 行数：61
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test/fixtures` 的职责见上文分类。

#### `queryEngineCustomPricingBudget.fixture.ts`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/queryEngineCustomPricingBudget.fixture.ts`
- 行数：234
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test/fixtures` 的职责见上文分类。

#### `queryEngineGoalStatus.fixture.ts`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/queryEngineGoalStatus.fixture.ts`
- 行数：101
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test/fixtures` 的职责见上文分类。

#### `queryEngineManualCompactCooldown.fixture.ts`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/queryEngineManualCompactCooldown.fixture.ts`
- 行数：156
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test/fixtures` 的职责见上文分类。

#### `settingsTransactionWriter.fixture.ts`

- 所在目录：`src/test/fixtures`
- 完整路径：`src/test/fixtures/settingsTransactionWriter.fixture.ts`
- 行数：171
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test/fixtures` 的职责见上文分类。

#### `mockMacro.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/mockMacro.ts`
- 行数：32
- 主要功能简介：测试辅助。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/test` 的职责见上文分类。

#### `providerModuleIsolation.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/providerModuleIsolation.ts`
- 行数：60
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test` 的职责见上文分类。

#### `safetyLevelTestHelpers.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/safetyLevelTestHelpers.ts`
- 行数：14
- 主要功能简介：测试辅助。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/test` 的职责见上文分类。

#### `settingSourceState.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/settingSourceState.ts`
- 行数：51
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test` 的职责见上文分类。

#### `sharedMutationLock.test.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/sharedMutationLock.test.ts`
- 行数：45
- 主要功能简介：测试辅助。
- 说明：运行方式：`bun test ./src/test/sharedMutationLock.test.ts`。所在目录 `src/test` 的职责见上文分类。

#### `sharedMutationLock.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/sharedMutationLock.ts`
- 行数：60
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test` 的职责见上文分类。

#### `toolFixtures.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/toolFixtures.ts`
- 行数：61
- 主要功能简介：测试辅助。
- 说明：所在目录 `src/test` 的职责见上文分类。

#### `typedMocks.ts`

- 所在目录：`src/test`
- 完整路径：`src/test/typedMocks.ts`
- 行数：28
- 主要功能简介：测试辅助。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/test` 的职责见上文分类。

### 2.116 目录组 `src/tools`（277 个文件，约 69301 行）

该组位于仓库相对路径 `src/tools`。子 Agent 加载、运行、恢复、内置角色。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `AgentTool.copilotScheduling.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/AgentTool.copilotScheduling.test.ts`
- 行数：252
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/AgentTool.copilotScheduling.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `AgentTool.routing.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/AgentTool.routing.test.ts`
- 行数：381
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/AgentTool.routing.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `AgentTool.schema.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/AgentTool.schema.test.ts`
- 行数：378
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/AgentTool.schema.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `AgentTool.teammateModel.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/AgentTool.teammateModel.test.ts`
- 行数：506
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/AgentTool.teammateModel.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `AgentTool.tsx`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/AgentTool.tsx`
- 行数：1591
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1591 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/UI.tsx`
- 行数：759
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `agentColorManager.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/agentColorManager.ts`
- 行数：66
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `agentDisplay.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/agentDisplay.ts`
- 行数：105
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `agentMemory.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/agentMemory.ts`
- 行数：177
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `agentMemorySnapshot.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/agentMemorySnapshot.ts`
- 行数：197
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `agentToolUtils.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/agentToolUtils.ts`
- 行数：705
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `claudeCodeGuideAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/claudeCodeGuideAgent.ts`
- 行数：200
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `codeReviewerAgent.test.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/codeReviewerAgent.test.ts`
- 行数：262
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/built-in/codeReviewerAgent.test.ts`。所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `codeReviewerAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/codeReviewerAgent.ts`
- 行数：101
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `exploreAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/exploreAgent.ts`
- 行数：82
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `generalPurposeAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/generalPurposeAgent.ts`
- 行数：34
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `planAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/planAgent.ts`
- 行数：92
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `statuslineSetup.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/statuslineSetup.ts`
- 行数：150
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `verificationAgent.ts`

- 所在目录：`src/tools/AgentTool/built-in`
- 完整路径：`src/tools/AgentTool/built-in/verificationAgent.ts`
- 行数：152
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool/built-in` 的职责见上文分类。

#### `builtInAgents.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/builtInAgents.ts`
- 行数：88
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/constants.ts`
- 行数：12
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `forkSubagent.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/forkSubagent.ts`
- 行数：213
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `loadAgentsDir.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/loadAgentsDir.test.ts`
- 行数：316
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/loadAgentsDir.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `loadAgentsDir.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/loadAgentsDir.ts`
- 行数：776
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `prompt.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/prompt.test.ts`
- 行数：61
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/tools/AgentTool/prompt.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/prompt.ts`
- 行数：276
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `resumeAgent.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/resumeAgent.test.ts`
- 行数：176
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/resumeAgent.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `resumeAgent.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/resumeAgent.ts`
- 行数：290
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `runAgent.persistence.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/runAgent.persistence.test.ts`
- 行数：99
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/runAgent.persistence.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `runAgent.routing.test.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/runAgent.routing.test.ts`
- 行数：304
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：运行方式：`bun test ./src/tools/AgentTool/runAgent.routing.test.ts`。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `runAgent.ts`

- 所在目录：`src/tools/AgentTool`
- 完整路径：`src/tools/AgentTool/runAgent.ts`
- 行数：1068
- 主要功能简介：子 Agent 加载、运行、恢复、内置角色。
- 说明：体量较大（约 1068 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/AgentTool` 的职责见上文分类。

#### `AskUserQuestionTool.tsx`

- 所在目录：`src/tools/AskUserQuestionTool`
- 完整路径：`src/tools/AskUserQuestionTool/AskUserQuestionTool.tsx`
- 行数：266
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/AskUserQuestionTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/AskUserQuestionTool`
- 完整路径：`src/tools/AskUserQuestionTool/prompt.ts`
- 行数：44
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/AskUserQuestionTool` 的职责见上文分类。

#### `BashTool.errorOutput.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/BashTool.errorOutput.test.ts`
- 行数：512
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/BashTool.errorOutput.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `BashTool.sandboxAnalysis.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/BashTool.sandboxAnalysis.test.ts`
- 行数：190
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/BashTool.sandboxAnalysis.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `BashTool.tsx`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/BashTool.tsx`
- 行数：1422
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1422 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `BashToolResultMessage.tsx`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/BashToolResultMessage.tsx`
- 行数：192
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/UI.tsx`
- 行数：184
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashCommandAnalysis.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashCommandAnalysis.test.ts`
- 行数：138
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/bashCommandAnalysis.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashCommandAnalysis.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashCommandAnalysis.ts`
- 行数：166
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashCommandHelpers.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashCommandHelpers.ts`
- 行数：265
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashPermissions.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashPermissions.test.ts`
- 行数：531
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/bashPermissions.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashPermissions.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashPermissions.ts`
- 行数：2667
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：体量较大（约 2667 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashSecurity.safety.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashSecurity.safety.test.ts`
- 行数：30
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/bashSecurity.safety.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashSecurity.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashSecurity.test.ts`
- 行数：361
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/bashSecurity.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `bashSecurity.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/bashSecurity.ts`
- 行数：2941
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：体量较大（约 2941 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `commandSemantics.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/commandSemantics.test.ts`
- 行数：595
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/commandSemantics.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `commandSemantics.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/commandSemantics.ts`
- 行数：777
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `commentLabel.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/commentLabel.ts`
- 行数：13
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `destructiveCommandWarning.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/destructiveCommandWarning.ts`
- 行数：102
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `modeValidation.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/modeValidation.test.ts`
- 行数：53
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/modeValidation.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `modeValidation.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/modeValidation.ts`
- 行数：243
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `pathValidation.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/pathValidation.test.ts`
- 行数：39
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/pathValidation.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `pathValidation.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/pathValidation.ts`
- 行数：1355
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：体量较大（约 1355 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/prompt.ts`
- 行数：342
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `readOnlyValidation.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/readOnlyValidation.test.ts`
- 行数：41
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/readOnlyValidation.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `readOnlyValidation.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/readOnlyValidation.ts`
- 行数：1936
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：体量较大（约 1936 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `sedEditParser.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/sedEditParser.test.ts`
- 行数：531
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/sedEditParser.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `sedEditParser.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/sedEditParser.ts`
- 行数：786
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `sedValidation.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/sedValidation.ts`
- 行数：684
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `shouldUseSandbox.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/shouldUseSandbox.test.ts`
- 行数：112
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/shouldUseSandbox.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `shouldUseSandbox.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/shouldUseSandbox.ts`
- 行数：204
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `toolName.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/toolName.ts`
- 行数：2
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `utils.test.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/utils.test.ts`
- 行数：300
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：运行方式：`bun test ./src/tools/BashTool/utils.test.ts`。所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/tools/BashTool`
- 完整路径：`src/tools/BashTool/utils.ts`
- 行数：259
- 主要功能简介：Bash 命令执行、语义分析、沙箱与权限校验。
- 说明：所在目录 `src/tools/BashTool` 的职责见上文分类。

#### `BriefTool.ts`

- 所在目录：`src/tools/BriefTool`
- 完整路径：`src/tools/BriefTool/BriefTool.ts`
- 行数：204
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/BriefTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/BriefTool`
- 完整路径：`src/tools/BriefTool/UI.tsx`
- 行数：100
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/BriefTool` 的职责见上文分类。

#### `attachments.ts`

- 所在目录：`src/tools/BriefTool`
- 完整路径：`src/tools/BriefTool/attachments.ts`
- 行数：110
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/BriefTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/BriefTool`
- 完整路径：`src/tools/BriefTool/prompt.ts`
- 行数：22
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/BriefTool` 的职责见上文分类。

#### `upload.ts`

- 所在目录：`src/tools/BriefTool`
- 完整路径：`src/tools/BriefTool/upload.ts`
- 行数：174
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/BriefTool` 的职责见上文分类。

#### `CtxInspectTool.test.ts`

- 所在目录：`src/tools/CtxInspectTool`
- 完整路径：`src/tools/CtxInspectTool/CtxInspectTool.test.ts`
- 行数：79
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/CtxInspectTool/CtxInspectTool.test.ts`。所在目录 `src/tools/CtxInspectTool` 的职责见上文分类。

#### `CtxInspectTool.ts`

- 所在目录：`src/tools/CtxInspectTool`
- 完整路径：`src/tools/CtxInspectTool/CtxInspectTool.ts`
- 行数：89
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/CtxInspectTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/CtxInspectTool`
- 完整路径：`src/tools/CtxInspectTool/prompt.ts`
- 行数：9
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/CtxInspectTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/DiscoverSkillsTool`
- 完整路径：`src/tools/DiscoverSkillsTool/prompt.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/DiscoverSkillsTool` 的职责见上文分类。

#### `EnterPlanModeTool.ts`

- 所在目录：`src/tools/EnterPlanModeTool`
- 完整路径：`src/tools/EnterPlanModeTool/EnterPlanModeTool.ts`
- 行数：118
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/EnterPlanModeTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/EnterPlanModeTool`
- 完整路径：`src/tools/EnterPlanModeTool/UI.tsx`
- 行数：33
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/EnterPlanModeTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/EnterPlanModeTool`
- 完整路径：`src/tools/EnterPlanModeTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/EnterPlanModeTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/EnterPlanModeTool`
- 完整路径：`src/tools/EnterPlanModeTool/prompt.ts`
- 行数：103
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/EnterPlanModeTool` 的职责见上文分类。

#### `EnterWorktreeTool.ts`

- 所在目录：`src/tools/EnterWorktreeTool`
- 完整路径：`src/tools/EnterWorktreeTool/EnterWorktreeTool.ts`
- 行数：127
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/EnterWorktreeTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/EnterWorktreeTool`
- 完整路径：`src/tools/EnterWorktreeTool/UI.tsx`
- 行数：19
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/EnterWorktreeTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/EnterWorktreeTool`
- 完整路径：`src/tools/EnterWorktreeTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/EnterWorktreeTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/EnterWorktreeTool`
- 完整路径：`src/tools/EnterWorktreeTool/prompt.ts`
- 行数：30
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/EnterWorktreeTool` 的职责见上文分类。

#### `ExitPlanModeV2Tool.test.ts`

- 所在目录：`src/tools/ExitPlanModeTool`
- 完整路径：`src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.test.ts`
- 行数：197
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.test.ts`。所在目录 `src/tools/ExitPlanModeTool` 的职责见上文分类。

#### `ExitPlanModeV2Tool.ts`

- 所在目录：`src/tools/ExitPlanModeTool`
- 完整路径：`src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts`
- 行数：498
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ExitPlanModeTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/ExitPlanModeTool`
- 完整路径：`src/tools/ExitPlanModeTool/UI.tsx`
- 行数：81
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/ExitPlanModeTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/ExitPlanModeTool`
- 完整路径：`src/tools/ExitPlanModeTool/constants.ts`
- 行数：2
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ExitPlanModeTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ExitPlanModeTool`
- 完整路径：`src/tools/ExitPlanModeTool/prompt.ts`
- 行数：29
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ExitPlanModeTool` 的职责见上文分类。

#### `ExitWorktreeTool.ts`

- 所在目录：`src/tools/ExitWorktreeTool`
- 完整路径：`src/tools/ExitWorktreeTool/ExitWorktreeTool.ts`
- 行数：329
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ExitWorktreeTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/ExitWorktreeTool`
- 完整路径：`src/tools/ExitWorktreeTool/UI.tsx`
- 行数：24
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ExitWorktreeTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/ExitWorktreeTool`
- 完整路径：`src/tools/ExitWorktreeTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ExitWorktreeTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ExitWorktreeTool`
- 完整路径：`src/tools/ExitWorktreeTool/prompt.ts`
- 行数：32
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ExitWorktreeTool` 的职责见上文分类。

#### `FileEditTool.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/FileEditTool.ts`
- 行数：628
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/UI.tsx`
- 行数：300
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/constants.ts`
- 行数：11
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/prompt.ts`
- 行数：28
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/types.ts`
- 行数：85
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `utils.test.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/utils.test.ts`
- 行数：235
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：运行方式：`bun test ./src/tools/FileEditTool/utils.test.ts`。所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/tools/FileEditTool`
- 完整路径：`src/tools/FileEditTool/utils.ts`
- 行数：1057
- 主要功能简介：精确补丁编辑与冲突检测。
- 说明：体量较大（约 1057 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/FileEditTool` 的职责见上文分类。

#### `FileReadTool.oversized.test.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/FileReadTool.oversized.test.ts`
- 行数：54
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：运行方式：`bun test ./src/tools/FileReadTool/FileReadTool.oversized.test.ts`。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `FileReadTool.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/FileReadTool.ts`
- 行数：1296
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：体量较大（约 1296 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/UI.tsx`
- 行数：184
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/constants.ts`
- 行数：2
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `imageProcessor.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/imageProcessor.ts`
- 行数：119
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `limits.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/limits.ts`
- 行数：92
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/prompt.ts`
- 行数：48
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `prompt.vision.test.ts`

- 所在目录：`src/tools/FileReadTool`
- 完整路径：`src/tools/FileReadTool/prompt.vision.test.ts`
- 行数：130
- 主要功能简介：文件/图像/PDF 读取与缓存。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/tools/FileReadTool/prompt.vision.test.ts`。所在目录 `src/tools/FileReadTool` 的职责见上文分类。

#### `FileWriteTool.ts`

- 所在目录：`src/tools/FileWriteTool`
- 完整路径：`src/tools/FileWriteTool/FileWriteTool.ts`
- 行数：437
- 主要功能简介：整文件写入。
- 说明：所在目录 `src/tools/FileWriteTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/FileWriteTool`
- 完整路径：`src/tools/FileWriteTool/UI.tsx`
- 行数：423
- 主要功能简介：整文件写入。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/FileWriteTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/FileWriteTool`
- 完整路径：`src/tools/FileWriteTool/constants.ts`
- 行数：1
- 主要功能简介：整文件写入。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/FileWriteTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/FileWriteTool`
- 完整路径：`src/tools/FileWriteTool/prompt.ts`
- 行数：19
- 主要功能简介：整文件写入。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/FileWriteTool` 的职责见上文分类。

#### `GlobTool.ts`

- 所在目录：`src/tools/GlobTool`
- 完整路径：`src/tools/GlobTool/GlobTool.ts`
- 行数：198
- 主要功能简介：文件名 glob 搜索。
- 说明：所在目录 `src/tools/GlobTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/GlobTool`
- 完整路径：`src/tools/GlobTool/UI.tsx`
- 行数：62
- 主要功能简介：文件名 glob 搜索。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/GlobTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/GlobTool`
- 完整路径：`src/tools/GlobTool/prompt.ts`
- 行数：7
- 主要功能简介：文件名 glob 搜索。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/GlobTool` 的职责见上文分类。

#### `GrepTool.test.ts`

- 所在目录：`src/tools/GrepTool`
- 完整路径：`src/tools/GrepTool/GrepTool.test.ts`
- 行数：101
- 主要功能简介：ripgrep 内容搜索。
- 说明：运行方式：`bun test ./src/tools/GrepTool/GrepTool.test.ts`。所在目录 `src/tools/GrepTool` 的职责见上文分类。

#### `GrepTool.ts`

- 所在目录：`src/tools/GrepTool`
- 完整路径：`src/tools/GrepTool/GrepTool.ts`
- 行数：576
- 主要功能简介：ripgrep 内容搜索。
- 说明：所在目录 `src/tools/GrepTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/GrepTool`
- 完整路径：`src/tools/GrepTool/UI.tsx`
- 行数：200
- 主要功能简介：ripgrep 内容搜索。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/GrepTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/GrepTool`
- 完整路径：`src/tools/GrepTool/prompt.ts`
- 行数：18
- 主要功能简介：ripgrep 内容搜索。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/GrepTool` 的职责见上文分类。

#### `LSPTool.ts`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/LSPTool.ts`
- 行数：860
- 主要功能简介：语言服务器桥。
- 说明：体量较大（约 860 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/UI.tsx`
- 行数：227
- 主要功能简介：语言服务器桥。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `formatters.ts`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/formatters.ts`
- 行数：592
- 主要功能简介：语言服务器桥。
- 说明：所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/prompt.ts`
- 行数：21
- 主要功能简介：语言服务器桥。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `schemas.ts`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/schemas.ts`
- 行数：215
- 主要功能简介：语言服务器桥。
- 说明：所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `symbolContext.ts`

- 所在目录：`src/tools/LSPTool`
- 完整路径：`src/tools/LSPTool/symbolContext.ts`
- 行数：90
- 主要功能简介：语言服务器桥。
- 说明：所在目录 `src/tools/LSPTool` 的职责见上文分类。

#### `ListMcpResourcesTool.ts`

- 所在目录：`src/tools/ListMcpResourcesTool`
- 完整路径：`src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts`
- 行数：123
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ListMcpResourcesTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/ListMcpResourcesTool`
- 完整路径：`src/tools/ListMcpResourcesTool/UI.tsx`
- 行数：28
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ListMcpResourcesTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ListMcpResourcesTool`
- 完整路径：`src/tools/ListMcpResourcesTool/prompt.ts`
- 行数：20
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ListMcpResourcesTool` 的职责见上文分类。

#### `MCPTool.test.ts`

- 所在目录：`src/tools/MCPTool`
- 完整路径：`src/tools/MCPTool/MCPTool.test.ts`
- 行数：261
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/MCPTool/MCPTool.test.ts`。所在目录 `src/tools/MCPTool` 的职责见上文分类。

#### `MCPTool.ts`

- 所在目录：`src/tools/MCPTool`
- 完整路径：`src/tools/MCPTool/MCPTool.ts`
- 行数：183
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/MCPTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/MCPTool`
- 完整路径：`src/tools/MCPTool/UI.tsx`
- 行数：405
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/MCPTool` 的职责见上文分类。

#### `classifyForCollapse.ts`

- 所在目录：`src/tools/MCPTool`
- 完整路径：`src/tools/MCPTool/classifyForCollapse.ts`
- 行数：604
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/MCPTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/MCPTool`
- 完整路径：`src/tools/MCPTool/prompt.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/MCPTool` 的职责见上文分类。

#### `McpAuthTool.ts`

- 所在目录：`src/tools/McpAuthTool`
- 完整路径：`src/tools/McpAuthTool/McpAuthTool.ts`
- 行数：215
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/McpAuthTool` 的职责见上文分类。

#### `MonitorTool.ts`

- 所在目录：`src/tools/MonitorTool`
- 完整路径：`src/tools/MonitorTool/MonitorTool.ts`
- 行数：195
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/MonitorTool` 的职责见上文分类。

#### `NotebookEditTool.ts`

- 所在目录：`src/tools/NotebookEditTool`
- 完整路径：`src/tools/NotebookEditTool/NotebookEditTool.ts`
- 行数：490
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/NotebookEditTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/NotebookEditTool`
- 完整路径：`src/tools/NotebookEditTool/UI.tsx`
- 行数：92
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/NotebookEditTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/NotebookEditTool`
- 完整路径：`src/tools/NotebookEditTool/constants.ts`
- 行数：2
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/NotebookEditTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/NotebookEditTool`
- 完整路径：`src/tools/NotebookEditTool/prompt.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/NotebookEditTool` 的职责见上文分类。

#### `OverflowTestTool.ts`

- 所在目录：`src/tools/OverflowTestTool`
- 完整路径：`src/tools/OverflowTestTool/OverflowTestTool.ts`
- 行数：41
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/OverflowTestTool` 的职责见上文分类。

#### `PowerShellTool.errorOutput.test.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/PowerShellTool.errorOutput.test.ts`
- 行数：281
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：运行方式：`bun test ./src/tools/PowerShellTool/PowerShellTool.errorOutput.test.ts`。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `PowerShellTool.tsx`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/PowerShellTool.tsx`
- 行数：1370
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1370 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/UI.tsx`
- 行数：131
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `clmTypes.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/clmTypes.ts`
- 行数：211
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `commandSemantics.test.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/commandSemantics.test.ts`
- 行数：493
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：运行方式：`bun test ./src/tools/PowerShellTool/commandSemantics.test.ts`。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `commandSemantics.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/commandSemantics.ts`
- 行数：847
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：体量较大（约 847 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `commonParameters.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/commonParameters.ts`
- 行数：30
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `destructiveCommandWarning.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/destructiveCommandWarning.ts`
- 行数：109
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `gitSafety.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/gitSafety.ts`
- 行数：176
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `modeValidation.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/modeValidation.ts`
- 行数：405
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `pathValidation.protoName.test.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/pathValidation.protoName.test.ts`
- 行数：96
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：运行方式：`bun test ./src/tools/PowerShellTool/pathValidation.protoName.test.ts`。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `pathValidation.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/pathValidation.ts`
- 行数：2059
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：体量较大（约 2059 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `powershellPermissions.test.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/powershellPermissions.test.ts`
- 行数：330
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：运行方式：`bun test ./src/tools/PowerShellTool/powershellPermissions.test.ts`。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `powershellPermissions.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/powershellPermissions.ts`
- 行数：1684
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：体量较大（约 1684 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `powershellSecurity.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/powershellSecurity.ts`
- 行数：1090
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：体量较大（约 1090 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/prompt.ts`
- 行数：150
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `readOnlyValidation.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/readOnlyValidation.ts`
- 行数：1831
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：体量较大（约 1831 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `toolName.ts`

- 所在目录：`src/tools/PowerShellTool`
- 完整路径：`src/tools/PowerShellTool/toolName.ts`
- 行数：2
- 主要功能简介：Windows PowerShell 执行与安全。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/PowerShellTool` 的职责见上文分类。

#### `REPLTool.ts`

- 所在目录：`src/tools/REPLTool`
- 完整路径：`src/tools/REPLTool/REPLTool.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/REPLTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/REPLTool`
- 完整路径：`src/tools/REPLTool/constants.ts`
- 行数：38
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/REPLTool` 的职责见上文分类。

#### `primitiveTools.ts`

- 所在目录：`src/tools/REPLTool`
- 完整路径：`src/tools/REPLTool/primitiveTools.ts`
- 行数：39
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/REPLTool` 的职责见上文分类。

#### `ReadMcpResourceTool.ts`

- 所在目录：`src/tools/ReadMcpResourceTool`
- 完整路径：`src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts`
- 行数：167
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ReadMcpResourceTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/ReadMcpResourceTool`
- 完整路径：`src/tools/ReadMcpResourceTool/UI.tsx`
- 行数：36
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ReadMcpResourceTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ReadMcpResourceTool`
- 完整路径：`src/tools/ReadMcpResourceTool/prompt.ts`
- 行数：16
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ReadMcpResourceTool` 的职责见上文分类。

#### `RemoteTriggerTool.ts`

- 所在目录：`src/tools/RemoteTriggerTool`
- 完整路径：`src/tools/RemoteTriggerTool/RemoteTriggerTool.ts`
- 行数：161
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/RemoteTriggerTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/RemoteTriggerTool`
- 完整路径：`src/tools/RemoteTriggerTool/UI.tsx`
- 行数：16
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/RemoteTriggerTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/RemoteTriggerTool`
- 完整路径：`src/tools/RemoteTriggerTool/prompt.ts`
- 行数：15
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/RemoteTriggerTool` 的职责见上文分类。

#### `RepoMapTool.test.ts`

- 所在目录：`src/tools/RepoMapTool`
- 完整路径：`src/tools/RepoMapTool/RepoMapTool.test.ts`
- 行数：299
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/RepoMapTool/RepoMapTool.test.ts`。所在目录 `src/tools/RepoMapTool` 的职责见上文分类。

#### `RepoMapTool.ts`

- 所在目录：`src/tools/RepoMapTool`
- 完整路径：`src/tools/RepoMapTool/RepoMapTool.ts`
- 行数：160
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/RepoMapTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/RepoMapTool`
- 完整路径：`src/tools/RepoMapTool/UI.tsx`
- 行数：88
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/RepoMapTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/RepoMapTool`
- 完整路径：`src/tools/RepoMapTool/prompt.ts`
- 行数：31
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/RepoMapTool` 的职责见上文分类。

#### `ReviewArtifactTool.ts`

- 所在目录：`src/tools/ReviewArtifactTool`
- 完整路径：`src/tools/ReviewArtifactTool/ReviewArtifactTool.ts`
- 行数：43
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ReviewArtifactTool` 的职责见上文分类。

#### `CronCreateTool.ts`

- 所在目录：`src/tools/ScheduleCronTool`
- 完整路径：`src/tools/ScheduleCronTool/CronCreateTool.ts`
- 行数：174
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ScheduleCronTool` 的职责见上文分类。

#### `CronDeleteTool.ts`

- 所在目录：`src/tools/ScheduleCronTool`
- 完整路径：`src/tools/ScheduleCronTool/CronDeleteTool.ts`
- 行数：95
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ScheduleCronTool` 的职责见上文分类。

#### `CronListTool.ts`

- 所在目录：`src/tools/ScheduleCronTool`
- 完整路径：`src/tools/ScheduleCronTool/CronListTool.ts`
- 行数：97
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ScheduleCronTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/ScheduleCronTool`
- 完整路径：`src/tools/ScheduleCronTool/UI.tsx`
- 行数：59
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/ScheduleCronTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ScheduleCronTool`
- 完整路径：`src/tools/ScheduleCronTool/prompt.ts`
- 行数：131
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/ScheduleCronTool` 的职责见上文分类。

#### `SendMessageTool.ts`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/SendMessageTool.ts`
- 行数：940
- 主要功能简介：模型可调用工具实现。
- 说明：体量较大（约 940 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/UI.tsx`
- 行数：30
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/prompt.ts`
- 行数：49
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `shutdownInterruptionTrace.test.ts`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/shutdownInterruptionTrace.test.ts`
- 行数：66
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/SendMessageTool/shutdownInterruptionTrace.test.ts`。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `shutdownInterruptionTrace.ts`

- 所在目录：`src/tools/SendMessageTool`
- 完整路径：`src/tools/SendMessageTool/shutdownInterruptionTrace.ts`
- 行数：23
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SendMessageTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/SendUserFileTool`
- 完整路径：`src/tools/SendUserFileTool/prompt.ts`
- 行数：10
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SendUserFileTool` 的职责见上文分类。

#### `SkillTool.test.ts`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/SkillTool.test.ts`
- 行数：98
- 主要功能简介：技能加载与执行。
- 说明：运行方式：`bun test ./src/tools/SkillTool/SkillTool.test.ts`。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `SkillTool.ts`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/SkillTool.ts`
- 行数：1118
- 主要功能简介：技能加载与执行。
- 说明：体量较大（约 1118 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/UI.tsx`
- 行数：130
- 主要功能简介：技能加载与执行。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/constants.ts`
- 行数：1
- 主要功能简介：技能加载与执行。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `prompt.test.ts`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/prompt.test.ts`
- 行数：76
- 主要功能简介：技能加载与执行。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/tools/SkillTool/prompt.test.ts`。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/SkillTool`
- 完整路径：`src/tools/SkillTool/prompt.ts`
- 行数：255
- 主要功能简介：技能加载与执行。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/SkillTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/SleepTool`
- 完整路径：`src/tools/SleepTool/prompt.ts`
- 行数：17
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SleepTool` 的职责见上文分类。

#### `SnipTool.test.ts`

- 所在目录：`src/tools/SnipTool`
- 完整路径：`src/tools/SnipTool/SnipTool.test.ts`
- 行数：46
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/SnipTool/SnipTool.test.ts`。所在目录 `src/tools/SnipTool` 的职责见上文分类。

#### `SnipTool.ts`

- 所在目录：`src/tools/SnipTool`
- 完整路径：`src/tools/SnipTool/SnipTool.ts`
- 行数：69
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/SnipTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/SnipTool`
- 完整路径：`src/tools/SnipTool/prompt.ts`
- 行数：15
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SnipTool` 的职责见上文分类。

#### `SuggestBackgroundPRTool.ts`

- 所在目录：`src/tools/SuggestBackgroundPRTool`
- 完整路径：`src/tools/SuggestBackgroundPRTool/SuggestBackgroundPRTool.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/SuggestBackgroundPRTool` 的职责见上文分类。

#### `SyntheticOutputTool.test.ts`

- 所在目录：`src/tools/SyntheticOutputTool`
- 完整路径：`src/tools/SyntheticOutputTool/SyntheticOutputTool.test.ts`
- 行数：102
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/SyntheticOutputTool/SyntheticOutputTool.test.ts`。所在目录 `src/tools/SyntheticOutputTool` 的职责见上文分类。

#### `SyntheticOutputTool.ts`

- 所在目录：`src/tools/SyntheticOutputTool`
- 完整路径：`src/tools/SyntheticOutputTool/SyntheticOutputTool.ts`
- 行数：201
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/SyntheticOutputTool` 的职责见上文分类。

#### `TaskCreateTool.ts`

- 所在目录：`src/tools/TaskCreateTool`
- 完整路径：`src/tools/TaskCreateTool/TaskCreateTool.ts`
- 行数：138
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TaskCreateTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TaskCreateTool`
- 完整路径：`src/tools/TaskCreateTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskCreateTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TaskCreateTool`
- 完整路径：`src/tools/TaskCreateTool/prompt.ts`
- 行数：56
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/TaskCreateTool` 的职责见上文分类。

#### `TaskGetTool.ts`

- 所在目录：`src/tools/TaskGetTool`
- 完整路径：`src/tools/TaskGetTool/TaskGetTool.ts`
- 行数：128
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TaskGetTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TaskGetTool`
- 完整路径：`src/tools/TaskGetTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskGetTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TaskGetTool`
- 完整路径：`src/tools/TaskGetTool/prompt.ts`
- 行数：24
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskGetTool` 的职责见上文分类。

#### `TaskListTool.ts`

- 所在目录：`src/tools/TaskListTool`
- 完整路径：`src/tools/TaskListTool/TaskListTool.ts`
- 行数：116
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TaskListTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TaskListTool`
- 完整路径：`src/tools/TaskListTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskListTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TaskListTool`
- 完整路径：`src/tools/TaskListTool/prompt.ts`
- 行数：49
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/TaskListTool` 的职责见上文分类。

#### `TaskOutputTool.activity.test.ts`

- 所在目录：`src/tools/TaskOutputTool`
- 完整路径：`src/tools/TaskOutputTool/TaskOutputTool.activity.test.ts`
- 行数：75
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/TaskOutputTool/TaskOutputTool.activity.test.ts`。所在目录 `src/tools/TaskOutputTool` 的职责见上文分类。

#### `TaskOutputTool.tsx`

- 所在目录：`src/tools/TaskOutputTool`
- 完整路径：`src/tools/TaskOutputTool/TaskOutputTool.tsx`
- 行数：601
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/TaskOutputTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TaskOutputTool`
- 完整路径：`src/tools/TaskOutputTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskOutputTool` 的职责见上文分类。

#### `TaskStopTool.ts`

- 所在目录：`src/tools/TaskStopTool`
- 完整路径：`src/tools/TaskStopTool/TaskStopTool.ts`
- 行数：131
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TaskStopTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/TaskStopTool`
- 完整路径：`src/tools/TaskStopTool/UI.tsx`
- 行数：37
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskStopTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TaskStopTool`
- 完整路径：`src/tools/TaskStopTool/prompt.ts`
- 行数：8
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskStopTool` 的职责见上文分类。

#### `TaskUpdateTool.ts`

- 所在目录：`src/tools/TaskUpdateTool`
- 完整路径：`src/tools/TaskUpdateTool/TaskUpdateTool.ts`
- 行数：406
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TaskUpdateTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TaskUpdateTool`
- 完整路径：`src/tools/TaskUpdateTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TaskUpdateTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TaskUpdateTool`
- 完整路径：`src/tools/TaskUpdateTool/prompt.ts`
- 行数：77
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/TaskUpdateTool` 的职责见上文分类。

#### `TeamCreateTool.ts`

- 所在目录：`src/tools/TeamCreateTool`
- 完整路径：`src/tools/TeamCreateTool/TeamCreateTool.ts`
- 行数：240
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TeamCreateTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/TeamCreateTool`
- 完整路径：`src/tools/TeamCreateTool/UI.tsx`
- 行数：5
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TeamCreateTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TeamCreateTool`
- 完整路径：`src/tools/TeamCreateTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TeamCreateTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TeamCreateTool`
- 完整路径：`src/tools/TeamCreateTool/prompt.ts`
- 行数：111
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/TeamCreateTool` 的职责见上文分类。

#### `TeamDeleteTool.ts`

- 所在目录：`src/tools/TeamDeleteTool`
- 完整路径：`src/tools/TeamDeleteTool/TeamDeleteTool.ts`
- 行数：139
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/TeamDeleteTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/TeamDeleteTool`
- 完整路径：`src/tools/TeamDeleteTool/UI.tsx`
- 行数：19
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TeamDeleteTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TeamDeleteTool`
- 完整路径：`src/tools/TeamDeleteTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TeamDeleteTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TeamDeleteTool`
- 完整路径：`src/tools/TeamDeleteTool/prompt.ts`
- 行数：16
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TeamDeleteTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TerminalCaptureTool`
- 完整路径：`src/tools/TerminalCaptureTool/prompt.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TerminalCaptureTool` 的职责见上文分类。

#### `TodoWriteTool.ts`

- 所在目录：`src/tools/TodoWriteTool`
- 完整路径：`src/tools/TodoWriteTool/TodoWriteTool.ts`
- 行数：115
- 主要功能简介：会话待办。
- 说明：所在目录 `src/tools/TodoWriteTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/TodoWriteTool`
- 完整路径：`src/tools/TodoWriteTool/constants.ts`
- 行数：1
- 主要功能简介：会话待办。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TodoWriteTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/TodoWriteTool`
- 完整路径：`src/tools/TodoWriteTool/prompt.ts`
- 行数：184
- 主要功能简介：会话待办。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/TodoWriteTool` 的职责见上文分类。

#### `ToolSearchTool.ts`

- 所在目录：`src/tools/ToolSearchTool`
- 完整路径：`src/tools/ToolSearchTool/ToolSearchTool.ts`
- 行数：471
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/ToolSearchTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/ToolSearchTool`
- 完整路径：`src/tools/ToolSearchTool/constants.ts`
- 行数：1
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/ToolSearchTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/ToolSearchTool`
- 完整路径：`src/tools/ToolSearchTool/prompt.ts`
- 行数：121
- 主要功能简介：模型可调用工具实现。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/ToolSearchTool` 的职责见上文分类。

#### `TungstenLiveMonitor.ts`

- 所在目录：`src/tools/TungstenTool`
- 完整路径：`src/tools/TungstenTool/TungstenLiveMonitor.ts`
- 行数：2
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TungstenTool` 的职责见上文分类。

#### `TungstenTool.ts`

- 所在目录：`src/tools/TungstenTool`
- 完整路径：`src/tools/TungstenTool/TungstenTool.ts`
- 行数：2
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/TungstenTool` 的职责见上文分类。

#### `VerifyPlanExecutionTool.ts`

- 所在目录：`src/tools/VerifyPlanExecutionTool`
- 完整路径：`src/tools/VerifyPlanExecutionTool/VerifyPlanExecutionTool.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/VerifyPlanExecutionTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/VerifyPlanExecutionTool`
- 完整路径：`src/tools/VerifyPlanExecutionTool/constants.ts`
- 行数：3
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/VerifyPlanExecutionTool` 的职责见上文分类。

#### `WebBrowserPanel.tsx`

- 所在目录：`src/tools/WebBrowserTool`
- 完整路径：`src/tools/WebBrowserTool/WebBrowserPanel.tsx`
- 行数：7
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/WebBrowserTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/UI.tsx`
- 行数：71
- 主要功能简介：URL 抓取与可读化。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `WebFetchTool.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/WebFetchTool.ts`
- 行数：359
- 主要功能简介：URL 抓取与可读化。
- 说明：所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `applyPromptFallback.test.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/applyPromptFallback.test.ts`
- 行数：96
- 主要功能简介：URL 抓取与可读化。
- 说明：运行方式：`bun test ./src/tools/WebFetchTool/applyPromptFallback.test.ts`。所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `domainCheck.test.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/domainCheck.test.ts`
- 行数：87
- 主要功能简介：URL 抓取与可读化。
- 说明：运行方式：`bun test ./src/tools/WebFetchTool/domainCheck.test.ts`。所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `preapproved.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/preapproved.ts`
- 行数：166
- 主要功能简介：URL 抓取与可读化。
- 说明：所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/prompt.ts`
- 行数：46
- 主要功能简介：URL 抓取与可读化。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/tools/WebFetchTool`
- 完整路径：`src/tools/WebFetchTool/utils.ts`
- 行数：671
- 主要功能简介：URL 抓取与可读化。
- 说明：所在目录 `src/tools/WebFetchTool` 的职责见上文分类。

#### `UI.tsx`

- 所在目录：`src/tools/WebSearchTool`
- 完整路径：`src/tools/WebSearchTool/UI.tsx`
- 行数：100
- 主要功能简介：多供应商网页搜索适配。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/WebSearchTool` 的职责见上文分类。

#### `WebSearchTool.test.ts`

- 所在目录：`src/tools/WebSearchTool`
- 完整路径：`src/tools/WebSearchTool/WebSearchTool.test.ts`
- 行数：106
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/WebSearchTool.test.ts`。所在目录 `src/tools/WebSearchTool` 的职责见上文分类。

#### `WebSearchTool.ts`

- 所在目录：`src/tools/WebSearchTool`
- 完整路径：`src/tools/WebSearchTool/WebSearchTool.ts`
- 行数：945
- 主要功能简介：多供应商网页搜索适配。
- 说明：体量较大（约 945 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/WebSearchTool` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/tools/WebSearchTool`
- 完整路径：`src/tools/WebSearchTool/prompt.ts`
- 行数：34
- 主要功能简介：多供应商网页搜索适配。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/WebSearchTool` 的职责见上文分类。

#### `bing.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/bing.ts`
- 行数：47
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `brave.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/brave.test.ts`
- 行数：156
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/brave.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `brave.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/brave.ts`
- 行数：53
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `custom.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/custom.test.ts`
- 行数：457
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/custom.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `custom.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/custom.ts`
- 行数：676
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `duckduckgo.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/duckduckgo.test.ts`
- 行数：103
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/duckduckgo.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `duckduckgo.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/duckduckgo.ts`
- 行数：133
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `exa.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/exa.test.ts`
- 行数：154
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/exa.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `exa.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/exa.ts`
- 行数：84
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `firecrawl.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/firecrawl.test.ts`
- 行数：59
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/firecrawl.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `firecrawl.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/firecrawl.ts`
- 行数：49
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `index.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/index.test.ts`
- 行数：309
- 主要功能简介：多供应商网页搜索适配。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。运行方式：`bun test ./src/tools/WebSearchTool/providers/index.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/index.ts`
- 行数：200
- 主要功能简介：多供应商网页搜索适配。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `jina.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/jina.ts`
- 行数：50
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `linkup.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/linkup.ts`
- 行数：52
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `mojeek.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/mojeek.ts`
- 行数：57
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `tavily.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/tavily.ts`
- 行数：52
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `timeout.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/timeout.test.ts`
- 行数：177
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/timeout.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `timeout.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/timeout.ts`
- 行数：157
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `types.test.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/types.test.ts`
- 行数：262
- 主要功能简介：多供应商网页搜索适配。
- 说明：运行方式：`bun test ./src/tools/WebSearchTool/providers/types.test.ts`。所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/types.ts`
- 行数：128
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `you.ts`

- 所在目录：`src/tools/WebSearchTool/providers`
- 完整路径：`src/tools/WebSearchTool/providers/you.ts`
- 行数：52
- 主要功能简介：多供应商网页搜索适配。
- 说明：所在目录 `src/tools/WebSearchTool/providers` 的职责见上文分类。

#### `WorkflowPermissionRequest.tsx`

- 所在目录：`src/tools/WorkflowTool`
- 完整路径：`src/tools/WorkflowTool/WorkflowPermissionRequest.tsx`
- 行数：11
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/WorkflowTool` 的职责见上文分类。

#### `WorkflowTool.ts`

- 所在目录：`src/tools/WorkflowTool`
- 完整路径：`src/tools/WorkflowTool/WorkflowTool.ts`
- 行数：40
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/WorkflowTool` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/tools/WorkflowTool`
- 完整路径：`src/tools/WorkflowTool/constants.ts`
- 行数：2
- 主要功能简介：模型可调用工具实现。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/tools/WorkflowTool` 的职责见上文分类。

#### `createWorkflowCommand.ts`

- 所在目录：`src/tools/WorkflowTool`
- 完整路径：`src/tools/WorkflowTool/createWorkflowCommand.ts`
- 行数：7
- 主要功能简介：模型可调用工具实现。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/tools/WorkflowTool` 的职责见上文分类。

#### `client.test.ts`

- 所在目录：`src/tools/firecrawl`
- 完整路径：`src/tools/firecrawl/client.test.ts`
- 行数：233
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/firecrawl/client.test.ts`。所在目录 `src/tools/firecrawl` 的职责见上文分类。

#### `client.ts`

- 所在目录：`src/tools/firecrawl`
- 完整路径：`src/tools/firecrawl/client.ts`
- 行数：183
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/firecrawl` 的职责见上文分类。

#### `gitOperationTracking.ts`

- 所在目录：`src/tools/shared`
- 完整路径：`src/tools/shared/gitOperationTracking.ts`
- 行数：272
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools/shared` 的职责见上文分类。

#### `spawnMultiAgent.ts`

- 所在目录：`src/tools/shared`
- 完整路径：`src/tools/shared/spawnMultiAgent.ts`
- 行数：1164
- 主要功能简介：模型可调用工具实现。
- 说明：体量较大（约 1164 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `src/tools/shared` 的职责见上文分类。

#### `shellToolResultMappers.test.ts`

- 所在目录：`src/tools`
- 完整路径：`src/tools/shellToolResultMappers.test.ts`
- 行数：71
- 主要功能简介：模型可调用工具实现。
- 说明：运行方式：`bun test ./src/tools/shellToolResultMappers.test.ts`。所在目录 `src/tools` 的职责见上文分类。

#### `TestingPermissionTool.tsx`

- 所在目录：`src/tools/testing`
- 完整路径：`src/tools/testing/TestingPermissionTool.tsx`
- 行数：73
- 主要功能简介：模型可调用工具实现。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/tools/testing` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/tools`
- 完整路径：`src/tools/utils.ts`
- 行数：40
- 主要功能简介：模型可调用工具实现。
- 说明：所在目录 `src/tools` 的职责见上文分类。

### 2.117 目录组 `src/tools.lsp.test.ts`（1 个文件，约 81 行）

该组位于仓库相对路径 `src/tools.lsp.test.ts`。自动化测试，守护对应实现的行为不变量。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `tools.lsp.test.ts`

- 所在目录：`src`
- 完整路径：`src/tools.lsp.test.ts`
- 行数：81
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/tools.lsp.test.ts`。所在目录 `src` 的职责见上文分类。

### 2.118 目录组 `src/tools.ts`（1 个文件，约 388 行）

该组位于仓库相对路径 `src/tools.ts`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `tools.ts`

- 所在目录：`src`
- 完整路径：`src/tools.ts`
- 行数：388
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src` 的职责见上文分类。

### 2.119 目录组 `src/types`（21 个文件，约 3318 行）

该组位于仓库相对路径 `src/types`。共享 TypeScript 类型。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `command.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/command.ts`
- 行数：257
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `connectorText.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/connectorText.ts`
- 行数：24
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

#### `fileSuggestion.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/fileSuggestion.ts`
- 行数：18
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

#### `claude_code_internal_event.ts`

- 所在目录：`src/types/generated/events_mono/claude_code/v1`
- 完整路径：`src/types/generated/events_mono/claude_code/v1/claude_code_internal_event.ts`
- 行数：108
- 主要功能简介：共享 TypeScript 类型。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `src/types/generated/events_mono/claude_code/v1` 的职责见上文分类。

#### `auth.ts`

- 所在目录：`src/types/generated/events_mono/common/v1`
- 完整路径：`src/types/generated/events_mono/common/v1/auth.ts`
- 行数：25
- 主要功能简介：共享 TypeScript 类型。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。短文件，多为常量、再导出或薄包装。所在目录 `src/types/generated/events_mono/common/v1` 的职责见上文分类。

#### `growthbook_experiment_event.ts`

- 所在目录：`src/types/generated/events_mono/growthbook/v1`
- 完整路径：`src/types/generated/events_mono/growthbook/v1/growthbook_experiment_event.ts`
- 行数：40
- 主要功能简介：共享 TypeScript 类型。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `src/types/generated/events_mono/growthbook/v1` 的职责见上文分类。

#### `timestamp.ts`

- 所在目录：`src/types/generated/google/protobuf`
- 完整路径：`src/types/generated/google/protobuf/timestamp.ts`
- 行数：19
- 主要功能简介：共享 TypeScript 类型。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。短文件，多为常量、再导出或薄包装。所在目录 `src/types/generated/google/protobuf` 的职责见上文分类。

#### `hooks.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/hooks.ts`
- 行数：290
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `ids.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/ids.ts`
- 行数：44
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `logs.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/logs.ts`
- 行数：480
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `message.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/message.ts`
- 行数：497
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `messageQueueTypes.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/messageQueueTypes.ts`
- 行数：19
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

#### `notebook.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/notebook.ts`
- 行数：77
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `permissions.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/permissions.ts`
- 行数：442
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `plugin.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/plugin.ts`
- 行数：365
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `statusLine.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/statusLine.ts`
- 行数：94
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `textInputTypes.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/textInputTypes.ts`
- 行数：409
- 主要功能简介：共享 TypeScript 类型。
- 说明：所在目录 `src/types` 的职责见上文分类。

#### `tools.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/tools.ts`
- 行数：17
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

#### `utils.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/utils.ts`
- 行数：23
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

#### `utils.types.test.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/utils.types.test.ts`
- 行数：63
- 主要功能简介：共享 TypeScript 类型。
- 说明：运行方式：`bun test ./src/types/utils.types.test.ts`。所在目录 `src/types` 的职责见上文分类。

#### `vitest-compat.d.ts`

- 所在目录：`src/types`
- 完整路径：`src/types/vitest-compat.d.ts`
- 行数：7
- 主要功能简介：共享 TypeScript 类型。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/types` 的职责见上文分类。

### 2.120 目录组 `src/upstreamproxy`（3 个文件，约 819 行）

该组位于仓库相对路径 `src/upstreamproxy`。项目源码文件，服务于 CLI、SDK 或配套工具。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `relay.ts`

- 所在目录：`src/upstreamproxy`
- 完整路径：`src/upstreamproxy/relay.ts`
- 行数：464
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/upstreamproxy` 的职责见上文分类。

#### `upstreamproxy.test.ts`

- 所在目录：`src/upstreamproxy`
- 完整路径：`src/upstreamproxy/upstreamproxy.test.ts`
- 行数：42
- 主要功能简介：自动化测试，守护对应实现的行为不变量。
- 说明：运行方式：`bun test ./src/upstreamproxy/upstreamproxy.test.ts`。所在目录 `src/upstreamproxy` 的职责见上文分类。

#### `upstreamproxy.ts`

- 所在目录：`src/upstreamproxy`
- 完整路径：`src/upstreamproxy/upstreamproxy.ts`
- 行数：313
- 主要功能简介：项目源码文件，服务于 CLI、SDK 或配套工具。
- 说明：所在目录 `src/upstreamproxy` 的职责见上文分类。

### 2.121 目录组 `src/utils`（956 个文件，约 297498 行）

该组位于仓库相对路径 `src/utils`。共享工具函数。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `CircularBuffer.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/CircularBuffer.ts`
- 行数：84
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `Cursor.nfc.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/Cursor.nfc.test.ts`
- 行数：31
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/Cursor.nfc.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `Cursor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/Cursor.ts`
- 行数：1638
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1638 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `QueryGuard.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/QueryGuard.test.ts`
- 行数：821
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/QueryGuard.test.ts`。体量较大（约 821 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `QueryGuard.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/QueryGuard.ts`
- 行数：719
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `Shell.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/Shell.ts`
- 行数：513
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `ShellCommand.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ShellCommand.test.ts`
- 行数：125
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/ShellCommand.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `ShellCommand.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ShellCommand.ts`
- 行数：548
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `abortController.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/abortController.ts`
- 行数：141
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `abortReasons.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/abortReasons.test.ts`
- 行数：197
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/abortReasons.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `abortReasons.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/abortReasons.ts`
- 行数：183
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `activityManager.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/activityManager.ts`
- 行数：129
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `advisor.modelGate.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/advisor.modelGate.test.ts`
- 行数：37
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/advisor.modelGate.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `advisor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/advisor.ts`
- 行数：149
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `agentContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/agentContext.ts`
- 行数：178
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `agentId.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/agentId.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `agentSwarmsEnabled.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/agentSwarmsEnabled.ts`
- 行数：44
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `agenticSessionSearch.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/agenticSessionSearch.ts`
- 行数：307
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `analyzeContext.mcp.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/analyzeContext.mcp.test.ts`
- 行数：185
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/analyzeContext.mcp.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `analyzeContext.messageBreakdown.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/analyzeContext.messageBreakdown.test.ts`
- 行数：142
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/analyzeContext.messageBreakdown.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `analyzeContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/analyzeContext.ts`
- 行数：1437
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1437 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `ansiToPng.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ansiToPng.ts`
- 行数：334
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `ansiToSvg.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ansiToSvg.ts`
- 行数：272
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `anthropicAttribution.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/anthropicAttribution.test.ts`
- 行数：233
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/anthropicAttribution.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `anthropicAttribution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/anthropicAttribution.ts`
- 行数：139
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `anthropicBaseUrl.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/anthropicBaseUrl.test.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/anthropicBaseUrl.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `anthropicBaseUrl.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/anthropicBaseUrl.ts`
- 行数：129
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `api.swarmFields.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/api.swarmFields.test.ts`
- 行数：46
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/api.swarmFields.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `api.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/api.test.ts`
- 行数：105
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/api.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `api.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/api.ts`
- 行数：790
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `apiPreconnect.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/apiPreconnect.test.ts`
- 行数：134
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/apiPreconnect.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `apiPreconnect.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/apiPreconnect.ts`
- 行数：89
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `appleTerminalBackup.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/appleTerminalBackup.ts`
- 行数：124
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `argumentSubstitution.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/argumentSubstitution.test.ts`
- 行数：82
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/argumentSubstitution.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `argumentSubstitution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/argumentSubstitution.ts`
- 行数：175
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `array.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/array.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `asciicast.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/asciicast.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `atomicReplace.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/atomicReplace.test.ts`
- 行数：273
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/atomicReplace.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `atomicReplace.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/atomicReplace.ts`
- 行数：294
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.contextEfficiency.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.contextEfficiency.test.ts`
- 行数：121
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.contextEfficiency.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.editedImage.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.editedImage.test.ts`
- 行数：82
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.editedImage.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.extractors.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.extractors.test.ts`
- 行数：102
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.extractors.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.lspDiagnostics.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.lspDiagnostics.test.ts`
- 行数：269
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.lspDiagnostics.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.nestedDirs.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.nestedDirs.test.ts`
- 行数：116
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.nestedDirs.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.performance.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.performance.test.ts`
- 行数：221
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.performance.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.ts`
- 行数：4454
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 4454 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.ultracode.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.ultracode.test.ts`
- 行数：113
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.ultracode.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attachments.ultrathink.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attachments.ultrathink.test.ts`
- 行数：62
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attachments.ultrathink.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `attribution.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attribution.test.ts`
- 行数：434
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/attribution.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attribution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attribution.ts`
- 行数：480
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `attributionHooks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attributionHooks.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `attributionTrailer.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/attributionTrailer.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `auth.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/auth.test.ts`
- 行数：158
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/auth.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `auth.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/auth.ts`
- 行数：2058
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2058 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `authFileDescriptor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/authFileDescriptor.ts`
- 行数：196
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `authPortable.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/authPortable.ts`
- 行数：19
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `autoModeDenials.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/autoModeDenials.ts`
- 行数：26
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `autoRunIssue.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/autoRunIssue.tsx`
- 行数：102
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils` 的职责见上文分类。

#### `autoUpdater.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/autoUpdater.ts`
- 行数：636
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `autoUpdaterRouting.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/autoUpdaterRouting.ts`
- 行数：36
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `aws.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/aws.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `awsAuthStatusManager.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/awsAuthStatusManager.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `preconditions.ts`

- 所在目录：`src/utils/background/remote`
- 完整路径：`src/utils/background/remote/preconditions.ts`
- 行数：235
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/background/remote` 的职责见上文分类。

#### `remoteSession.ts`

- 所在目录：`src/utils/background/remote`
- 完整路径：`src/utils/background/remote/remoteSession.ts`
- 行数：98
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/background/remote` 的职责见上文分类。

#### `backgroundHousekeeping.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/backgroundHousekeeping.ts`
- 行数：94
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `backgroundSessionTermination.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/backgroundSessionTermination.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `ParsedCommand.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/ParsedCommand.ts`
- 行数：318
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `ShellSnapshot.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/ShellSnapshot.ts`
- 行数：582
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `ast.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/ast.ts`
- 行数：2679
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2679 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `bashParser.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/bashParser.ts`
- 行数：4436
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 4436 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `bashPipeCommand.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/bashPipeCommand.ts`
- 行数：294
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `commands.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/commands.ts`
- 行数：1339
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1339 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `heredoc.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/heredoc.ts`
- 行数：733
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `parser.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/parser.ts`
- 行数：230
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `prefix.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/prefix.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `registry.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/registry.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellCompletion.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellCompletion.ts`
- 行数：259
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellPrefix.test.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellPrefix.test.ts`
- 行数：94
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/bash/shellPrefix.test.ts`。所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellPrefix.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellPrefix.ts`
- 行数：62
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellQuote.test.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellQuote.test.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/bash/shellQuote.test.ts`。所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellQuote.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellQuote.ts`
- 行数：343
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `shellQuoting.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/shellQuoting.ts`
- 行数：128
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/bash` 的职责见上文分类。

#### `alias.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/alias.ts`
- 行数：14
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/index.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `nohup.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/nohup.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `pyright.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/pyright.ts`
- 行数：91
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `sleep.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/sleep.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `srun.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/srun.ts`
- 行数：31
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `time.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/time.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `timeout.ts`

- 所在目录：`src/utils/bash/specs`
- 完整路径：`src/utils/bash/specs/timeout.ts`
- 行数：20
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/bash/specs` 的职责见上文分类。

#### `treeSitterAnalysis.ts`

- 所在目录：`src/utils/bash`
- 完整路径：`src/utils/bash/treeSitterAnalysis.ts`
- 行数：506
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/bash` 的职责见上文分类。

#### `betas.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/betas.test.ts`
- 行数：341
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/betas.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `betas.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/betas.ts`
- 行数：487
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `billing.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/billing.ts`
- 行数：78
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `binaryCheck.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/binaryCheck.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `boundedAsync.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/boundedAsync.test.ts`
- 行数：213
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/boundedAsync.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `boundedAsync.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/boundedAsync.ts`
- 行数：103
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `browser.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/browser.ts`
- 行数：68
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `bufferedWriter.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/bufferedWriter.ts`
- 行数：100
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `bundledMode.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/bundledMode.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `caCerts.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/caCerts.ts`
- 行数：115
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `caCertsConfig.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/caCertsConfig.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `cachePaths.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cachePaths.ts`
- 行数：38
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `classifierApprovals.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/classifierApprovals.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `classifierApprovalsHook.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/classifierApprovalsHook.ts`
- 行数：17
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `claudeCodeHints.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/claudeCodeHints.ts`
- 行数：266
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `claudeDesktop.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/claudeDesktop.test.ts`
- 行数：65
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/claudeDesktop.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `claudeDesktop.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/claudeDesktop.ts`
- 行数：186
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `chromeNativeHost.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/chromeNativeHost.ts`
- 行数：527
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `common.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/common.ts`
- 行数：540
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `mcpServer.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/mcpServer.ts`
- 行数：291
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `prompt.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/prompt.ts`
- 行数：83
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `setup.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/setup.ts`
- 行数：399
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `setupPortable.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/setupPortable.ts`
- 行数：234
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `startup.test.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/startup.test.ts`
- 行数：133
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/claudeInChrome/startup.test.ts`。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `startup.ts`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/startup.ts`
- 行数：76
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `toolRendering.tsx`

- 所在目录：`src/utils/claudeInChrome`
- 完整路径：`src/utils/claudeInChrome/toolRendering.tsx`
- 行数：261
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/claudeInChrome` 的职责见上文分类。

#### `claudemd.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/claudemd.ts`
- 行数：1522
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1522 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `cleanup.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cleanup.test.ts`
- 行数：48
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/cleanup.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `cleanup.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cleanup.ts`
- 行数：614
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `cleanupRegistry.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cleanupRegistry.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `cliArgs.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cliArgs.ts`
- 行数：78
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `cliHighlight.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cliHighlight.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `codeIndexing.proto.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/codeIndexing.proto.test.ts`
- 行数：37
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/codeIndexing.proto.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `codeIndexing.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/codeIndexing.ts`
- 行数：213
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `codexCredentials.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/codexCredentials.test.ts`
- 行数：957
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/codexCredentials.test.ts`。体量较大（约 957 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `codexCredentials.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/codexCredentials.ts`
- 行数：558
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `collapseBackgroundBashNotifications.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/collapseBackgroundBashNotifications.ts`
- 行数：84
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `collapseHookSummaries.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/collapseHookSummaries.ts`
- 行数：59
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `collapseReadSearch.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/collapseReadSearch.ts`
- 行数：1109
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1109 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `collapseTeammateShutdowns.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/collapseTeammateShutdowns.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `combinedAbortSignal.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/combinedAbortSignal.test.ts`
- 行数：190
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/combinedAbortSignal.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `combinedAbortSignal.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/combinedAbortSignal.ts`
- 行数：118
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `commandLifecycle.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/commandLifecycle.ts`
- 行数：21
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `commitAttribution.modelName.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/commitAttribution.modelName.test.ts`
- 行数：17
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/commitAttribution.modelName.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `commitAttribution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/commitAttribution.ts`
- 行数：963
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 963 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `completionCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/completionCache.ts`
- 行数：166
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `appNames.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/appNames.ts`
- 行数：196
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `cleanup.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/cleanup.ts`
- 行数：86
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `common.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/common.ts`
- 行数：61
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `computerUseLock.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/computerUseLock.ts`
- 行数：215
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `drainRunLoop.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/drainRunLoop.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `escHotkey.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/escHotkey.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `executor.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/executor.ts`
- 行数：658
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `gates.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/gates.ts`
- 行数：72
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `hostAdapter.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/hostAdapter.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `inputLoader.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/inputLoader.ts`
- 行数：30
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `mcpServer.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/mcpServer.ts`
- 行数：110
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `setup.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/setup.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `swiftLoader.ts`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/swiftLoader.ts`
- 行数：23
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `toolRendering.tsx`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/toolRendering.tsx`
- 行数：124
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `wrapper.tsx`

- 所在目录：`src/utils/computerUse`
- 完整路径：`src/utils/computerUse/wrapper.tsx`
- 行数：343
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/computerUse` 的职责见上文分类。

#### `concurrentSessions.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/concurrentSessions.ts`
- 行数：234
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `config.backupRecovery.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/config.backupRecovery.test.ts`
- 行数：413
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/config.backupRecovery.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `config.deferredWrite.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/config.deferredWrite.test.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/config.deferredWrite.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `config.showCacheStats.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/config.showCacheStats.test.ts`
- 行数：126
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/config.showCacheStats.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `config.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/config.ts`
- 行数：2226
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2226 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `configConstants.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/configConstants.ts`
- 行数：23
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `contentArray.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/contentArray.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `context.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/context.test.ts`
- 行数：1273
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/context.test.ts`。体量较大（约 1273 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `context.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/context.ts`
- 行数：447
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `contextAnalysis.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/contextAnalysis.ts`
- 行数：272
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `contextPartitioning.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/contextPartitioning.test.ts`
- 行数：86
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/contextPartitioning.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `contextPartitioning.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/contextPartitioning.ts`
- 行数：136
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `contextSuggestions.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/contextSuggestions.ts`
- 行数：235
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `continuation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/continuation.ts`
- 行数：215
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `controlMessageCompat.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/controlMessageCompat.ts`
- 行数：32
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `conversationArc.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationArc.test.ts`
- 行数：616
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/conversationArc.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `conversationArc.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationArc.ts`
- 行数：729
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `conversationCache.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationCache.test.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/conversationCache.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `conversationCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationCache.ts`
- 行数：172
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `conversationRecovery.hooks.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationRecovery.hooks.test.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/conversationRecovery.hooks.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `conversationRecovery.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationRecovery.test.ts`
- 行数：756
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/conversationRecovery.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `conversationRecovery.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/conversationRecovery.ts`
- 行数：863
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 863 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `copilotOptimization.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/copilotOptimization.test.ts`
- 行数：228
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/copilotOptimization.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `copilotOptimization.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/copilotOptimization.ts`
- 行数：96
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `cron.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cron.ts`
- 行数：308
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `cronJitterConfig.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cronJitterConfig.ts`
- 行数：76
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `cronScheduler.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cronScheduler.ts`
- 行数：565
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `cronTasks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cronTasks.ts`
- 行数：458
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `cronTasksLock.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cronTasksLock.ts`
- 行数：195
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `crossProjectResume.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/crossProjectResume.ts`
- 行数：75
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `crypto.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/crypto.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `cwd.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cwd.test.ts`
- 行数：39
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/cwd.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `cwd.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/cwd.ts`
- 行数：94
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `dangerousSkipFlags.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/dangerousSkipFlags.test.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/dangerousSkipFlags.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `dangerousSkipFlags.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/dangerousSkipFlags.ts`
- 行数：40
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `debug.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/debug.ts`
- 行数：266
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `debugFilter.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/debugFilter.ts`
- 行数：157
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `banner.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/banner.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `parseDeepLink.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/parseDeepLink.ts`
- 行数：170
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `protocolHandler.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/protocolHandler.ts`
- 行数：136
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `registerProtocol.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/registerProtocol.ts`
- 行数：354
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `terminalLauncher.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/terminalLauncher.ts`
- 行数：557
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `terminalPreference.ts`

- 所在目录：`src/utils/deepLink`
- 完整路径：`src/utils/deepLink/terminalPreference.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/deepLink` 的职责见上文分类。

#### `deferredConfigWrites.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/deferredConfigWrites.test.ts`
- 行数：207
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/deferredConfigWrites.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `deferredConfigWrites.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/deferredConfigWrites.ts`
- 行数：82
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `desktopDeepLink.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/desktopDeepLink.ts`
- 行数：236
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `detectRepository.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/detectRepository.ts`
- 行数：178
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `devChannelRegistration.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/devChannelRegistration.ts`
- 行数：30
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `diagLogs.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/diagLogs.ts`
- 行数：129
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `issueReport.test.ts`

- 所在目录：`src/utils/diagnostics`
- 完整路径：`src/utils/diagnostics/issueReport.test.ts`
- 行数：348
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/diagnostics/issueReport.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/diagnostics` 的职责见上文分类。

#### `issueReport.ts`

- 所在目录：`src/utils/diagnostics`
- 完整路径：`src/utils/diagnostics/issueReport.ts`
- 行数：779
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/diagnostics` 的职责见上文分类。

#### `redaction.test.ts`

- 所在目录：`src/utils/diagnostics`
- 完整路径：`src/utils/diagnostics/redaction.test.ts`
- 行数：779
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/diagnostics/redaction.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/diagnostics` 的职责见上文分类。

#### `diff.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/diff.test.ts`
- 行数：85
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/diff.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `diff.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/diff.ts`
- 行数：178
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `directMemberMessage.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/directMemberMessage.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `displayTags.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/displayTags.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `doctorContextWarnings.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doctorContextWarnings.ts`
- 行数：379
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `doctorDiagnostic.settingsPath.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doctorDiagnostic.settingsPath.test.ts`
- 行数：97
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/doctorDiagnostic.settingsPath.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `doctorDiagnostic.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doctorDiagnostic.test.ts`
- 行数：32
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/doctorDiagnostic.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `doctorDiagnostic.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doctorDiagnostic.ts`
- 行数：701
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `doomLoop.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doomLoop.test.ts`
- 行数：131
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/doomLoop.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `doomLoop.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/doomLoop.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `dragDropPaths.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/dragDropPaths.test.ts`
- 行数：113
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/dragDropPaths.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `dragDropPaths.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/dragDropPaths.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `helpers.ts`

- 所在目录：`src/utils/dxt`
- 完整路径：`src/utils/dxt/helpers.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/dxt` 的职责见上文分类。

#### `zip.ts`

- 所在目录：`src/utils/dxt`
- 完整路径：`src/utils/dxt/zip.ts`
- 行数：226
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/dxt` 的职责见上文分类。

#### `earlyInput.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/earlyInput.ts`
- 行数：191
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `editor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/editor.ts`
- 行数：183
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `effort.codex.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/effort.codex.test.ts`
- 行数：1732
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/effort.codex.test.ts`。体量较大（约 1732 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `effort.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/effort.test.ts`
- 行数：440
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/effort.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `effort.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/effort.ts`
- 行数：1359
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1359 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `effort.ultracode-display.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/effort.ultracode-display.test.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/effort.ultracode-display.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `embeddedTools.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/embeddedTools.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `env.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/env.test.ts`
- 行数：233
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/env.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `env.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/env.ts`
- 行数：385
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `envDynamic.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envDynamic.ts`
- 行数：151
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `envFile.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envFile.test.ts`
- 行数：539
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/envFile.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `envFile.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envFile.ts`
- 行数：381
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `envProviderOption.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envProviderOption.test.ts`
- 行数：134
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/envProviderOption.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `envProviderOption.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envProviderOption.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `envUtils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envUtils.ts`
- 行数：258
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `envValidation.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envValidation.test.ts`
- 行数：43
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/envValidation.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `envValidation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/envValidation.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `errorLogSink.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/errorLogSink.ts`
- 行数：239
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `errors.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/errors.ts`
- 行数：343
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `exampleCommands.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/exampleCommands.ts`
- 行数：184
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `execFileNoThrow.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/execFileNoThrow.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/execFileNoThrow.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `execFileNoThrow.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/execFileNoThrow.ts`
- 行数：338
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `execFileNoThrowPortable.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/execFileNoThrowPortable.ts`
- 行数：92
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `execSyncWrapper.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/execSyncWrapper.ts`
- 行数：38
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `exportFormats.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/exportFormats.test.ts`
- 行数：245
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/exportFormats.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `exportFormats.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/exportFormats.ts`
- 行数：195
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `exportRenderer.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/exportRenderer.test.ts`
- 行数：976
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/exportRenderer.test.ts`。体量较大（约 976 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `exportRenderer.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/exportRenderer.tsx`
- 行数：782
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `extraUsage.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/extraUsage.test.ts`
- 行数：44
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/extraUsage.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `extraUsage.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/extraUsage.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `fastMode.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fastMode.test.ts`
- 行数：319
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/fastMode.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `fastMode.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fastMode.ts`
- 行数：536
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `file.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/file.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/file.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `file.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/file.ts`
- 行数：584
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `fileHistory.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fileHistory.ts`
- 行数：1104
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1104 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `fileOperationAnalytics.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fileOperationAnalytics.ts`
- 行数：71
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `filePersistence.ts`

- 所在目录：`src/utils/filePersistence`
- 完整路径：`src/utils/filePersistence/filePersistence.ts`
- 行数：287
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/filePersistence` 的职责见上文分类。

#### `outputsScanner.ts`

- 所在目录：`src/utils/filePersistence`
- 完整路径：`src/utils/filePersistence/outputsScanner.ts`
- 行数：130
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/filePersistence` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/utils/filePersistence`
- 完整路径：`src/utils/filePersistence/types.ts`
- 行数：21
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/filePersistence` 的职责见上文分类。

#### `fileRead.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fileRead.ts`
- 行数：102
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `fileReadCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fileReadCache.ts`
- 行数：96
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `fileStateCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fileStateCache.ts`
- 行数：142
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `findExecutable.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/findExecutable.ts`
- 行数：17
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `fingerprint.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fingerprint.ts`
- 行数：82
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `forkedAgent.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/forkedAgent.ts`
- 行数：713
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `format.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/format.test.ts`
- 行数：65
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/format.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `format.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/format.ts`
- 行数：330
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `formatBriefTimestamp.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/formatBriefTimestamp.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `fpsTracker.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fpsTracker.test.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/fpsTracker.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `fpsTracker.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fpsTracker.ts`
- 行数：66
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `frontmatterParser.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/frontmatterParser.test.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/frontmatterParser.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `frontmatterParser.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/frontmatterParser.ts`
- 行数：431
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `fsOperations.mkdir.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fsOperations.mkdir.test.ts`
- 行数：58
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/fsOperations.mkdir.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `fsOperations.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fsOperations.ts`
- 行数：962
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 962 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `fullscreen.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/fullscreen.ts`
- 行数：214
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `geminiAuth.optionalRuntime.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/geminiAuth.optionalRuntime.test.ts`
- 行数：61
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/geminiAuth.optionalRuntime.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `geminiAuth.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/geminiAuth.test.ts`
- 行数：283
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/geminiAuth.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `geminiAuth.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/geminiAuth.ts`
- 行数：256
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `geminiCredentials.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/geminiCredentials.test.ts`
- 行数：73
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/geminiCredentials.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `geminiCredentials.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/geminiCredentials.ts`
- 行数：76
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `generatedFiles.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/generatedFiles.ts`
- 行数：136
- 主要功能简介：共享工具函数。
- 说明：生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `src/utils` 的职责见上文分类。

#### `generators.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/generators.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `genericProcessUtils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/genericProcessUtils.ts`
- 行数：184
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `getWorktreePaths.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/getWorktreePaths.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `getWorktreePathsPortable.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/getWorktreePathsPortable.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `ghPrStatus.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ghPrStatus.ts`
- 行数：106
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `git.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/git.ts`
- 行数：926
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 926 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `gitConfigParser.ts`

- 所在目录：`src/utils/git`
- 完整路径：`src/utils/git/gitConfigParser.ts`
- 行数：277
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/git` 的职责见上文分类。

#### `gitFilesystem.ts`

- 所在目录：`src/utils/git`
- 完整路径：`src/utils/git/gitFilesystem.ts`
- 行数：699
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/git` 的职责见上文分类。

#### `gitignore.ts`

- 所在目录：`src/utils/git`
- 完整路径：`src/utils/git/gitignore.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/git` 的职责见上文分类。

#### `gitDiff.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gitDiff.test.ts`
- 行数：133
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/gitDiff.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `gitDiff.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gitDiff.ts`
- 行数：548
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `gitSettings.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gitSettings.test.ts`
- 行数：107
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/gitSettings.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `gitSettings.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gitSettings.ts`
- 行数：31
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `ghAuthStatus.ts`

- 所在目录：`src/utils/github`
- 完整路径：`src/utils/github/ghAuthStatus.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/github` 的职责见上文分类。

#### `githubModelsCredentials.hydrate.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/githubModelsCredentials.hydrate.test.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/githubModelsCredentials.hydrate.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `githubModelsCredentials.refresh.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/githubModelsCredentials.refresh.test.ts`
- 行数：422
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/githubModelsCredentials.refresh.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `githubModelsCredentials.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/githubModelsCredentials.test.ts`
- 行数：160
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/githubModelsCredentials.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `githubModelsCredentials.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/githubModelsCredentials.ts`
- 行数：318
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `githubRepoPathMapping.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/githubRepoPathMapping.ts`
- 行数：162
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `glob.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/glob.ts`
- 行数：130
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `globalPackageManager.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/globalPackageManager.test.ts`
- 行数：124
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/globalPackageManager.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `globalPackageManager.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/globalPackageManager.ts`
- 行数：212
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `governancePolicy.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/governancePolicy.test.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/governancePolicy.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `governancePolicy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/governancePolicy.ts`
- 行数：103
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `gracefulShutdown.interruptionTrace.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gracefulShutdown.interruptionTrace.test.ts`
- 行数：60
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/gracefulShutdown.interruptionTrace.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `gracefulShutdown.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/gracefulShutdown.ts`
- 行数：568
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `groupToolUses.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/groupToolUses.ts`
- 行数：182
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `handlePromptSubmit.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/handlePromptSubmit.test.ts`
- 行数：732
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/handlePromptSubmit.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `handlePromptSubmit.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/handlePromptSubmit.ts`
- 行数：714
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hash.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hash.ts`
- 行数：46
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `headlessProfiler.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/headlessProfiler.ts`
- 行数：179
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `heapDumpService.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/heapDumpService.test.ts`
- 行数：66
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/heapDumpService.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `heapDumpService.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/heapDumpService.ts`
- 行数：355
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `heatmap.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/heatmap.ts`
- 行数：198
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `highlightMatch.test.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/highlightMatch.test.tsx`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/utils/highlightMatch.test.tsx`。所在目录 `src/utils` 的职责见上文分类。

#### `highlightMatch.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/highlightMatch.tsx`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils` 的职责见上文分类。

#### `hookChains.integration.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hookChains.integration.test.ts`
- 行数：400
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/hookChains.integration.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hookChains.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hookChains.test.ts`
- 行数：485
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/hookChains.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hookChains.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hookChains.ts`
- 行数：1518
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1518 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hooks.fallbackAgent.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hooks.fallbackAgent.test.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/hooks.fallbackAgent.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `hooks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hooks.ts`
- 行数：5281
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 5281 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `AsyncHookRegistry.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/AsyncHookRegistry.ts`
- 行数：309
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `apiQueryHookHelper.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/apiQueryHookHelper.ts`
- 行数：141
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `execAgentHook.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/execAgentHook.ts`
- 行数：339
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `execHttpHook.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/execHttpHook.ts`
- 行数：242
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `execPromptHook.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/execPromptHook.ts`
- 行数：211
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `fileChangedWatcher.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/fileChangedWatcher.ts`
- 行数：191
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hookEvents.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hookEvents.ts`
- 行数：192
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hookHelpers.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hookHelpers.ts`
- 行数：83
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hooksConfigManager.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hooksConfigManager.ts`
- 行数：400
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hooksConfigSnapshot.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hooksConfigSnapshot.ts`
- 行数：133
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hooksSettings.test.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hooksSettings.test.ts`
- 行数：13
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/hooks/hooksSettings.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `hooksSettings.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/hooksSettings.ts`
- 行数：281
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `postSamplingHooks.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/postSamplingHooks.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `registerFrontmatterHooks.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/registerFrontmatterHooks.ts`
- 行数：67
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `registerSkillHooks.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/registerSkillHooks.ts`
- 行数：64
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/hooks` 的职责见上文分类。

#### `sessionHooks.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/sessionHooks.ts`
- 行数：447
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `skillImprovement.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/skillImprovement.ts`
- 行数：267
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `ssrfGuard.ts`

- 所在目录：`src/utils/hooks`
- 完整路径：`src/utils/hooks/ssrfGuard.ts`
- 行数：294
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/hooks` 的职责见上文分类。

#### `horizontalScroll.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/horizontalScroll.ts`
- 行数：137
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `http.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/http.test.ts`
- 行数：49
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/http.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `http.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/http.ts`
- 行数：141
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `hybridContextStrategy.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hybridContextStrategy.test.ts`
- 行数：230
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/hybridContextStrategy.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hybridContextStrategy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hybridContextStrategy.ts`
- 行数：311
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `hyperlink.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/hyperlink.ts`
- 行数：39
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `iTermBackup.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/iTermBackup.ts`
- 行数：73
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `ide.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ide.ts`
- 行数：1507
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1507 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `idePathConversion.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/idePathConversion.ts`
- 行数：90
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `idleTimeout.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/idleTimeout.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `imageMockLifecycle.consumer.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imageMockLifecycle.consumer.test.ts`
- 行数：50
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/imageMockLifecycle.consumer.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `imagePaste.clipboard-error.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imagePaste.clipboard-error.test.ts`
- 行数：86
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/imagePaste.clipboard-error.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `imagePaste.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imagePaste.test.ts`
- 行数：115
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/imagePaste.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `imagePaste.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imagePaste.ts`
- 行数：518
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `imagePaste.win32.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imagePaste.win32.test.ts`
- 行数：247
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/imagePaste.win32.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `imageResizer.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imageResizer.test.ts`
- 行数：842
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/imageResizer.test.ts`。体量较大（约 842 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `imageResizer.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imageResizer.ts`
- 行数：1333
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1333 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `imageStore.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imageStore.ts`
- 行数：167
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `imageValidation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/imageValidation.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `immediateCommand.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/immediateCommand.ts`
- 行数：15
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `inProcessTeammateHelpers.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/inProcessTeammateHelpers.ts`
- 行数：102
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `incrementalTokenCounter.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/incrementalTokenCounter.test.ts`
- 行数：300
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/incrementalTokenCounter.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `incrementalTokenCounter.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/incrementalTokenCounter.ts`
- 行数：257
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `ink.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ink.ts`
- 行数：26
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `interactivity.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interactivity.test.ts`
- 行数：110
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/interactivity.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `interactivity.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interactivity.ts`
- 行数：28
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `interruptionCorrection.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interruptionCorrection.test.ts`
- 行数：615
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/interruptionCorrection.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `interruptionCorrection.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interruptionCorrection.ts`
- 行数：256
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `interruptionTrace.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interruptionTrace.test.ts`
- 行数：923
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/interruptionTrace.test.ts`。体量较大（约 923 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `interruptionTrace.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/interruptionTrace.ts`
- 行数：790
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `intl.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/intl.ts`
- 行数：94
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `jetbrains.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/jetbrains.ts`
- 行数：191
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `json.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/json.ts`
- 行数：277
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `jsonRead.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/jsonRead.ts`
- 行数：16
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `keyboardShortcuts.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/keyboardShortcuts.ts`
- 行数：14
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `knowledgeGraph.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/knowledgeGraph.test.ts`
- 行数：537
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/knowledgeGraph.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `knowledgeGraph.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/knowledgeGraph.ts`
- 行数：1068
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1068 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `lazySchema.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/lazySchema.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `listSessionsImpl.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/listSessionsImpl.ts`
- 行数：454
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `localInstaller.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/localInstaller.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `lockfile.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/lockfile.ts`
- 行数：43
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `log.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/log.test.ts`
- 行数：110
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/log.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `log.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/log.ts`
- 行数：408
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `logoV2Utils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/logoV2Utils.ts`
- 行数：353
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `mailbox.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mailbox.ts`
- 行数：73
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `managedEnv.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/managedEnv.test.ts`
- 行数：169
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/managedEnv.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `managedEnv.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/managedEnv.ts`
- 行数：220
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `managedEnvConstants.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/managedEnvConstants.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `markdown.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/markdown.ts`
- 行数：381
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `markdownConfigLoader.scaling.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/markdownConfigLoader.scaling.test.ts`
- 行数：155
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/markdownConfigLoader.scaling.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `markdownConfigLoader.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/markdownConfigLoader.ts`
- 行数：697
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `maxActiveMessages.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/maxActiveMessages.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/maxActiveMessages.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `maxActiveMessages.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/maxActiveMessages.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `dateTimeParser.ts`

- 所在目录：`src/utils/mcp`
- 完整路径：`src/utils/mcp/dateTimeParser.ts`
- 行数：121
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/mcp` 的职责见上文分类。

#### `elicitationValidation.ts`

- 所在目录：`src/utils/mcp`
- 完整路径：`src/utils/mcp/elicitationValidation.ts`
- 行数：336
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/mcp` 的职责见上文分类。

#### `mcpInstructionsDelta.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mcpInstructionsDelta.ts`
- 行数：130
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `mcpOutputStorage.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mcpOutputStorage.ts`
- 行数：197
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `mcpValidation.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mcpValidation.test.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/mcpValidation.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `mcpValidation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mcpValidation.ts`
- 行数：220
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `mcpWebSocketTransport.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mcpWebSocketTransport.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `memoize.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/memoize.ts`
- 行数：269
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/utils/memory`
- 完整路径：`src/utils/memory/types.ts`
- 行数：12
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/memory` 的职责见上文分类。

#### `versions.ts`

- 所在目录：`src/utils/memory`
- 完整路径：`src/utils/memory/versions.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/memory` 的职责见上文分类。

#### `memoryCompaction.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/memoryCompaction.ts`
- 行数：42
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `memoryFileDetection.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/memoryFileDetection.ts`
- 行数：289
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `memoryPressure.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/memoryPressure.ts`
- 行数：160
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `messageFilters.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/messageFilters.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `messagePredicates.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/messagePredicates.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `messageQueueManager.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/messageQueueManager.test.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messageQueueManager.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `messageQueueManager.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/messageQueueManager.ts`
- 行数：569
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `messages.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/messages.ts`
- 行数：4226
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 4226 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `apiTransform.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/apiTransform.test.ts`
- 行数：417
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/apiTransform.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `apiTransform.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/apiTransform.ts`
- 行数：262
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `content.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/content.test.ts`
- 行数：89
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/content.test.ts`。所在目录 `src/utils/messages` 的职责见上文分类。

#### `content.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/content.ts`
- 行数：147
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/messages` 的职责见上文分类。

#### `factories.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/factories.test.ts`
- 行数：198
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/factories.test.ts`。所在目录 `src/utils/messages` 的职责见上文分类。

#### `factories.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/factories.ts`
- 行数：381
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `mappers.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/mappers.ts`
- 行数：291
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `normalize.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/normalize.test.ts`
- 行数：172
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/normalize.test.ts`。所在目录 `src/utils/messages` 的职责见上文分类。

#### `normalize.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/normalize.ts`
- 行数：237
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `planMode.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/planMode.test.ts`
- 行数：92
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/planMode.test.ts`。所在目录 `src/utils/messages` 的职责见上文分类。

#### `planMode.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/planMode.ts`
- 行数：371
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `streaming.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/streaming.test.ts`
- 行数：78
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/streaming.test.ts`。所在目录 `src/utils/messages` 的职责见上文分类。

#### `streaming.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/streaming.ts`
- 行数：198
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/messages` 的职责见上文分类。

#### `systemFactories.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/systemFactories.test.ts`
- 行数：233
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/systemFactories.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `systemFactories.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/systemFactories.ts`
- 行数：329
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `systemInit.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/systemInit.ts`
- 行数：96
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/messages` 的职责见上文分类。

#### `toolPairing.test.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/toolPairing.test.ts`
- 行数：361
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/messages/toolPairing.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `toolPairing.ts`

- 所在目录：`src/utils/messages`
- 完整路径：`src/utils/messages/toolPairing.ts`
- 行数：594
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/messages` 的职责见上文分类。

#### `agent.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/agent.test.ts`
- 行数：608
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/agent.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `agent.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/agent.ts`
- 行数：239
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `aliases.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/aliases.ts`
- 行数：25
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/model` 的职责见上文分类。

#### `antModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/antModels.ts`
- 行数：64
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `bedrock.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/bedrock.ts`
- 行数：273
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `check1mAccess.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/check1mAccess.ts`
- 行数：72
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `configs.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/configs.ts`
- 行数：293
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `contextWindowUpgradeCheck.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/contextWindowUpgradeCheck.ts`
- 行数：47
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `copilotModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/copilotModels.ts`
- 行数：383
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `deprecation.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/deprecation.ts`
- 行数：128
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `minimaxModels.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/minimaxModels.test.ts`
- 行数：33
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/minimaxModels.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/model` 的职责见上文分类。

#### `minimaxModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/minimaxModels.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `model.github.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/model.github.test.ts`
- 行数：76
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/model.github.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `model.openai-shim-providers.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/model.openai-shim-providers.test.ts`
- 行数：591
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/model.openai-shim-providers.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `model.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/model.ts`
- 行数：1078
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1078 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelAllowlist.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelAllowlist.ts`
- 行数：170
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `modelCapabilities.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelCapabilities.ts`
- 行数：16
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.catalogDedup.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.catalogDedup.test.ts`
- 行数：401
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.catalogDedup.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.codexRecovery.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.codexRecovery.test.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.codexRecovery.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.crossProfile.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.crossProfile.test.ts`
- 行数：921
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.crossProfile.test.ts`。体量较大（约 921 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.gateways.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.gateways.test.ts`
- 行数：273
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.gateways.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.github.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.github.test.ts`
- 行数：132
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.github.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.hicap.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.hicap.test.ts`
- 行数：171
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.hicap.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.switchMarker.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.switchMarker.test.ts`
- 行数：46
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.switchMarker.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.ts`
- 行数：1234
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1234 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelOptions.xiaomi-mimo.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelOptions.xiaomi-mimo.test.ts`
- 行数：101
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelOptions.xiaomi-mimo.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelStrings.github.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelStrings.github.test.ts`
- 行数：71
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/modelStrings.github.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelStrings.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelStrings.ts`
- 行数：173
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `modelSupportOverrides.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/modelSupportOverrides.ts`
- 行数：80
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `nvidiaNimModels.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/nvidiaNimModels.test.ts`
- 行数：186
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/nvidiaNimModels.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `nvidiaNimModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/nvidiaNimModels.ts`
- 行数：233
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `ollamaModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/ollamaModels.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `openaiContextWindows.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/openaiContextWindows.test.ts`
- 行数：167
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/openaiContextWindows.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `openaiContextWindows.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/openaiContextWindows.ts`
- 行数：246
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `openaiModelDiscovery.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/openaiModelDiscovery.test.ts`
- 行数：197
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/openaiModelDiscovery.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `openaiModelDiscovery.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/openaiModelDiscovery.ts`
- 行数：210
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `parseUserSpecifiedModel.bestTag.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/parseUserSpecifiedModel.bestTag.test.ts`
- 行数：30
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/parseUserSpecifiedModel.bestTag.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/model` 的职责见上文分类。

#### `parseUserSpecifiedModel.codexTag.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/parseUserSpecifiedModel.codexTag.test.ts`
- 行数：115
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/parseUserSpecifiedModel.codexTag.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `parseUserSpecifiedModel.disable1m.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/parseUserSpecifiedModel.disable1m.test.ts`
- 行数：143
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/parseUserSpecifiedModel.disable1m.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `providers.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/providers.test.ts`
- 行数：341
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/providers.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `providers.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/providers.ts`
- 行数：132
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `routeCatalogOptions.test.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/routeCatalogOptions.test.ts`
- 行数：114
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/model/routeCatalogOptions.test.ts`。所在目录 `src/utils/model` 的职责见上文分类。

#### `routeCatalogOptions.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/routeCatalogOptions.ts`
- 行数：89
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/model` 的职责见上文分类。

#### `validateModel.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/validateModel.ts`
- 行数：241
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/model` 的职责见上文分类。

#### `xiaomi-mimoModels.ts`

- 所在目录：`src/utils/model`
- 完整路径：`src/utils/model/xiaomi-mimoModels.ts`
- 行数：35
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/model` 的职责见上文分类。

#### `modelCost.modelGate.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/modelCost.modelGate.test.ts`
- 行数：276
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/modelCost.modelGate.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `modelCost.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/modelCost.ts`
- 行数：265
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `modifiers.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/modifiers.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `mtls.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/mtls.ts`
- 行数：179
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `multiTurnContext.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/multiTurnContext.test.ts`
- 行数：225
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/multiTurnContext.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `multiTurnContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/multiTurnContext.ts`
- 行数：158
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `nativeDistribution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/nativeDistribution.ts`
- 行数：21
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `download.ts`

- 所在目录：`src/utils/nativeInstaller`
- 完整路径：`src/utils/nativeInstaller/download.ts`
- 行数：529
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/nativeInstaller` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/utils/nativeInstaller`
- 完整路径：`src/utils/nativeInstaller/index.ts`
- 行数：19
- 主要功能简介：共享工具函数。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/nativeInstaller` 的职责见上文分类。

#### `installer.ts`

- 所在目录：`src/utils/nativeInstaller`
- 完整路径：`src/utils/nativeInstaller/installer.ts`
- 行数：1774
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1774 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/nativeInstaller` 的职责见上文分类。

#### `packageManagers.ts`

- 所在目录：`src/utils/nativeInstaller`
- 完整路径：`src/utils/nativeInstaller/packageManagers.ts`
- 行数：336
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/nativeInstaller` 的职责见上文分类。

#### `pidLock.ts`

- 所在目录：`src/utils/nativeInstaller`
- 完整路径：`src/utils/nativeInstaller/pidLock.ts`
- 行数：433
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/nativeInstaller` 的职责见上文分类。

#### `nodeRuntime.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/nodeRuntime.test.ts`
- 行数：59
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/nodeRuntime.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `nodeRuntime.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/nodeRuntime.ts`
- 行数：56
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `notebook.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/notebook.ts`
- 行数：224
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `objectGroupBy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/objectGroupBy.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `ollamaContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ollamaContext.ts`
- 行数：101
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `openclaudeDisplayPaths.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/openclaudeDisplayPaths.ts`
- 行数：33
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `openclaudeInstallSurfaces.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/openclaudeInstallSurfaces.test.ts`
- 行数：553
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/openclaudeInstallSurfaces.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `openclaudePaths.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/openclaudePaths.test.ts`
- 行数：380
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/openclaudePaths.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `openclaudeUiSurfaces.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/openclaudeUiSurfaces.test.ts`
- 行数：218
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/openclaudeUiSurfaces.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `opencodeProfile.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/opencodeProfile.test.ts`
- 行数：440
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/opencodeProfile.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `optionalRuntimeModule.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/optionalRuntimeModule.test.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/optionalRuntimeModule.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `optionalRuntimeModule.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/optionalRuntimeModule.ts`
- 行数：91
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `packageManagerUpdateGuidance.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/packageManagerUpdateGuidance.test.ts`
- 行数：64
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/packageManagerUpdateGuidance.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `packageManagerUpdateGuidance.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/packageManagerUpdateGuidance.ts`
- 行数：52
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `pasteStore.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/pasteStore.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `path.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/path.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `pdf.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/pdf.ts`
- 行数：300
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `pdfUtils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/pdfUtils.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `peerAddress.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/peerAddress.ts`
- 行数：21
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `PermissionMode.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionMode.ts`
- 行数：154
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionPromptToolResultSchema.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionPromptToolResultSchema.ts`
- 行数：173
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionResult.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionResult.ts`
- 行数：35
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionRule.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionRule.ts`
- 行数：40
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionUpdate.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionUpdate.test.ts`
- 行数：25
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/PermissionUpdate.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionUpdate.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionUpdate.ts`
- 行数：402
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `PermissionUpdateSchema.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/PermissionUpdateSchema.ts`
- 行数：78
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `autoModeState.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/autoModeState.ts`
- 行数：39
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `bashClassifier.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/bashClassifier.ts`
- 行数：61
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `bypassPermissionsKillswitch.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/bypassPermissionsKillswitch.test.ts`
- 行数：104
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/bypassPermissionsKillswitch.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `bypassPermissionsKillswitch.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/bypassPermissionsKillswitch.ts`
- 行数：181
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `classifierDecision.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/classifierDecision.ts`
- 行数：98
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `classifierShared.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/classifierShared.ts`
- 行数：39
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePrompt.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePrompt.test.ts`
- 行数：78
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/dangerousModePrompt.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePrompt.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePrompt.ts`
- 行数：56
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePromptFlow.test.tsx`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePromptFlow.test.tsx`
- 行数：144
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/utils/permissions/dangerousModePromptFlow.test.tsx`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePromptFlow.tsx`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePromptFlow.tsx`
- 行数：53
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePromptRuntime.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePromptRuntime.test.ts`
- 行数：71
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/dangerousModePromptRuntime.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousModePromptRuntime.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousModePromptRuntime.ts`
- 行数：36
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `dangerousPatterns.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/dangerousPatterns.ts`
- 行数：81
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `defaultPermissionModeOptions.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/defaultPermissionModeOptions.ts`
- 行数：33
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `denialTracking.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/denialTracking.ts`
- 行数：45
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `filesystem.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/filesystem.test.ts`
- 行数：294
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/filesystem.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `filesystem.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/filesystem.ts`
- 行数：2003
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：体量较大（约 2003 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `getNextPermissionMode.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/getNextPermissionMode.test.ts`
- 行数：25
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/getNextPermissionMode.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `getNextPermissionMode.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/getNextPermissionMode.ts`
- 行数：107
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `pathValidation.expandTilde.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/pathValidation.expandTilde.test.ts`
- 行数：43
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/pathValidation.expandTilde.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `pathValidation.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/pathValidation.ts`
- 行数：496
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionExplainer.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionExplainer.ts`
- 行数：250
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionModeChange.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionModeChange.ts`
- 行数：112
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionRuleParser.protoName.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionRuleParser.protoName.test.ts`
- 行数：68
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/permissionRuleParser.protoName.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionRuleParser.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionRuleParser.ts`
- 行数：207
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionSetup.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionSetup.test.ts`
- 行数：344
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/permissionSetup.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionSetup.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionSetup.ts`
- 行数：1848
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：体量较大（约 1848 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissions.headlessPlanHooks.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissions.headlessPlanHooks.test.ts`
- 行数：1253
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/permissions.headlessPlanHooks.test.ts`。体量较大（约 1253 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissions.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissions.test.ts`
- 行数：831
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/permissions.test.ts`。体量较大（约 831 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissions.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissions.ts`
- 行数：1898
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：体量较大（约 1898 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `permissionsLoader.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/permissionsLoader.ts`
- 行数：296
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `planFilePath.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/planFilePath.test.ts`
- 行数：544
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/planFilePath.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `safetyLevel.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/safetyLevel.test.ts`
- 行数：35
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/safetyLevel.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `safetyLevel.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/safetyLevel.ts`
- 行数：51
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：所在目录 `src/utils/permissions` 的职责见上文分类。

#### `shadowedRuleDetection.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/shadowedRuleDetection.ts`
- 行数：234
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `shellRuleMatching.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/shellRuleMatching.ts`
- 行数：228
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `yoloClassifier.test.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/yoloClassifier.test.ts`
- 行数：79
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：运行方式：`bun test ./src/utils/permissions/yoloClassifier.test.ts`。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `yoloClassifier.ts`

- 所在目录：`src/utils/permissions`
- 完整路径：`src/utils/permissions/yoloClassifier.ts`
- 行数：1603
- 主要功能简介：权限规则、分类器、路径校验。
- 说明：体量较大（约 1603 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/permissions` 的职责见上文分类。

#### `planModeV2.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/planModeV2.ts`
- 行数：92
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `plans.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/plans.ts`
- 行数：740
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `platform.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/platform.ts`
- 行数：150
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `addDirPluginSettings.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/addDirPluginSettings.ts`
- 行数：71
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `cacheUtils.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/cacheUtils.ts`
- 行数：196
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `dependencyResolver.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/dependencyResolver.ts`
- 行数：305
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `fetchTelemetry.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/fetchTelemetry.ts`
- 行数：135
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `gitAvailability.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/gitAvailability.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `gitEnv.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/gitEnv.test.ts`
- 行数：113
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/gitEnv.test.ts`。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `gitEnv.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/gitEnv.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `headlessPluginInstall.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/headlessPluginInstall.ts`
- 行数：174
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `hintRecommendation.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/hintRecommendation.ts`
- 行数：164
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `installCounts.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/installCounts.ts`
- 行数：292
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `installedPluginsManager.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/installedPluginsManager.ts`
- 行数：1268
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1268 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `loadPluginAgents.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/loadPluginAgents.test.ts`
- 行数：143
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/loadPluginAgents.test.ts`。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `loadPluginAgents.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/loadPluginAgents.ts`
- 行数：361
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `loadPluginCommands.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/loadPluginCommands.ts`
- 行数：980
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 980 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `loadPluginHooks.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/loadPluginHooks.ts`
- 行数：287
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `loadPluginOutputStyles.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/loadPluginOutputStyles.ts`
- 行数：178
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `lspPluginIntegration.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/lspPluginIntegration.ts`
- 行数：387
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `lspRecommendation.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/lspRecommendation.test.ts`
- 行数：269
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/lspRecommendation.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `lspRecommendation.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/lspRecommendation.ts`
- 行数：412
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `managedPlugins.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/managedPlugins.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `marketplaceHelpers.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/marketplaceHelpers.ts`
- 行数：601
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `marketplaceHostPattern.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/marketplaceHostPattern.test.ts`
- 行数：97
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/marketplaceHostPattern.test.ts`。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `marketplaceManager.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/marketplaceManager.test.ts`
- 行数：902
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/marketplaceManager.test.ts`。体量较大（约 902 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `marketplaceManager.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/marketplaceManager.ts`
- 行数：2799
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2799 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `mcpPluginIntegration.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/mcpPluginIntegration.ts`
- 行数：634
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `mcpbHandler.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/mcpbHandler.ts`
- 行数：968
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 968 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `officialMarketplace.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/officialMarketplace.ts`
- 行数：25
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `officialMarketplaceGcs.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/officialMarketplaceGcs.ts`
- 行数：216
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `officialMarketplaceStartupCheck.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/officialMarketplaceStartupCheck.test.ts`
- 行数：209
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/officialMarketplaceStartupCheck.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `officialMarketplaceStartupCheck.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/officialMarketplaceStartupCheck.ts`
- 行数：436
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `orphanedPluginFilter.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/orphanedPluginFilter.ts`
- 行数：114
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `parseMarketplaceInput.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/parseMarketplaceInput.ts`
- 行数：162
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `performStartupChecks.tsx`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/performStartupChecks.tsx`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginAutoupdate.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginAutoupdate.ts`
- 行数：284
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginBlocklist.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginBlocklist.ts`
- 行数：127
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginDirectories.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginDirectories.ts`
- 行数：178
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginFlagging.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginFlagging.ts`
- 行数：208
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginIdentifier.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginIdentifier.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginInstallationHelpers.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginInstallationHelpers.ts`
- 行数：595
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginLoader.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginLoader.test.ts`
- 行数：509
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/pluginLoader.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginLoader.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginLoader.ts`
- 行数：3585
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 3585 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginOptionsStorage.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginOptionsStorage.ts`
- 行数：400
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginPolicy.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginPolicy.ts`
- 行数：20
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginStartupCheck.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginStartupCheck.ts`
- 行数：341
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `pluginVersioning.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/pluginVersioning.ts`
- 行数：157
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `reconciler.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/reconciler.test.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/reconciler.test.ts`。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `reconciler.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/reconciler.ts`
- 行数：271
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `refresh.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/refresh.ts`
- 行数：220
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `schemas.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/schemas.ts`
- 行数：1721
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1721 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `validateOfficialNameSource.test.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/validateOfficialNameSource.test.ts`
- 行数：68
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/plugins/validateOfficialNameSource.test.ts`。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `validatePlugin.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/validatePlugin.ts`
- 行数：903
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 903 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `walkPluginMarkdown.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/walkPluginMarkdown.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `zipCache.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/zipCache.ts`
- 行数：406
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/plugins` 的职责见上文分类。

#### `zipCacheAdapters.ts`

- 所在目录：`src/utils/plugins`
- 完整路径：`src/utils/plugins/zipCacheAdapters.ts`
- 行数：164
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/plugins` 的职责见上文分类。

#### `postCommitAttribution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/postCommitAttribution.ts`
- 行数：23
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `dangerousCmdlets.ts`

- 所在目录：`src/utils/powershell`
- 完整路径：`src/utils/powershell/dangerousCmdlets.ts`
- 行数：185
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/powershell` 的职责见上文分类。

#### `parser.ts`

- 所在目录：`src/utils/powershell`
- 完整路径：`src/utils/powershell/parser.ts`
- 行数：1804
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1804 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/powershell` 的职责见上文分类。

#### `staticPrefix.ts`

- 所在目录：`src/utils/powershell`
- 完整路径：`src/utils/powershell/staticPrefix.ts`
- 行数：316
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/powershell` 的职责见上文分类。

#### `preflightChecks.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/preflightChecks.test.ts`
- 行数：271
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/preflightChecks.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `preflightChecks.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/preflightChecks.tsx`
- 行数：154
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils` 的职责见上文分类。

#### `printFlag.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/printFlag.test.ts`
- 行数：217
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/printFlag.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `printFlag.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/printFlag.ts`
- 行数：278
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `privacyLevel.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/privacyLevel.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `process.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/process.ts`
- 行数：68
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `processBashCommand.test.tsx`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processBashCommand.test.tsx`
- 行数：117
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/utils/processUserInput/processBashCommand.test.tsx`。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processBashCommand.tsx`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processBashCommand.tsx`
- 行数：144
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processSlashCommand.goal.test.tsx`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processSlashCommand.goal.test.tsx`
- 行数：50
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/utils/processUserInput/processSlashCommand.goal.test.tsx`。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processSlashCommand.test.ts`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processSlashCommand.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/processUserInput/processSlashCommand.test.ts`。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processSlashCommand.tsx`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processSlashCommand.tsx`
- 行数：953
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 953 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processTextPrompt.ts`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processTextPrompt.ts`
- 行数：100
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `processUserInput.ts`

- 所在目录：`src/utils/processUserInput`
- 完整路径：`src/utils/processUserInput/processUserInput.ts`
- 行数：611
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/processUserInput` 的职责见上文分类。

#### `profilerBase.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/profilerBase.ts`
- 行数：87
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `profilerRetention.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/profilerRetention.test.ts`
- 行数：388
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/profilerRetention.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `projectInstructions.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/projectInstructions.test.ts`
- 行数：105
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/projectInstructions.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `projectInstructions.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/projectInstructions.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `promptCategory.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/promptCategory.ts`
- 行数：49
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。所在目录 `src/utils` 的职责见上文分类。

#### `promptEditor.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/promptEditor.test.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/utils/promptEditor.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `promptEditor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/promptEditor.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `promptShellExecution.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/promptShellExecution.test.ts`
- 行数：320
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。运行方式：`bun test ./src/utils/promptShellExecution.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `promptShellExecution.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/promptShellExecution.ts`
- 行数：275
- 主要功能简介：共享工具函数。
- 说明：主要存放给模型看的工具/命令说明书文本，改动会直接影响模型如何使用该能力。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `protectedNamespace.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/protectedNamespace.ts`
- 行数：4
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `providerAutoDetect.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerAutoDetect.test.ts`
- 行数：399
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerAutoDetect.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerAutoDetect.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerAutoDetect.ts`
- 行数：399
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerCustomHeaders.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerCustomHeaders.test.ts`
- 行数：58
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerCustomHeaders.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `providerCustomHeaders.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerCustomHeaders.ts`
- 行数：118
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `providerDiscovery.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerDiscovery.test.ts`
- 行数：421
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerDiscovery.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerDiscovery.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerDiscovery.ts`
- 行数：543
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerFallback.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerFallback.test.ts`
- 行数：149
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerFallback.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `providerFallback.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerFallback.ts`
- 行数：108
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `providerFlag.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerFlag.test.ts`
- 行数：1850
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerFlag.test.ts`。体量较大（约 1850 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerFlag.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerFlag.ts`
- 行数：940
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 940 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerModels.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerModels.test.ts`
- 行数：146
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerModels.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `providerModels.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerModels.ts`
- 行数：35
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `providerProfile.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerProfile.test.ts`
- 行数：3475
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerProfile.test.ts`。体量较大（约 3475 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerProfile.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerProfile.ts`
- 行数：2721
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2721 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerProfiles.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerProfiles.test.ts`
- 行数：5469
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerProfiles.test.ts`。体量较大（约 5469 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerProfiles.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerProfiles.ts`
- 行数：2275
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 2275 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerRecommendation.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerRecommendation.test.ts`
- 行数：194
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerRecommendation.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `providerRecommendation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerRecommendation.ts`
- 行数：317
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerSecrets.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerSecrets.test.ts`
- 行数：336
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerSecrets.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerSecrets.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerSecrets.ts`
- 行数：388
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerStartupOverrides.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerStartupOverrides.test.ts`
- 行数：62
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerStartupOverrides.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `providerStartupOverrides.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerStartupOverrides.ts`
- 行数：103
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `providerValidation.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerValidation.test.ts`
- 行数：961
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/providerValidation.test.ts`。体量较大（约 961 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `providerValidation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/providerValidation.ts`
- 行数：745
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `proxy.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/proxy.test.ts`
- 行数：92
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/proxy.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `proxy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/proxy.ts`
- 行数：458
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `queryContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryContext.ts`
- 行数：179
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `queryEventDriver.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryEventDriver.test.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/queryEventDriver.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `queryEventDriver.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryEventDriver.ts`
- 行数：30
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `queryGuardConfig.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryGuardConfig.test.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/queryGuardConfig.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `queryGuardConfig.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryGuardConfig.ts`
- 行数：67
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `queryHelpers.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryHelpers.ts`
- 行数：560
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `queryLifecycle.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryLifecycle.test.ts`
- 行数：132
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/queryLifecycle.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `queryLifecycle.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryLifecycle.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `queryProfiler.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queryProfiler.ts`
- 行数：325
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `queueProcessor.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/queueProcessor.ts`
- 行数：95
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `readEditContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/readEditContext.ts`
- 行数：227
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `readFileInRange.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/readFileInRange.test.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/readFileInRange.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `readFileInRange.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/readFileInRange.ts`
- 行数：399
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `redaction.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/redaction.ts`
- 行数：965
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 965 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `releaseNotes.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/releaseNotes.test.ts`
- 行数：121
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/releaseNotes.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `releaseNotes.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/releaseNotes.ts`
- 行数：610
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `relevancePruning.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/relevancePruning.test.ts`
- 行数：193
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/relevancePruning.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `relevancePruning.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/relevancePruning.ts`
- 行数：274
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `renderOptions.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/renderOptions.ts`
- 行数：77
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `replInterruption.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replInterruption.test.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/replInterruption.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `replInterruption.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replInterruption.ts`
- 行数：39
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `replMaxTurns.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replMaxTurns.ts`
- 行数：131
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `replayFormat.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayFormat.test.ts`
- 行数：17
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/replayFormat.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `replayFormat.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayFormat.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `replayIndex.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayIndex.test.ts`
- 行数：194
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/replayIndex.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `replayIndex.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayIndex.ts`
- 行数：191
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `replayIndexBuilder.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayIndexBuilder.test.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/replayIndexBuilder.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `replayIndexBuilder.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/replayIndexBuilder.ts`
- 行数：248
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `reportTask.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/reportTask.test.ts`
- 行数：2191
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/reportTask.test.ts`。体量较大（约 2191 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `requestImageValidation.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestImageValidation.test.ts`
- 行数：184
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/requestImageValidation.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `requestImageValidation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestImageValidation.ts`
- 行数：117
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `requestLogging.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestLogging.test.ts`
- 行数：87
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/requestLogging.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `requestLogging.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestLogging.ts`
- 行数：89
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `requestSizeBreakdown.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestSizeBreakdown.test.ts`
- 行数：492
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/requestSizeBreakdown.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `requestSizeBreakdown.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/requestSizeBreakdown.ts`
- 行数：388
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `ripgrep.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ripgrep.test.ts`
- 行数：115
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/ripgrep.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `ripgrep.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/ripgrep.ts`
- 行数：779
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sandbox-adapter.test.ts`

- 所在目录：`src/utils/sandbox`
- 完整路径：`src/utils/sandbox/sandbox-adapter.test.ts`
- 行数：107
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sandbox/sandbox-adapter.test.ts`。所在目录 `src/utils/sandbox` 的职责见上文分类。

#### `sandbox-adapter.ts`

- 所在目录：`src/utils/sandbox`
- 完整路径：`src/utils/sandbox/sandbox-adapter.ts`
- 行数：1015
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1015 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/sandbox` 的职责见上文分类。

#### `sandbox-ui-utils.ts`

- 所在目录：`src/utils/sandbox`
- 完整路径：`src/utils/sandbox/sandbox-ui-utils.ts`
- 行数：12
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/sandbox` 的职责见上文分类。

#### `sanitization.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sanitization.ts`
- 行数：91
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `schemaSanitizer.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/schemaSanitizer.test.ts`
- 行数：68
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/schemaSanitizer.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `schemaSanitizer.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/schemaSanitizer.ts`
- 行数：258
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `screenshotClipboard.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/screenshotClipboard.ts`
- 行数：121
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sdkEventQueue.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sdkEventQueue.ts`
- 行数：134
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `fallbackStorage.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/fallbackStorage.ts`
- 行数：70
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `index.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/index.ts`
- 行数：93
- 主要功能简介：共享工具函数。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `keychainPrefetch.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/keychainPrefetch.ts`
- 行数：116
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `linuxSecretStorage.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/linuxSecretStorage.ts`
- 行数：102
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `macOsKeychainHelpers.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/macOsKeychainHelpers.ts`
- 行数：131
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `macOsKeychainStorage.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/macOsKeychainStorage.ts`
- 行数：231
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `plainTextStorage.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/plainTextStorage.ts`
- 行数：84
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `platformStorage.test.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/platformStorage.test.ts`
- 行数：463
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/secureStorage/platformStorage.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `windowsCredentialStorage.ts`

- 所在目录：`src/utils/secureStorage`
- 完整路径：`src/utils/secureStorage/windowsCredentialStorage.ts`
- 行数：287
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/secureStorage` 的职责见上文分类。

#### `semanticBoolean.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/semanticBoolean.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `semanticNumber.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/semanticNumber.ts`
- 行数：36
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `semver.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/semver.ts`
- 行数：59
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sentry.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sentry.test.ts`
- 行数：87
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sentry.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sentry.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sentry.ts`
- 行数：72
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sequential.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sequential.ts`
- 行数：56
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `serializationStability.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/serializationStability.test.ts`
- 行数：142
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/serializationStability.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sessionActivity.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionActivity.ts`
- 行数：133
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sessionEnvVars.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionEnvVars.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `sessionEnvironment.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionEnvironment.ts`
- 行数：166
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sessionFileAccessHooks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionFileAccessHooks.ts`
- 行数：250
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionIngressAuth.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionIngressAuth.ts`
- 行数：140
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sessionPersistence.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionPersistence.test.ts`
- 行数：94
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionPersistence.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sessionPersistence.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionPersistence.ts`
- 行数：173
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sessionPersistencePolicy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionPersistencePolicy.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `sessionRestore.goal.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionRestore.goal.test.ts`
- 行数：151
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionRestore.goal.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sessionRestore.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionRestore.test.ts`
- 行数：424
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionRestore.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionRestore.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionRestore.ts`
- 行数：585
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionStart.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStart.ts`
- 行数：232
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionState.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionState.ts`
- 行数：150
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sessionStorage.atomicReplace.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStorage.atomicReplace.test.ts`
- 行数：807
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionStorage.atomicReplace.test.ts`。体量较大（约 807 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionStorage.liteTag.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStorage.liteTag.test.ts`
- 行数：66
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionStorage.liteTag.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sessionStorage.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStorage.test.ts`
- 行数：901
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionStorage.test.ts`。体量较大（约 901 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionStorage.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStorage.ts`
- 行数：6196
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 6196 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionStoragePortable.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionStoragePortable.ts`
- 行数：789
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionTitle.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionTitle.test.ts`
- 行数：502
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sessionTitle.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionTitle.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionTitle.ts`
- 行数：465
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sessionUrl.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sessionUrl.ts`
- 行数：64
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `set.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/set.ts`
- 行数：53
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `agentModelsSchema.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/agentModelsSchema.test.ts`
- 行数：35
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/agentModelsSchema.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `allErrors.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/allErrors.ts`
- 行数：32
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `allowBypassPermissionsMode.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/allowBypassPermissionsMode.test.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/allowBypassPermissionsMode.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `applySettingsChange.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/applySettingsChange.ts`
- 行数：92
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `changeDetector.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/changeDetector.test.ts`
- 行数：299
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/changeDetector.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `changeDetector.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/changeDetector.ts`
- 行数：584
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/constants.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `flagSettings.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/flagSettings.test.ts`
- 行数：96
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/flagSettings.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `flagSettings.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/flagSettings.ts`
- 行数：112
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `internalWrites.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/internalWrites.ts`
- 行数：37
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `managedPath.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/managedPath.ts`
- 行数：34
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/utils/settings/mdm`
- 完整路径：`src/utils/settings/mdm/constants.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。所在目录 `src/utils/settings/mdm` 的职责见上文分类。

#### `rawRead.ts`

- 所在目录：`src/utils/settings/mdm`
- 完整路径：`src/utils/settings/mdm/rawRead.ts`
- 行数：130
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings/mdm` 的职责见上文分类。

#### `settings.ts`

- 所在目录：`src/utils/settings/mdm`
- 完整路径：`src/utils/settings/mdm/settings.ts`
- 行数：338
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings/mdm` 的职责见上文分类。

#### `modelPricing.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/modelPricing.test.ts`
- 行数：185
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/modelPricing.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `modelPricing.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/modelPricing.ts`
- 行数：64
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `modelPricingSchema.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/modelPricingSchema.test.ts`
- 行数：140
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/modelPricingSchema.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `permissionValidation.protoName.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/permissionValidation.protoName.test.ts`
- 行数：80
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/permissionValidation.protoName.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `permissionValidation.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/permissionValidation.ts`
- 行数：262
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `pluginOnlyPolicy.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/pluginOnlyPolicy.ts`
- 行数：60
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `providerProfileModelPickerMode.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/providerProfileModelPickerMode.test.ts`
- 行数：26
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/providerProfileModelPickerMode.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `schemaOutput.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/schemaOutput.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/settings` 的职责见上文分类。

#### `settings.transaction.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/settings.transaction.test.ts`
- 行数：1279
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/settings.transaction.test.ts`。体量较大（约 1279 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `settings.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/settings.ts`
- 行数：1111
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1111 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `settingsCache.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/settingsCache.ts`
- 行数：80
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `settingsFileTransaction.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/settingsFileTransaction.ts`
- 行数：516
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `settingsMergeCustomizer.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/settingsMergeCustomizer.test.ts`
- 行数：119
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/settingsMergeCustomizer.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `toolValidationConfig.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/toolValidationConfig.ts`
- 行数：112
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/types.ts`
- 行数：1474
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1474 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `validateEditTool.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/validateEditTool.ts`
- 行数：45
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `validation.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/validation.ts`
- 行数：288
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/settings` 的职责见上文分类。

#### `validationTips.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/validationTips.ts`
- 行数：154
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/settings` 的职责见上文分类。

#### `worktreeSettings.test.ts`

- 所在目录：`src/utils/settings`
- 完整路径：`src/utils/settings/worktreeSettings.test.ts`
- 行数：44
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/settings/worktreeSettings.test.ts`。所在目录 `src/utils/settings` 的职责见上文分类。

#### `setupScreenGates.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/setupScreenGates.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/setupScreenGates.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `setupScreenGates.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/setupScreenGates.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `bashProvider.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/bashProvider.ts`
- 行数：256
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/shell` 的职责见上文分类。

#### `outputLimits.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/outputLimits.ts`
- 行数：14
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/shell` 的职责见上文分类。

#### `powershellDetection.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/powershellDetection.ts`
- 行数：107
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/shell` 的职责见上文分类。

#### `powershellProvider.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/powershellProvider.ts`
- 行数：124
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/shell` 的职责见上文分类。

#### `prefix.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/prefix.ts`
- 行数：367
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/shell` 的职责见上文分类。

#### `readOnlyCommandValidation.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/readOnlyCommandValidation.ts`
- 行数：1893
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1893 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/shell` 的职责见上文分类。

#### `resolveDefaultShell.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/resolveDefaultShell.ts`
- 行数：14
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/shell` 的职责见上文分类。

#### `shellProvider.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/shellProvider.ts`
- 行数：33
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/shell` 的职责见上文分类。

#### `shellToolUtils.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/shellToolUtils.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/shell` 的职责见上文分类。

#### `specPrefix.ts`

- 所在目录：`src/utils/shell`
- 完整路径：`src/utils/shell/specPrefix.ts`
- 行数：241
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/shell` 的职责见上文分类。

#### `shellConfig.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/shellConfig.ts`
- 行数：167
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sideQuery.attribution.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sideQuery.attribution.test.ts`
- 行数：372
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sideQuery.attribution.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sideQuery.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sideQuery.ts`
- 行数：235
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sideQuestion.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sideQuestion.ts`
- 行数：155
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `signal.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/signal.ts`
- 行数：43
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sinks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sinks.ts`
- 行数：16
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `skillChangeDetector.test.ts`

- 所在目录：`src/utils/skills`
- 完整路径：`src/utils/skills/skillChangeDetector.test.ts`
- 行数：373
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/skills/skillChangeDetector.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/skills` 的职责见上文分类。

#### `skillChangeDetector.ts`

- 所在目录：`src/utils/skills`
- 完整路径：`src/utils/skills/skillChangeDetector.ts`
- 行数：375
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/skills` 的职责见上文分类。

#### `slashCommandParsing.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/slashCommandParsing.ts`
- 行数：60
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sleep.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sleep.ts`
- 行数：84
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `sliceAnsi.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sliceAnsi.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `slowOperations.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/slowOperations.ts`
- 行数：286
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `sshPreParse.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sshPreParse.test.ts`
- 行数：193
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/sshPreParse.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `sshPreParse.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/sshPreParse.ts`
- 行数：174
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `stableStringify.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stableStringify.test.ts`
- 行数：241
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/stableStringify.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `stableStringify.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stableStringify.ts`
- 行数：214
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `standaloneAgent.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/standaloneAgent.ts`
- 行数：23
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `startupProfiler.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/startupProfiler.ts`
- 行数：218
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `staticRender.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/staticRender.tsx`
- 行数：119
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。所在目录 `src/utils` 的职责见上文分类。

#### `stats.totalDays.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stats.totalDays.test.ts`
- 行数：337
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/stats.totalDays.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `stats.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stats.ts`
- 行数：1104
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1104 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `statsCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statsCache.ts`
- 行数：546
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `status.routes.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/status.routes.test.ts`
- 行数：425
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/status.routes.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `status.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/status.test.ts`
- 行数：380
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/status.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `status.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/status.tsx`
- 行数：814
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 814 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `statusNoticeDefinitions.safety.test.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statusNoticeDefinitions.safety.test.tsx`
- 行数：267
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。运行方式：`bun test ./src/utils/statusNoticeDefinitions.safety.test.tsx`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `statusNoticeDefinitions.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statusNoticeDefinitions.tsx`
- 行数：351
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `statusNoticeHelpers.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statusNoticeHelpers.ts`
- 行数：20
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `statusNoticeLocalModel.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statusNoticeLocalModel.ts`
- 行数：222
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `statusRedaction.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/statusRedaction.test.ts`
- 行数：189
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/statusRedaction.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `stream.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stream.ts`
- 行数：76
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `streamJsonStdoutGuard.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/streamJsonStdoutGuard.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `streamingOptimizer.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/streamingOptimizer.test.ts`
- 行数：61
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/streamingOptimizer.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `streamingOptimizer.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/streamingOptimizer.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `streamlinedTransform.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/streamlinedTransform.ts`
- 行数：202
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `stringUtils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/stringUtils.ts`
- 行数：235
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `subprocessEnv.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/subprocessEnv.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `commandSuggestions.test.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/commandSuggestions.test.ts`
- 行数：1082
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/suggestions/commandSuggestions.test.ts`。体量较大（约 1082 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `commandSuggestions.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/commandSuggestions.ts`
- 行数：857
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 857 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `directoryCompletion.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/directoryCompletion.ts`
- 行数：263
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `shellHistoryCompletion.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/shellHistoryCompletion.ts`
- 行数：119
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `skillUsageTracking.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/skillUsageTracking.ts`
- 行数：55
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `slackChannelSuggestions.ts`

- 所在目录：`src/utils/suggestions`
- 完整路径：`src/utils/suggestions/slackChannelSuggestions.ts`
- 行数：209
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/suggestions` 的职责见上文分类。

#### `It2SetupPrompt.tsx`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/It2SetupPrompt.tsx`
- 行数：379
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `ITermBackend.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/ITermBackend.ts`
- 行数：370
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `InProcessBackend.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/InProcessBackend.ts`
- 行数：341
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `PaneBackendExecutor.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/PaneBackendExecutor.ts`
- 行数：354
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `TmuxBackend.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/TmuxBackend.ts`
- 行数：764
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `detection.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/detection.ts`
- 行数：128
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `it2Setup.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/it2Setup.ts`
- 行数：245
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `registry.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/registry.ts`
- 行数：464
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `teammateModeSnapshot.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/teammateModeSnapshot.ts`
- 行数：87
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/utils/swarm/backends`
- 完整路径：`src/utils/swarm/backends/types.ts`
- 行数：313
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm/backends` 的职责见上文分类。

#### `constants.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/constants.ts`
- 行数：33
- 主要功能简介：共享工具函数。
- 说明：抽出名称常量以打断循环依赖，例如工具对外名。短文件，多为常量、再导出或薄包装。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `inProcessPermissionAbort.test.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/inProcessPermissionAbort.test.ts`
- 行数：138
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/swarm/inProcessPermissionAbort.test.ts`。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `inProcessPermissionAbort.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/inProcessPermissionAbort.ts`
- 行数：61
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `inProcessRunner.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/inProcessRunner.ts`
- 行数：1707
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1707 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `leaderPermissionBridge.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/leaderPermissionBridge.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `permissionSync.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/permissionSync.ts`
- 行数：955
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 955 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `reconnection.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/reconnection.ts`
- 行数：119
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `spawnInProcess.interruptionTrace.test.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/spawnInProcess.interruptionTrace.test.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/swarm/spawnInProcess.interruptionTrace.test.ts`。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `spawnInProcess.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/spawnInProcess.ts`
- 行数：356
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `spawnUtils.test.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/spawnUtils.test.ts`
- 行数：104
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/swarm/spawnUtils.test.ts`。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `spawnUtils.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/spawnUtils.ts`
- 行数：180
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teamHelpers.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teamHelpers.ts`
- 行数：683
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teammateInit.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teammateInit.ts`
- 行数：129
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teammateLayoutManager.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teammateLayoutManager.ts`
- 行数：107
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teammateModel.test.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teammateModel.test.ts`
- 行数：57
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/swarm/teammateModel.test.ts`。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teammateModel.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teammateModel.ts`
- 行数：10
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `teammatePromptAddendum.ts`

- 所在目录：`src/utils/swarm`
- 完整路径：`src/utils/swarm/teammatePromptAddendum.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/swarm` 的职责见上文分类。

#### `systemDirectories.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/systemDirectories.ts`
- 行数：74
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `systemPrompt.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/systemPrompt.ts`
- 行数：123
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `systemPromptType.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/systemPromptType.ts`
- 行数：14
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `systemTheme.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/systemTheme.ts`
- 行数：119
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `systemThemeWatcher.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/systemThemeWatcher.ts`
- 行数：25
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `TaskOutput.ts`

- 所在目录：`src/utils/task`
- 完整路径：`src/utils/task/TaskOutput.ts`
- 行数：390
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/task` 的职责见上文分类。

#### `diskOutput.ts`

- 所在目录：`src/utils/task`
- 完整路径：`src/utils/task/diskOutput.ts`
- 行数：451
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/task` 的职责见上文分类。

#### `framework.ts`

- 所在目录：`src/utils/task`
- 完整路径：`src/utils/task/framework.ts`
- 行数：308
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/task` 的职责见上文分类。

#### `outputFormatting.ts`

- 所在目录：`src/utils/task`
- 完整路径：`src/utils/task/outputFormatting.ts`
- 行数：38
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/task` 的职责见上文分类。

#### `sdkProgress.ts`

- 所在目录：`src/utils/task`
- 完整路径：`src/utils/task/sdkProgress.ts`
- 行数：36
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/task` 的职责见上文分类。

#### `taskNotificationIdentity.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/taskNotificationIdentity.test.ts`
- 行数：134
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/taskNotificationIdentity.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `taskNotificationIdentity.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/taskNotificationIdentity.ts`
- 行数：114
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `taskReport.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/taskReport.ts`
- 行数：1593
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1593 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `taskSummary.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/taskSummary.ts`
- 行数：34
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `tasks.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tasks.ts`
- 行数：862
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 862 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `teamDiscovery.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teamDiscovery.ts`
- 行数：81
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `teamMemoryOps.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teamMemoryOps.ts`
- 行数：88
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `teammate.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teammate.ts`
- 行数：292
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `teammateContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teammateContext.ts`
- 行数：96
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `teammateMailbox.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teammateMailbox.ts`
- 行数：1183
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1183 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `events.ts`

- 所在目录：`src/utils/telemetry`
- 完整路径：`src/utils/telemetry/events.ts`
- 行数：16
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/telemetry` 的职责见上文分类。

#### `perfettoTracing.ts`

- 所在目录：`src/utils/telemetry`
- 完整路径：`src/utils/telemetry/perfettoTracing.ts`
- 行数：91
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/telemetry` 的职责见上文分类。

#### `pluginTelemetry.ts`

- 所在目录：`src/utils/telemetry`
- 完整路径：`src/utils/telemetry/pluginTelemetry.ts`
- 行数：289
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/telemetry` 的职责见上文分类。

#### `sessionTracing.ts`

- 所在目录：`src/utils/telemetry`
- 完整路径：`src/utils/telemetry/sessionTracing.ts`
- 行数：112
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/telemetry` 的职责见上文分类。

#### `skillLoadedEvent.ts`

- 所在目录：`src/utils/telemetry`
- 完整路径：`src/utils/telemetry/skillLoadedEvent.ts`
- 行数：39
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/telemetry` 的职责见上文分类。

#### `teleport.tsx`

- 所在目录：`src/utils`
- 完整路径：`src/utils/teleport.tsx`
- 行数：1225
- 主要功能简介：共享工具函数。
- 说明：包含 Ink/React 界面，变更需注意终端宽度与主题色。体量较大（约 1225 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `api.ts`

- 所在目录：`src/utils/teleport`
- 完整路径：`src/utils/teleport/api.ts`
- 行数：466
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/teleport` 的职责见上文分类。

#### `environmentSelection.ts`

- 所在目录：`src/utils/teleport`
- 完整路径：`src/utils/teleport/environmentSelection.ts`
- 行数：77
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/teleport` 的职责见上文分类。

#### `environments.ts`

- 所在目录：`src/utils/teleport`
- 完整路径：`src/utils/teleport/environments.ts`
- 行数：120
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/teleport` 的职责见上文分类。

#### `gitBundle.ts`

- 所在目录：`src/utils/teleport`
- 完整路径：`src/utils/teleport/gitBundle.ts`
- 行数：292
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/teleport` 的职责见上文分类。

#### `tempfile.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tempfile.ts`
- 行数：31
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `terminal.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/terminal.ts`
- 行数：131
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `terminalAnsi.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/terminalAnsi.ts`
- 行数：8
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `terminalPanel.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/terminalPanel.ts`
- 行数：191
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `textHighlighting.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/textHighlighting.ts`
- 行数：169
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `theme.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/theme.ts`
- 行数：671
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `thinking.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/thinking.test.ts`
- 行数：151
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/thinking.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `thinking.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/thinking.ts`
- 行数：219
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `thinkingTokens.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/thinkingTokens.test.ts`
- 行数：69
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/thinkingTokens.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `timeouts.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/timeouts.test.ts`
- 行数：51
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/timeouts.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `timeouts.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/timeouts.ts`
- 行数：59
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `tmuxSocket.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tmuxSocket.ts`
- 行数：427
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/utils/todo`
- 完整路径：`src/utils/todo/types.ts`
- 行数：18
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils/todo` 的职责见上文分类。

#### `tokenBudget.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tokenBudget.ts`
- 行数：73
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `tokens.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tokens.test.ts`
- 行数：248
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/tokens.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `tokens.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/tokens.ts`
- 行数：590
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `toolErrors.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolErrors.test.ts`
- 行数：152
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/toolErrors.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `toolErrors.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolErrors.ts`
- 行数：138
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `toolPool.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolPool.ts`
- 行数：79
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `toolResultStorage.preview.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolResultStorage.preview.test.ts`
- 行数：491
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/toolResultStorage.preview.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `toolResultStorage.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolResultStorage.test.ts`
- 行数：120
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/toolResultStorage.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `toolResultStorage.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolResultStorage.ts`
- 行数：1368
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1368 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `toolSchemaCache.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolSchemaCache.ts`
- 行数：44
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `toolSearch.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolSearch.test.ts`
- 行数：85
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/toolSearch.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `toolSearch.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/toolSearch.ts`
- 行数：803
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 803 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `transcriptFileLock.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/transcriptFileLock.ts`
- 行数：200
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `transcriptSearch.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/transcriptSearch.ts`
- 行数：202
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `treeify.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/treeify.ts`
- 行数：170
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `truncate.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/truncate.test.ts`
- 行数：26
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/truncate.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `truncate.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/truncate.ts`
- 行数：186
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `udsClient.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/udsClient.ts`
- 行数：36
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `udsMessaging.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/udsMessaging.ts`
- 行数：47
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `ccrSession.ts`

- 所在目录：`src/utils/ultraplan`
- 完整路径：`src/utils/ultraplan/ccrSession.ts`
- 行数：353
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils/ultraplan` 的职责见上文分类。

#### `keyword.ts`

- 所在目录：`src/utils/ultraplan`
- 完整路径：`src/utils/ultraplan/keyword.ts`
- 行数：127
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils/ultraplan` 的职责见上文分类。

#### `unaryLogging.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/unaryLogging.ts`
- 行数：39
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `undercover.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/undercover.ts`
- 行数：11
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `updateStrategy.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/updateStrategy.test.ts`
- 行数：204
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/updateStrategy.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `updateStrategy.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/updateStrategy.ts`
- 行数：138
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `urlRedaction.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/urlRedaction.test.ts`
- 行数：276
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/urlRedaction.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `user.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/user.test.ts`
- 行数：158
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/user.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `user.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/user.ts`
- 行数：176
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `userAgent.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/userAgent.ts`
- 行数：20
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `userPromptKeywords.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/userPromptKeywords.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `uuid.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/uuid.ts`
- 行数：27
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `validation.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/validation.ts`
- 行数：54
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `version.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/version.ts`
- 行数：60
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `visionUtils.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/visionUtils.test.ts`
- 行数：297
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/visionUtils.test.ts`。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `visionUtils.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/visionUtils.ts`
- 行数：213
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `warningHandler.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/warningHandler.test.ts`
- 行数：194
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/warningHandler.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `warningHandler.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/warningHandler.ts`
- 行数：150
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `which.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/which.ts`
- 行数：82
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `windowsPaths.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/windowsPaths.ts`
- 行数：173
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `withResolvers.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/withResolvers.ts`
- 行数：22
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `words.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/words.ts`
- 行数：797
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `workloadContext.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/workloadContext.ts`
- 行数：57
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `worktree.agentBase.fixture.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.agentBase.fixture.ts`
- 行数：29
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `worktree.agentBase.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.agentBase.test.ts`
- 行数：107
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/worktree.agentBase.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `worktree.git.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.git.test.ts`
- 行数：182
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/worktree.git.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `worktree.multiRepoParent.fixture.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.multiRepoParent.fixture.ts`
- 行数：40
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `worktree.multiRepoParent.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.multiRepoParent.test.ts`
- 行数：132
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/worktree.multiRepoParent.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `worktree.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.test.ts`
- 行数：133
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/worktree.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `worktree.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktree.ts`
- 行数：1704
- 主要功能简介：共享工具函数。
- 说明：体量较大（约 1704 行），阅读时建议先看导出符号再看内部辅助函数。属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `worktreeModeEnabled.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/worktreeModeEnabled.ts`
- 行数：11
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `xaiCredentials.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/xaiCredentials.test.ts`
- 行数：99
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/xaiCredentials.test.ts`。所在目录 `src/utils` 的职责见上文分类。

#### `xaiCredentials.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/xaiCredentials.ts`
- 行数：341
- 主要功能简介：共享工具函数。
- 说明：属于共享工具层，改动可能影响大量调用方，需要更完整的回归。所在目录 `src/utils` 的职责见上文分类。

#### `xdg.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/xdg.ts`
- 行数：65
- 主要功能简介：共享工具函数。
- 说明：所在目录 `src/utils` 的职责见上文分类。

#### `xml.test.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/xml.test.ts`
- 行数：33
- 主要功能简介：共享工具函数。
- 说明：运行方式：`bun test ./src/utils/xml.test.ts`。短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `xml.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/xml.ts`
- 行数：34
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `yaml.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/yaml.ts`
- 行数：15
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

#### `zodToJsonSchema.ts`

- 所在目录：`src/utils`
- 完整路径：`src/utils/zodToJsonSchema.ts`
- 行数：23
- 主要功能简介：共享工具函数。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `src/utils` 的职责见上文分类。

### 2.122 目录组 `src/vim`（5 个文件，约 1513 行）

该组位于仓库相对路径 `src/vim`。Vim 运动与操作符。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `motions.ts`

- 所在目录：`src/vim`
- 完整路径：`src/vim/motions.ts`
- 行数：82
- 主要功能简介：Vim 运动与操作符。
- 说明：所在目录 `src/vim` 的职责见上文分类。

#### `operators.ts`

- 所在目录：`src/vim`
- 完整路径：`src/vim/operators.ts`
- 行数：556
- 主要功能简介：Vim 运动与操作符。
- 说明：所在目录 `src/vim` 的职责见上文分类。

#### `textObjects.ts`

- 所在目录：`src/vim`
- 完整路径：`src/vim/textObjects.ts`
- 行数：186
- 主要功能简介：Vim 运动与操作符。
- 说明：所在目录 `src/vim` 的职责见上文分类。

#### `transitions.ts`

- 所在目录：`src/vim`
- 完整路径：`src/vim/transitions.ts`
- 行数：490
- 主要功能简介：Vim 运动与操作符。
- 说明：所在目录 `src/vim` 的职责见上文分类。

#### `types.ts`

- 所在目录：`src/vim`
- 完整路径：`src/vim/types.ts`
- 行数：199
- 主要功能简介：Vim 运动与操作符。
- 说明：所在目录 `src/vim` 的职责见上文分类。

### 2.123 目录组 `src/voice`（1 个文件，约 54 行）

该组位于仓库相对路径 `src/voice`。语音模式开关。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `voiceModeEnabled.ts`

- 所在目录：`src/voice`
- 完整路径：`src/voice/voiceModeEnabled.ts`
- 行数：54
- 主要功能简介：语音模式开关。
- 说明：所在目录 `src/voice` 的职责见上文分类。

### 2.124 目录组 `tests/build`（1 个文件，约 214 行）

该组位于仓库相对路径 `tests/build`。仓库级测试。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `scanner-filedir.test.ts`

- 所在目录：`tests/build`
- 完整路径：`tests/build/scanner-filedir.test.ts`
- 行数：214
- 主要功能简介：仓库级测试。
- 说明：运行方式：`bun test ./tests/build/scanner-filedir.test.ts`。所在目录 `tests/build` 的职责见上文分类。

### 2.125 目录组 `tests/sdk`（21 个文件，约 7353 行）

该组位于仓库相对路径 `tests/sdk`。SDK 包契约测试。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `casing.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/casing.test.ts`
- 行数：92
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/casing.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `engine-mutators.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/engine-mutators.test.ts`
- 行数：204
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/engine-mutators.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `generated-types.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/generated-types.test.ts`
- 行数：436
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/generated-types.test.ts`。生成文件，应修改生成器或描述符后运行 integrations:generate。所在目录 `tests/sdk` 的职责见上文分类。

#### `mock-engine.ts`

- 所在目录：`tests/sdk/helpers`
- 完整路径：`tests/sdk/helpers/mock-engine.ts`
- 行数：84
- 主要功能简介：SDK 包契约测试。
- 说明：所在目录 `tests/sdk/helpers` 的职责见上文分类。

#### `query-test-doubles.ts`

- 所在目录：`tests/sdk/helpers`
- 完整路径：`tests/sdk/helpers/query-test-doubles.ts`
- 行数：198
- 主要功能简介：SDK 包契约测试。
- 说明：所在目录 `tests/sdk/helpers` 的职责见上文分类。

#### `mcp-cleanup.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/mcp-cleanup.test.ts`
- 行数：237
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/mcp-cleanup.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `package-consumer-types.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/package-consumer-types.test.ts`
- 行数：327
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/package-consumer-types.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `permissions.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/permissions.test.ts`
- 行数：1400
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/permissions.test.ts`。体量较大（约 1400 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `tests/sdk` 的职责见上文分类。

#### `query-concurrency.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/query-concurrency.test.ts`
- 行数：333
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/query-concurrency.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `query-happy-path.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/query-happy-path.test.ts`
- 行数：395
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/query-happy-path.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `query-lifecycle.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/query-lifecycle.test.ts`
- 行数：730
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/query-lifecycle.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `query-methods.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/query-methods.test.ts`
- 行数：244
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/query-methods.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `sdk-context-isolation.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/sdk-context-isolation.test.ts`
- 行数：451
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/sdk-context-isolation.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `sdk-factories.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/sdk-factories.test.ts`
- 行数：100
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/sdk-factories.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `sdk-mcp-sdk-tools.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/sdk-mcp-sdk-tools.test.ts`
- 行数：126
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/sdk-mcp-sdk-tools.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `sdk-preserved-segment.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/sdk-preserved-segment.test.ts`
- 行数：506
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/sdk-preserved-segment.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `sdk-v2-lifecycle.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/sdk-v2-lifecycle.test.ts`
- 行数：751
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/sdk-v2-lifecycle.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `session-functions.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/session-functions.test.ts`
- 行数：385
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/session-functions.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `shared-utils.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/shared-utils.test.ts`
- 行数：167
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/shared-utils.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `stub-leak-detect.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/stub-leak-detect.test.ts`
- 行数：90
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/stub-leak-detect.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

#### `tool-schema-cache.test.ts`

- 所在目录：`tests/sdk`
- 完整路径：`tests/sdk/tool-schema-cache.test.ts`
- 行数：97
- 主要功能简介：SDK 包契约测试。
- 说明：运行方式：`bun test ./tests/sdk/tool-schema-cache.test.ts`。所在目录 `tests/sdk` 的职责见上文分类。

### 2.126 目录组 `vendor/node-domexception-shim`（1 个文件，约 3 行）

该组位于仓库相对路径 `vendor/node-domexception-shim`。第三方垫片。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `index.js`

- 所在目录：`vendor/node-domexception-shim`
- 完整路径：`vendor/node-domexception-shim/index.js`
- 行数：3
- 主要功能简介：第三方垫片。
- 说明：该目录的装配入口，通常负责导出命令对象或聚合子模块。短文件，多为常量、再导出或薄包装。所在目录 `vendor/node-domexception-shim` 的职责见上文分类。

### 2.127 目录组 `vscode-extension/openclaude-vscode`（16 个文件，约 5932 行）

该组位于仓库相对路径 `vscode-extension/openclaude-vscode`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `lint.js`

- 所在目录：`vscode-extension/openclaude-vscode/scripts`
- 完整路径：`vscode-extension/openclaude-vscode/scripts/lint.js`
- 行数：17
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `vscode-extension/openclaude-vscode/scripts` 的职责见上文分类。

#### `chatProvider.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/chatProvider.js`
- 行数：683
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `chatRenderer.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/chatRenderer.js`
- 行数：1354
- 主要功能简介：VS Code 扩展宿主。
- 说明：体量较大（约 1354 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `diffController.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/diffController.js`
- 行数：90
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `messageParser.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/messageParser.js`
- 行数：177
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `permissionResponse.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/permissionResponse.js`
- 行数：46
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `permissionResponse.test.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/permissionResponse.test.js`
- 行数：42
- 主要功能简介：VS Code 扩展宿主。
- 说明：运行方式：`bun test ./vscode-extension/openclaude-vscode/src/chat/permissionResponse.test.js`。所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `processManager.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/processManager.js`
- 行数：194
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `protocol.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/protocol.js`
- 行数：186
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `sessionManager.js`

- 所在目录：`vscode-extension/openclaude-vscode/src/chat`
- 完整路径：`vscode-extension/openclaude-vscode/src/chat/sessionManager.js`
- 行数：282
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src/chat` 的职责见上文分类。

#### `extension.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/extension.js`
- 行数：1437
- 主要功能简介：VS Code 扩展宿主。
- 说明：体量较大（约 1437 行），阅读时建议先看导出符号再看内部辅助函数。所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

#### `extension.test.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/extension.test.js`
- 行数：273
- 主要功能简介：VS Code 扩展宿主。
- 说明：运行方式：`bun test ./vscode-extension/openclaude-vscode/src/extension.test.js`。所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

#### `presentation.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/presentation.js`
- 行数：202
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

#### `presentation.test.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/presentation.test.js`
- 行数：291
- 主要功能简介：VS Code 扩展宿主。
- 说明：运行方式：`bun test ./vscode-extension/openclaude-vscode/src/presentation.test.js`。所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

#### `state.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/state.js`
- 行数：412
- 主要功能简介：VS Code 扩展宿主。
- 说明：所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

#### `state.test.js`

- 所在目录：`vscode-extension/openclaude-vscode/src`
- 完整路径：`vscode-extension/openclaude-vscode/src/state.test.js`
- 行数：246
- 主要功能简介：VS Code 扩展宿主。
- 说明：运行方式：`bun test ./vscode-extension/openclaude-vscode/src/state.test.js`。所在目录 `vscode-extension/openclaude-vscode/src` 的职责见上文分类。

### 2.128 目录组 `web/astro.config.mjs`（1 个文件，约 11 行）

该组位于仓库相对路径 `web/astro.config.mjs`。文档站点。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `astro.config.mjs`

- 所在目录：`web`
- 完整路径：`web/astro.config.mjs`
- 行数：11
- 主要功能简介：文档站点。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `web` 的职责见上文分类。

### 2.129 目录组 `web/scripts`（2 个文件，约 246 行）

该组位于仓库相对路径 `web/scripts`。构建、医生、集成生成、隐私扫描。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `verify-dist.test.ts`

- 所在目录：`web/scripts`
- 完整路径：`web/scripts/verify-dist.test.ts`
- 行数：148
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：运行方式：`bun test ./web/scripts/verify-dist.test.ts`。所在目录 `web/scripts` 的职责见上文分类。

#### `verify-dist.ts`

- 所在目录：`web/scripts`
- 完整路径：`web/scripts/verify-dist.ts`
- 行数：98
- 主要功能简介：构建、医生、集成生成、隐私扫描。
- 说明：所在目录 `web/scripts` 的职责见上文分类。

### 2.130 目录组 `web/src`（10 个文件，约 1137 行）

该组位于仓库相对路径 `web/src`。文档站点。 阅读本组时，优先打开不含 `.test.` 的入口文件，再对照同名测试理解不变量。 若本组是 generated，请改上游描述符。若本组是 UI，注意与 `web/src/data` 是否需要同步用户文案。

#### `buddy.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/buddy.ts`
- 行数：100
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `cliFlags.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/cliFlags.ts`
- 行数：187
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `commands.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/commands.ts`
- 行数：164
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `configuration.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/configuration.ts`
- 行数：99
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `docsNav.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/docsNav.ts`
- 行数：52
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `keybindings.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/keybindings.ts`
- 行数：26
- 主要功能简介：文档站点。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `web/src/data` 的职责见上文分类。

#### `partners.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/partners.ts`
- 行数：63
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `providers.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/providers.ts`
- 行数：391
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

#### `site.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/site.ts`
- 行数：15
- 主要功能简介：文档站点。
- 说明：短文件，多为常量、再导出或薄包装。所在目录 `web/src/data` 的职责见上文分类。

#### `skills.ts`

- 所在目录：`web/src/data`
- 完整路径：`web/src/data/skills.ts`
- 行数：40
- 主要功能简介：文档站点。
- 说明：所在目录 `web/src/data` 的职责见上文分类。

## 3. 如何在这份清单上继续工作

1. 找功能：按命令名搜 `src/commands/<name>`，按工具名搜 `src/tools/<Name>Tool`。
2. 找类型：`src/types/` 与 `src/Tool.ts`。
3. 找供应商：`src/integrations/`。
4. 找测试：同目录 `*.test.ts` 或 `tests/sdk`。
5. 改用户文案：同步 `web/src/data/*`。
6. 改构建：`scripts/build.ts` 与 CI 工作流。

本清单随仓库增长会过时。重新生成时应使用同样的排除规则，并更新第 1.3 节数字。 设计行为见 `1DESIGN.md`，使用方法见 `2HELP.md`，演进见 `3SOLUTION.md`。
