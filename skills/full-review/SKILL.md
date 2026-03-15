---
name: full-review
description: >
  六层递进式项目代码审计（死代码 → 冗余逻辑 → 依赖健康 → 业务审计 → 安全性 → 全局健康度）。
  支持 Rust / Flutter / Go / Java / JS·TS / Kotlin / C / C++ / Zig / Python 与 Web / CLI / Frontend / Library / FFI 的自由组合。
  当用户发送 /full-review 或提到代码审计、代码检查、死代码清理、安全审计、项目体检时触发。
---

# Code Review Skill

当用户发送 `/full-review` 或表达"审计/检查项目代码"意图时触发。

## 工作流程

### Step 1：询问用户选择

使用 `AskUserQuestion` 工具，一次性问两个问题：

**问题 1 — 语言（以下选项，支持多选）**：
- Rust
- Flutter / Dart
- Go
- Java
- JavaScript / TypeScript
- Kotlin
- C
- C++
- Zig
- Python

**问题 2 — 项目类型（以下选项，支持多选）**：
- Web（后端 API / 全栈）
- CLI（命令行工具）
- Frontend（SPA / SSR / 移动端 UI）
- Library（可复用库 / SDK）
- FFI（跨语言调用 / 原生绑定）

### Step 2：加载参考文件

根据用户选择，使用 `Read` 工具读取对应的 reference 文件：

1. **始终加载**：`references/core-prompt.md` + `references/lang-core.md` + `references/type-core.md`
2. **按语言加载**（可多个）：
   - Rust → `references/lang-rust.md`
   - Flutter / Dart → `references/lang-flutter.md`
   - Go → `references/lang-go.md`
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
   - Frontend → `references/type-frontend.md`
   - Library → `references/type-library.md`
   - FFI → `references/type-ffi.md`

### Step 3：执行审计

将核心提示词 + 所有选中的扩展内容组合为完整的审计指令，然后对用户提供的代码执行审计。审计时严格按照六层递进结构逐层检查，每一项发现都必须标注具体文件路径和行号。

### Step 4：输出报告

按照核心提示词中定义的输出格式生成报告（共 3 章节）：
1. 审计摘要（🔴🟡🟢 分级 + 安全风险等级 + 业务审计发现数）
2. 审计发现（所有层级合并为单一表格，序号从 1 连续自增，按 L1→L6 排列）
3. 清理路线图（Phase 1 安全与业务风险 → Phase 2 死代码与冗余清理 → Phase 3 架构与全局优化，通过序号引用发现表）

如项目较大，建议分模块审计后追加跨模块依赖关系图（Mermaid），最后做跨模块汇总。

## 注意事项

- 如果用户未上传代码或未指定目录路径，提示用户提供后再开始审计
