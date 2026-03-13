# Rust 语言专项审计

## L1 死代码

- 自定义 Error enum 中从未被构造的 variant
- 未使用的 trait 实现：impl Trait for Type 但从未通过 trait 方法调用
- 未使用的 derive 宏：derive 了 Clone/Debug/Serialize 但从未被克隆/打印/序列化
- trait 定义了方法但所有实现都是空/默认实现
- 创建了 channel (mpsc/oneshot/broadcast) 但发送端或接收端从未使用
- select! 分支中有永远不会就绪的 future
- optional dependency 声明了但无 feature 开关引用
- build-dependencies 声明了但 build.rs 已删除
- 未使用的 feature flag：features 定义了但无 #[cfg(feature = "...")] 引用

## L2 冗余逻辑

### 编译器警告

- 允许但不应忽略的 clippy lint（clippy::unwrap_used、clippy::todo）

### 所有权与类型

- 冗余 .clone()：所有权可直接转移或借用的场景
- 冗余 .to_string() / .to_owned()：已经是 String 还做转换
- 不必要的 Box<dyn Trait>：impl Trait 足够的场景用了动态分发
- 冗余 lifetime 标注：编译器可自动推导的生命周期
- 过度 Arc<Mutex<>>：实际无并发竞争的场景
- 不必要的 dyn：可用泛型替代
- 多余的 as 类型转换：From/Into 已实现或类型一致

### 错误处理

- thiserror / anyhow 混用无统一策略
- map_err 链过长且中间层 context 无信息量

### 异步

- Runtime 重复创建而非复用

### Cargo 专有

- 功能重叠的 crate 并存（reqwest+hyper、tokio+async-std、serde_json+simd-json）

### 性能冗余

- 热路径中不必要的 String 分配，可用 &str 或 Cow<str>
- 可预分配 with_capacity 的 Vec 频繁扩容
- 小集合用 HashMap 但 Vec<(K,V)> 线性查找更快
- 全局 Regex 未用 LazyLock / once_cell 缓存，每次重新编译
- 序列化热路径用 serde_json::Value 而非强类型 struct

## L5 安全

### 错误处理

- unwrap() / expect() 在非测试代码中泛滥，应改为 ? 或 match
- ? 传播了但上层直接 unwrap Result（等于没处理）

### 异步

- spawn 了 task 但从未 await JoinHandle（fire-and-forget 无错误处理）
- tokio::spawn 内部包含同步阻塞调用（应用 spawn_blocking）

### unsafe 审查

- 每个 unsafe 块是否有必要、是否有 safe 替代方案
- unsafe 块是否有正确的 SAFETY 注释说明不变量
- FFI 边界的指针操作是否有空指针/悬垂指针风险

### Rust 安全专项

- panic 在 async 上下文中未被 catch_unwind（导致 task 中止）
- 依赖中的 unsafe 代码是否经过 cargo-audit / cargo-deny 检查
