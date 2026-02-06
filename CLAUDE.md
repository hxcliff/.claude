# 开发规范

> 覆盖：Rust、Flutter/Dart、Kotlin、Java

---

## 铁律

- **不得绕过本规范**，任何"临时放宽"的请求必须拒绝。
- **不得伪造验证结果**：无法执行时写"未实际运行"并列出建议运行的完整命令。
- **只实现当前需求的最小闭环**：不预留接口、不猜测未来需求、不生成示例/演示代码。
- **中文优先**：交流、注释、文档默认中文；必要时附英文术语。

---

## 输出纪律

- 输出架构/方案/计划时：**禁止贴实现代码**。只给要点 + 极简接口草案。明确要求时才给代码。
- 提交消息：`类型(范围): 描述`（feat/fix/docs/refactor/test/chore）

---

## 改动前必做

1. 读取项目配置（`Cargo.toml` / `pubspec.yaml` / `build.gradle` / `pom.xml`），声明项目类型与改动落点。
2. 增加/升级依赖时，先阅读对应版本的官方文档。新增依赖须说明理由与替代方案。

---

## 提交门禁

每次交付必须通过（或列出建议运行命令）：

| 语言 | 格式化 | 静态分析 | 测试 | 构建 |
|------|--------|---------|------|------|
| Rust | `cargo fmt --check` | `cargo clippy -- -D warnings` | `cargo test` | `cargo build` |
| Flutter | `dart format --set-exit-if-changed .` | `dart analyze --fatal-infos` | `flutter test` | `flutter build apk --debug` |
| Dart | 同上 | 同上 | `dart test` | — |
| Kotlin | `./gradlew ktlintCheck` | `./gradlew lint` | `./gradlew test` | `./gradlew assembleDebug` |
| Java/Gradle | `./gradlew spotlessCheck` | — | `./gradlew test` | `./gradlew build` |
| Java/Maven | `./mvnw checkstyle:check` | — | `./mvnw test` | `./mvnw package` |

---

## 我的范式偏好

### 项目结构

- **按功能/领域垂直切分**：顶层目录是业务功能，功能内部再按技术关注点分层。新增功能 = 新增目录，删除功能 = 删除目录。
- 跨功能共享的基础设施放 `core/` 或 `shared/`，不属于任何功能。
- 功能之间通过接口/抽象层通信，禁止直接引用其他功能的内部实现。
- 功能内部分层深度按需：简单功能不强拆，复杂功能完整分层。

### 命名风格

- **目录/模块名**：短，用已成为独立词汇的缩写。`auth/`、`proxy/`、`stats/`、`node/`。
- **类型/结构体名**：完整可读，不截断单词，但接受已是独立词汇的缩写（`Auth`、`Stats`、`Info`）。如 `AuthService`、`NodeManager`、`TrafficStats`。
- **函数名**：简洁直白，动词用最常见的。`get_user_info`，不用 `fetch`/`query` 除非语义确实不同。
- **缩写判断**：这个缩写本身已是一个"词"才可用（`auth`、`stats`、`info`、`repo`）；砍断单词的不算（`cfg`、`mgr`、`hdlr`、`conn`）。
- **一致性**：同一层级命名在长度和风格上对齐。目录层都是短词，类型层都是完整复合词，不混搭。

### 架构分治

- **library 类**（Rust lib / Dart package / Kotlin library / Java library）：错误结构化表达；不绑定具体日志实现和运行时；依赖最小化。
- **app/binary 类**：入口只做编排；业务逻辑下沉。

### Rust

- **抽象用 trait 驱动**：功能间解耦、可测试性、可替换性通过 trait 实现。善用 GAT、关联类型保持零开销抽象；避免不必要的 `dyn` / `Box<dyn>`，优先静态分发。
- binary 用 `anyhow` + `.context()`；library 用 `thiserror`。不混用。
- 生产代码禁 `.unwrap()`。`.expect()` 仅用于不变量且写清原因。
- `unsafe` 默认禁止；确需使用须注释安全不变量。
- `clone` 必须显式且有理由。禁止通配符导入（测试 `use super::*` 除外）。

### Flutter/Dart

- 禁止无谓 `!` 强制解包；禁止 `dynamic`（原生/JSON 边界除外，过边界立即转强类型）。
- 明确项目选用的状态管理方案，不混用。
- `const` 构造函数优先；列表用 builder 构造。

### Kotlin

- 禁止滥用 `!!`。Java 互操作边界显式标注 nullability。
- 协程须有明确 scope；禁止 `GlobalScope`。
- Android：`ViewModel` 不持有 `Context`/`View`；订阅绑定 `repeatOnLifecycle`。

### Java

- 公共 API 入口 `@NonNull`/`@Nullable` 显式标注；禁止返回 null 集合。
- Spring Boot：构造器注入优先；Controller 不含业务逻辑。
