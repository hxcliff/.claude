# Web 服务项目专项审计（后端 API / 全栈）

## L1 死代码

### 路由与端点
- 错误类型/响应类型定义了 variant 但从未被构造

### 数据持久层
- ORM Model/Entity 定义了字段但查询时从未 select/insert 该字段
- Repository/DAO 函数定义了但无 service 层调用
- 数据库迁移脚本建的表/字段在代码中无对应映射

### 序列化与 API 契约
- 请求/响应结构定义了字段但永远为 null/空
- DTO 与 Controller 返回时从未填充的字段

### 缓存与消息队列
- 缓存 key 定义了但从未被读取或写入
- 消息队列 consumer 注册了但对应 topic 从未有 producer 发送
- 定时任务配置了但执行逻辑为空或已无意义

## L2 冗余逻辑

### 数据持久层
- 连接池配置了但实际并发远低于上限
- N+1 查询：循环逐条查询可改为批量

## L5 安全

### 数据持久层
- 事务 begin 了但某些分支未 commit/rollback（事务泄漏）

### Web 安全专项
- CORS / 同源策略是否正确配置
- Cookie 未设置 HttpOnly / Secure / SameSite
- WebSocket 连接未验证身份
- URL 重定向参数未校验目标地址（开放重定向）
- 响应头缺少安全头（X-Frame-Options / CSP / HSTS）

## L6 全局健康

### 序列化与 API 契约
- API 文档（Swagger/OpenAPI）描述与实际签名不一致

### 部署与运行时
- 健康检查端点检查的服务已下线
- 日志级别不当：生产环境配置了 DEBUG
