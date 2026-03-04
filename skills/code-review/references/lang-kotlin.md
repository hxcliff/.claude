# Kotlin 语言专项审计

## 类与对象
- 未使用的 extension function（定义了但调用方为零）
- sealed class / sealed interface 中从未匹配的 variant
- companion object 成员未被外部引用
- data class 的 componentN / copy 从未被调用（可改为普通 class）
- object 声明但无任何引用（僵尸单例）
- 未使用的内部类 / 嵌套类

## 空安全与类型
- `!!` 泛滥：应改为安全调用 `?.` 或 Elvis `?:`
- 冗余 `?.` 链：接收者已为非空类型仍用安全调用
- 冗余 `as?`：类型已经确定仍用安全转换
- 不必要的 lateinit：可在构造时或声明处直接初始化
- 冗余 `is` 检查：smart cast 已覆盖的场景重复判断
- 不必要的平台类型标注：Kotlin 已推导为非空仍加 `!!`

## 协程
- launch / async 产生的 Deferred 从未 await（错误被静默吞掉）
- GlobalScope 滥用：应使用结构化并发（viewModelScope / lifecycleScope / 自定义 scope）
- Channel 创建但发送端或接收端从未使用
- Flow 收集器为空操作（collect {}）
- withContext 切换到与当前相同的 dispatcher（无效切换）
- SupervisorJob 下子协程异常未处理（未设 CoroutineExceptionHandler）
- runBlocking 在主线程或协程内部使用（阻塞风险）

## Android 专项（如使用）
- ViewModel 中定义了 LiveData / StateFlow 但 UI 层从未 observe / collect
- Compose 中 remember 了但未读取的状态
- @Composable 函数定义但未在任何组合中调用
- Hilt @Inject 构造注入但无使用方
- Navigation graph 中孤立 destination（无 route 可达）
- WorkManager 注册了但约束条件永不满足
- Fragment / Activity 中直接持有长生命周期引用（内存泄漏风险）

## Gradle KTS
- 多模块重复声明 parent 已有的依赖
- 自定义 task 从未被执行（无依赖链引用）
- buildSrc / version catalog 中未引用的常量或 plugin
- kapt / ksp 处理器声明了但无对应注解使用

## 冗余模式
- scope function（let/run/with/apply/also）嵌套超 2 层
- 单表达式函数体中多余的 return + 显式返回类型声明
- 冗余 when 分支：某分支行为与 else 完全一致
- 冗余 it 参数命名：lambda 单参数用 it 但又显式命名为 it
- 不必要的 Pair / Triple：应定义语义明确的 data class

## Kotlin 安全专项
- 序列化框架（kotlinx.serialization / Moshi）未限制多态类型
- kotlin-reflect 反射调用未校验输入（可导致任意方法调用）
- @JvmStatic / @JvmField 暴露了不应对 Java 侧可见的内部 API
- Ktor / Spring 路由未做鉴权校验
