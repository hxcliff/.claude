# Zig 语言专项审计

## 错误处理
- 函数返回 `!T` 但调用方用 `_ = may_fail()` 忽略错误返回值
- `catch unreachable` 用于非不变量场景（应显式处理错误）
- 错误集合中定义了但从未被返回的 error 值
- `try` 链过长但中间层无附加上下文（错误来源不可追溯）
- errdefer 缺失：分配资源后的错误路径未释放

## 内存管理
- 库代码使用全局分配器而非接受传入的 Allocator
- alloc / create 无配对 free / destroy
- defer / errdefer 遗漏：资源分配后退出路径未释放
- ArrayList / HashMap 等容器未调用 deinit
- 多次分配但仅在部分错误路径释放（不对称的 errdefer）

## Comptime
- comptime 块执行了运行时也能完成的逻辑（滥用编译期计算）
- comptime 参数定义了但函数体内未在编译期使用
- inline for / inline while 用于非性能关键路径（增加编译时间无收益）

## 类型与表达式
- `@as` 转换目标与源类型相同（无意义转换）
- `@intCast` / `@truncate` 未做范围检查（溢出风险）
- optional 解包用 `.?` 但未处理 null 分支（等同 unreachable）
- `union(enum)` 部分 variant 从未被构造或匹配
- packed struct 未验证目标平台的字节序与对齐

## 构建
- build.zig 中声明的 module / step 从未被引用
- build.zig.zon 中声明的依赖未被 `@import`
- 编译 flag 禁用了关键安全检查
- 不同 target 的构建配置不一致但无条件编译区分
- test 块中无断言的空测试函数

## Zig 安全专项
- `undefined` 用于可观测状态初始化（非大缓冲区场景）
- `@ptrCast` / `@alignCast` 未验证源指针的有效性
- 跨 C ABI 边界的指针未校验 null / 长度
- extern fn 回调中 panic 导致未定义行为（应在边界处捕获）
- 用户输入直接用于切片索引（越界风险）
