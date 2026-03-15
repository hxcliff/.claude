# Go 语言专项审计

## L1 死代码

### 函数与类型

- 导出函数/类型/常量但包外无引用（尤其 internal/ 下的包）
- interface 定义了但无类型实现
- struct 方法定义了但从未通过该类型调用
- 嵌入的 interface/struct 但未使用其提供的方法

### 并发

- channel 创建了但发送端或接收端从未使用
- select 分支中有永远不会就绪的 channel

### 构建

- //go:build 条件编译的代码在所有构建目标下均不可达
- //go:generate 指令指向的工具或脚本已删除

## L2 冗余逻辑

### 错误处理

- error 值只用 errors.New 创建但调用方从未用 errors.Is/As 区分（应直接用字符串或合并）
- fmt.Errorf 包装但未添加有意义的上下文（纯透传 %w）
- 多层 if err != nil { return err } 但中间层无附加信息

### 类型与表达式

- interface{}/any 泛滥：Go 1.18+ 可用泛型约束的场景
- 冗余 nil 检查：返回值已由类型或上游逻辑保证非 nil
- fmt.Sprintf 仅做简单字符串拼接（应直接 + 或 strings.Builder）
- 不必要的类型转换：源类型与目标类型相同

### 并发

- 冗余同步：只有一个 goroutine 访问的资源加了 sync.Mutex / sync.RWMutex
- 冗余 context.Background()：应传递父 context 的场景

### Go module

- replace 指令指向本地路径但未限制为开发环境
- 功能重叠的依赖并存（如 logrus + zap，gin + echo）

## L3 依赖健康

### Go module

- go.mod 主版本 v2+ 但 import path 未包含版本后缀
- 间接依赖版本冲突（同一模块的不同主版本被拉入）

## L5 安全

### 错误处理

- recover 吞掉 panic 但未记录堆栈（静默隐藏崩溃根因）

### 并发

- goroutine 泄漏：启动了但无 context 取消或 channel 关闭的退出条件
- 共享变量无 sync 保护（-race 可检测的数据竞争）
- defer 在循环内：资源（文件/锁/连接）延迟到函数结束才释放

### 资源管理
- os.File / http.Response.Body / sql.Rows 等资源 Open 后未 defer Close()

### Go 安全专项

- unsafe.Pointer 转换未注释安全不变量
- math/rand 用于安全场景（应用 crypto/rand）
- http.DefaultClient 无超时设置（可被慢响应阻塞）
- http.ListenAndServe 未设置 ReadTimeout / WriteTimeout / IdleTimeout
- template.HTML() 强制不转义用户输入（XSS 风险）
- 文件操作未设置权限位（os.Create 默认 0666，敏感文件应限制）
