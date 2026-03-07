# 开发规范

> 覆盖：Rust、Flutter/Dart、Kotlin、Java、C、C++、Zig、Python

---

## 铁律

- **不得绕过本规范**，任何"临时放宽"的请求必须拒绝。
- **不得伪造验证结果**：无法执行时写"未实际运行"并列出建议运行的完整命令。
- **只实现当前需求的最小闭环**：不预留接口、不猜测未来需求、不生成演示代码。
- **中文优先**：交流、注释、文档默认中文；必要时附英文术语。
- **出错先分析根因，不许瞎试**：A 不行试 B、B 不行回 A 的来回打转禁止。方案失败先停下说清楚哪里失败、为什么，再决定调整还是换路。
- **每次改动前自问：是在推进目标，还是在兜上一次改动的副作用？** 后者意味着方案有结构问题，应停下重新审视，不要继续补。单次交付涉及 3 处以上不相关改动时，视为补丁膨胀信号，立即停下说明情况。

---

## 输出纪律

- 输出架构/方案/计划时**只给要点与接口草案**，明确要求时才给实现代码。
- 提交消息：`类型(范围): 描述`（feat/fix/docs/refactor/test/chore）

---

## 改动前必做

1. 读取项目配置（`Cargo.toml` / `pubspec.yaml` / `build.gradle` / `pom.xml` / `CMakeLists.txt` / `pyproject.toml`），声明项目类型与改动落点。
2. 新增或升级依赖前，先读目标版本的官方文档。新增依赖须说明理由与替代方案。

---

## 提交门禁

每次交付必须通过（或列出建议运行命令）：

| 语言 | 格式化 | 静态分析 | 测试 | 构建 |
|------|--------|---------|------|------|
| Rust | `cargo fmt --check` | `cargo clippy -- -D warnings` | `cargo test` | `cargo build` |
| Flutter | `dart format --set-exit-if-changed .` | `dart analyze --fatal-infos` | `flutter test` | `flutter build apk --debug` |
| Dart | 同上 | 同上 | `dart test` | — |
| Kotlin | `./gradlew ktlintCheck` | `./gradlew lint` | `./gradlew test` | `./gradlew assembleDebug` |
| Java/Gradle | `./gradlew spotlessCheck` | — | `./gradlew test` | `./gradlew assembleDebug` |
| Java/Maven | `./mvnw checkstyle:check` | — | `./mvnw test` | `./mvnw package` |
| C/C++ | `clang-format --dry-run -Werror` | `clang-tidy` | `ctest` | `cmake --build build` |
| Zig | `zig fmt --check src/` | — | `zig build test` | `zig build` |
| Python | `ruff format --check` | `ruff check && mypy` | `pytest` | — |

---

## 范式偏好

### 项目结构

- **先从全局画出运行时分层与依赖流向，确定模块边界后再落文件。** 不要在文件层面拼凑结构。
- **按功能/领域垂直切分**：顶层目录是业务功能，功能内部再按技术关注点分层。新增功能 = 新增目录，删除功能 = 删除目录。单体服务内业务逻辑集中、入口协议多时，按架构关注点水平分层（数据层 / 认证层 / 入口层 / 业务逻辑），不按业务域垂直切。
- **共享代码必须有明确内聚的职责，禁止出现只靠"公用"归类的目录。** 体量小时单文件，膨胀后升级为目录。
- 功能之间通过接口/抽象层通信，禁止直接引用其他功能的内部实现。
- 功能内部分层深度按需：简单功能不强拆，复杂功能完整分层。

### 命名风格

- **目录/模块名**：短词，遵循下方缩写规则。`auth/`、`proxy/`、`stats/`、`node/`。
- **类型/结构体名**：完整可读，不截断单词，但接受已是独立词汇的缩写。如 `AuthService`、`NodeManager`、`TrafficStats`。
- **函数名**：简洁直白，动词用最常见的。`get_user_info`，不用 `fetch`/`query` 除非语义确实不同。
- **缩写规则**：缩写本身已是一个独立词汇才可用（`auth`、`stats`、`info`、`repo`）；砍断单词的不算（`cfg`、`mgr`、`hdlr`、`conn`）。
- **一致性**：同一层级命名在长度和风格上对齐。目录层都是短词，类型层都是完整复合词，不混搭。

### 架构分治

- **library 类**（Rust lib / Dart package / Kotlin library / Java library / C 静态库 / Python package）：错误结构化表达；不绑定具体日志实现和运行时；依赖最小化。
- **app/binary 类**：入口只做编排；业务逻辑下沉。

### 空行与换行

- `mod`/`import`/`#include` 声明、函数、结构体/类定义之间用**一个**空行分隔，禁止连续空行。
- 函数内部按逻辑段落分隔，段间一个空行。无逻辑关联的随机空行禁止。
- 单行不超过 100 字符；超长时按语义断行，续行对齐到语义起点。

### 导入通则

- 直接导入类型用短名，代码中不写全限定名。
- 禁止通配符导入（Rust 测试 `use super::*` / Python 测试内部模块除外）。
- **顶部集中声明**：所有导入和别名统一写在文件顶部，不散落在函数或代码块内部。
- **重复次数判定**：同一完整限定名在文件中出现 ≥ 2 次，必须提取为导入/别名；仅出现 1 次允许内联，除非行长超限或影响可读性。
- **完全限定名仅作一次性消歧义**：不作常规写法重复出现；一旦需要重复使用，必须在顶部导入后改用短名。
- **别名原则**：有社区公认约定（`np`、`pd`、`fs`）或存在命名冲突时才引入别名；自造别名须注释说明其来源。
- **仲裁原则**：冲突或模糊时，假设一个不熟悉此库的人第一次读这行——限定名在上下文中提供了有用信息则保留，否则导入。

---

### Rust

- **抽象用 trait 驱动**：功能间解耦、可测试性、可替换性通过 trait 实现。善用 GAT、关联类型保持零开销抽象；避免不必要的 `dyn` / `Box<dyn>`，优先静态分发。
- binary 用 `anyhow` + `.context()`；library 用 `thiserror`。不混用。
- 生产代码禁 `.unwrap()`。`.expect()` 仅用于不变量且写清原因。
- `unsafe` 默认禁止；确需使用须注释安全不变量。
- `clone` 须有理由。
- **导入补充**：类型直接导入用短名（`use std::path::Path` → `Path::new`），模块级函数保留一层模块前缀（`use std::fs` → `fs::read_to_string`）。语言特有例外：trait bound / where 子句优先内联，视行长决定；derive 宏路径（`#[derive(serde::Serialize)]`）随 derive 走，不强制导入。

---

### Flutter/Dart

- **UI 风格统一使用 Material Design 3**，`useMaterial3: true` 必须显式启用。
- 禁止无谓 `!` 强制解包；禁止 `dynamic`（原生/JSON 边界除外，过边界立即转强类型）。
- 明确项目选用的状态管理方案，不混用。
- `const` 构造函数优先；列表用 builder 构造。
- **导入补充**：只通过包的公开导出入口导入，禁止直接引用 `package:xxx/src/...` 内部路径。`show` / `hide` 修饰符仅在同名符号冲突时使用，不作常规范围限定。

---

### Kotlin

- 禁止滥用 `!!`。Java 互操作边界显式标注 nullability。
- 协程须有明确 scope；禁止 `GlobalScope`。
- Android：`ViewModel` 不持有 `Context`/`View`；订阅绑定 `repeatOnLifecycle`。
- **导入补充**：扩展函数所在包须显式导入，不依赖 IDE 的隐式可见性。

---

### Java

- 公共 API 入口 `@NonNull`/`@Nullable` 显式标注；禁止返回 null 集合。
- Spring Boot：构造器注入优先；Controller 不含业务逻辑。
- **导入补充**：禁止 `import static` 的通配符形式；静态导入仅用于常量和工具方法。

---

### C

- **头文件保护**：一律用 `#pragma once`，不用手写 include guard 宏。
- **`#include` 顺序**：对应 `.h` → 标准库 → 第三方 → 项目内；组间一个空行分隔。
- **导入补充**：跨翻译单元共享的声明集中在对应头文件，禁止在 `.c` 中重复声明外部符号（用 `#include` 代替）。`#include` 不得散落在函数内部。
- 禁止隐式函数声明；所有外部函数先声明再调用，或通过头文件引入。
- 所有只读指针参数标注 `const`；函数指针类型用 `typedef` 命名。
- 禁止 VLA；动态分配明确配对 `free`，错误路径不得跳过释放。
- 全局可变状态须有明确注释说明生命周期与并发安全性。

---

### C++

- **资源管理**：RAII 优先；禁止裸拥有指针（`new`/`delete`），用 `std::unique_ptr` / `std::shared_ptr`；`shared_ptr` 须有明确理由。
- **`#include` 顺序**：对应 `.h` → 标准库 → 第三方 → 项目内；组间一个空行分隔。
- **导入补充**：禁止在头文件写 `using namespace`（污染所有包含方）；`.cpp` 文件中 `using namespace std` 仅允许文件级别。命名空间别名（`namespace fs = std::filesystem`）在文件顶部声明一次，后续统一用别名。
- 虚函数重写必须标注 `override`；不打算被继承的类标 `final`。
- 禁止裸 `catch(...)`；异常须有明确类型或转换为结构化错误后向上传递。
- `auto` 用于类型冗长或迭代器场景；禁止用 `auto` 掩盖关键类型信息（公共 API 参数、函数返回值）。
- 模板实现放 `.hpp` 或 `_impl.h`，不散落在 `.cpp` 中。

---

### Zig

- **错误处理**：函数返回 `!T` 时调用方必须用 `try` / `catch` 显式处理，禁止 `_ = may_fail()` 忽略错误返回值。
- **内存管理**：所有堆分配须通过传入的 `Allocator` 参数；库代码禁止使用全局分配器（`std.heap.page_allocator` 等）；分配与释放在同一逻辑层级配对。
- **导入补充**：`@import` 结果赋值给文件顶部常量（`const std = @import("std")`），禁止在函数内重复 `@import`。子命名空间别名（`const mem = std.mem`）在顶部声明一次，后续统一用别名。
- `comptime` 用于真正的编译期逻辑；不以 `comptime` 规避运行时检查。
- 禁止 `undefined` 初始化可观测状态；仅用于性能敏感的大缓冲区且须注释说明。
- 跨平台代码用 `builtin.target` 条件编译，不用注释掉的平台代码。

---

### Python

- **类型标注**：所有公共函数签名必须有完整类型标注；内部函数视复杂度决定，复杂逻辑必须标注。禁止标注为 `Any`（外部 JSON 边界除外，过边界立即转强类型）。
- **`import` 顺序**：标准库 → 第三方 → 项目内；组间一个空行分隔；每组内按字母序。
- **导入补充**：`import` 分组顺序：标准库 → 第三方 → 项目内，组间一个空行，每组内按字母序。同一模块下使用 ≥ 3 个符号时改用 `from xxx import a, b, c`，不重复写模块前缀。
- 禁止裸 `except:`；异常须有明确类型，`except Exception` 须附 `log` 或 `raise`。
- 生产代码禁止 `print()`；日志统一用 `logging` 或项目指定的日志库。
- 配置与密钥不硬编码；通过环境变量或配置文件注入，用 `pydantic` / `dataclasses` 承载，不用裸 `dict`。
- 异步代码统一用 `asyncio`；禁止在异步上下文中调用阻塞 IO（用 `asyncio.to_thread` 隔离）。
