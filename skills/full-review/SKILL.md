---
name: full-review
description: >
  六层递进式项目代码审计（死代码 → 冗余逻辑 → 依赖健康 → 业务审计 → 安全性 → 全局健康度）。
  支持 Rust / Flutter / Java / JS·TS / Kotlin / C / C++ / Zig / Python 与 Web / CLI / 前端 / Library / FFI 的自由组合。
  当用户发送 /full-review 或提到代码审计、代码检查、死代码清理、安全审计、项目体检时触发。
---

# Code Review Skill

当用户发送 `/full-review` 或表达"审计/检查项目代码"意图时触发。

## 工作流程

### Step 0：上下文推断

在询问用户之前，使用 `Glob` + `Read` 对工作目录执行轻量探测，推断语言和项目类型的默认值。核心原则：**推断不出就不填，宁缺勿错。**

**1. 语言检测 — 基于配置文件存在性（高置信度）**

使用 `Glob` 检查工作目录根部是否存在以下配置文件：

| 配置文件 | 推断语言 |
|----------|----------|
| `Cargo.toml` | Rust |
| `pubspec.yaml` | Flutter / Dart |
| `build.gradle` 或 `build.gradle.kts` | Kotlin 或 Java（需消歧义） |
| `pom.xml` | Java |
| `package.json` | JavaScript / TypeScript |
| `CMakeLists.txt` | C 或 C++（需消歧义） |
| `build.zig` | Zig |
| `pyproject.toml` 或 `setup.py` | Python |

消歧义规则：
- Gradle 项目：`Glob("**/*.kt")` 有结果 → Kotlin，否则 → Java
- CMake 项目：`Glob("**/*.cpp")` 或 `Glob("**/*.cc")` 有结果 → C++，否则 → C

多语言项目：同时检测到多个配置文件时，全部列入默认值。

**2. 项目类型检测 — 基于依赖/结构特征（中置信度，仅在信号明确时推断）**

使用 `Read` 读取步骤 1 中检测到的配置文件，扫描依赖列表中的关键词：

| 信号 | 推断类型 |
|------|----------|
| Rust: `clap` / `structopt` 在 dependencies | CLI |
| Rust: `axum` / `actix-web` / `rocket` / `warp` / `tower-http` | Web 服务 |
| Rust: `pyo3` / `napi` / `wasm-bindgen` / `jni` | FFI |
| Rust: `Cargo.toml` 只有 `[lib]` 无 `[[bin]]` | Library |
| JS/TS: `express` / `fastify` / `koa` / `hono` / `next` | Web 服务 |
| JS/TS: `react` / `vue` / `angular` / `svelte` | 前端 |
| JS/TS: `commander` / `yargs` / `inquirer` | CLI |
| Python: `flask` / `django` / `fastapi` / `starlette` | Web 服务 |
| Python: `click` / `typer` / `argparse`（main entry） | CLI |
| Flutter: `pubspec.yaml` 含 flutter SDK 依赖 | 前端 |
| Java/Kotlin: `spring-boot` / `spring-web` | Web 服务 |
| Java/Kotlin: `picocli` / `jcommander` | CLI |

不推断的情况：依赖列表中无上述关键词，或信号冲突（如同时包含 Web 框架和 CLI 框架），则不设默认值。

**3. 项目描述与需求文档**：不推断，始终留给用户填写。

### Step 1：展示推断结果并确认

根据 Step 0 的推断结果，分两种情况处理：

**有推断结果时** — 先展示推断结果及依据，再让用户确认或调整。未能推断的选项仍正常提问。示例措辞：

```
检测到以下项目特征（基于工作目录配置文件）：
- 语言：Rust（依据：存在 Cargo.toml）
- 项目类型：Web 服务（依据：依赖中包含 axum）

以上如需调整请说明。另外请补充：
- 项目描述（可选）：简要描述项目核心业务/功能
- 需求文档路径（可选）：如有 PRD / 功能清单，提供后启用「业务-代码对齐度」审计

如无调整，回复「确认」即可继续。
```

**无推断结果时** — 退回完整提问模式，使用 `AskUserQuestion` 工具一次性问全部选项：

**问题 1 — 语言（多选）**：
- Rust
- Flutter / Dart
- Java
- JavaScript / TypeScript
- Kotlin
- C
- C++
- Zig
- Python

**问题 2 — 项目类型（多选）**：
- Web 服务（后端 API / 全栈）
- CLI（命令行工具）
- 前端（SPA / SSR / 移动端 UI）
- Library（可复用库 / SDK）
- FFI（跨语言调用 / 原生绑定）

**问题 3 — 项目描述（可选，自由文本）**：
- 请用户简要描述项目的核心业务/功能，例如：「电商订单系统，含支付、库存、物流三条主线」
- 用户可以跳过此项，审计仍可正常执行

**问题 4 — 需求文档路径（可选）**：
- 如有 PRD / 需求规格说明书 / 功能清单，请提供文件路径
- 提供后将启用「业务-代码对齐度」审计维度
- 用户可跳过

### Step 1.5：记录项目描述

如果用户提供了项目描述，将其作为 **业务上下文** 贯穿整个审计流程：
- L2 冗余逻辑：结合业务语义判断逻辑是否真正冗余，避免误判业务必要的防御性代码
- L4 业务审计：以描述中的业务主线为锚点，逐条验证流程闭环、规则治理、抽象质量
- L5 安全性：根据业务敏感度（如涉及支付、医疗、权限）提升对应检查项的优先级

如果用户未提供，则按通用审计模式执行，L4 基于代码自身推断业务流。

如果用户提供了需求文档路径：
- 读取文档内容，提取功能点清单作为对齐验证基准
- L4 追加「业务-代码对齐度」维度，逐条验证：是否有对应实现、实现语义是否匹配
- 文档过大时提示用户指定核心章节范围

### Step 2：加载参考文件

根据用户选择，使用 `Read` 工具读取对应的 reference 文件：

1. **始终加载**：`references/core-prompt.md` + `references/lang-core.md` + `references/type-core.md`
2. **按语言加载**（可多个）：
   - Rust → `references/lang-rust.md`
   - Flutter / Dart → `references/lang-flutter.md`
   - Java → `references/lang-java.md`
   - JavaScript / TypeScript → `references/lang-js-ts.md`
   - Kotlin → `references/lang-kotlin.md`
   - C → `references/lang-c.md`
   - C++ → `references/lang-cpp.md`
   - Zig → `references/lang-zig.md`
   - Python → `references/lang-python.md`
3. **按项目类型加载**（可多个）：
   - Web 服务 → `references/type-web.md`
   - CLI → `references/type-cli.md`
   - 前端 → `references/type-frontend.md`
   - Library → `references/type-library.md`
   - FFI → `references/type-ffi.md`

### Step 3：执行审计

将核心提示词 + 所有选中的扩展内容组合为完整的审计指令，然后对用户提供的代码执行审计。审计时严格按照六层递进结构逐层检查，每一项发现都必须标注具体文件路径和行号。

### Step 4：输出报告

按照核心提示词中定义的输出格式生成报告，包含：
- 审计摘要（🔴🟡🟢 分级 + 安全风险等级 + 业务审计发现数）
- 逐项发现表
- 安全风险专项（按 OWASP Top 10）
- 业务审计报告
- 依赖关系图（Mermaid）
- 清理路线图（Phase 1 安全 → Phase 2 死代码 → Phase 3 架构）

如果项目较大，可以建议用户分模块审计，最后做跨模块汇总。

## 注意事项

- 每项发现必须有文件+行号佐证，不要猜测
- 通过反射/动态加载/插件机制使用的代码标注而非误报
- 区分「确定无用」和「疑似无用需人工确认」
- 安全问题宁可误报不可漏报
- 如果用户未上传代码，提示用户上传项目文件或指定目录路径
