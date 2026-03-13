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

### Step 1：询问用户选择

使用 `AskUserQuestion` 工具，一次性问四个问题：

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
