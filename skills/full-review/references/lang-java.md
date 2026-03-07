# Java 语言专项审计

## 类与对象
- 仅一个实现的接口且无 SPI/策略模式需求（过度设计）
- POJO/DTO/VO/Entity 字段从未被读写（含 Lombok @Data 检查）
- 冗余继承链：中间类无自身逻辑，纯透传
- 未使用的内部类/匿名类

## Spring 框架（如使用）
- @Service/@Component/@Repository 注解但从未被注入
- @Bean 定义了但无组件依赖；@Value 注入了但类中未使用
- application.yml 中定义了但无 @Value/@ConfigurationProperties 读取的配置项
- 多 profile 配置文件中完全相同的配置项
- @Scheduled cron 永不触发；@EventListener 对应事件从未 publish
- @Transactional 标注但方法内无写操作
- @Aspect pointcut 匹配不到任何现存方法

## 持久层
- MyBatis XML 与 Mapper 接口 SQL 不对应
- JPA Repository 自定义方法从未被 Service 调用
- Flyway/Liquibase 建的表无对应 Entity
- N+1 查询；事务中某些路径未 commit/rollback

## Maven / Gradle 专有
- 多模块重复声明 parent 已有的依赖
- 依赖版本冲突（不同模块引入同一依赖的不同版本）
- release 分支残留 snapshot 版本
- provided scope 但运行时实际需要（反之亦然）

## 冗余模式
- 过度 Optional 且调用方直接 .get()
- 单线程场景的 synchronized
- 循环内 String + 拼接（应 StringBuilder）
- 工具类超 500 行，职责不单一
- 无效的 @SuppressWarnings

## Java 安全专项
- Actuator 端点未鉴权暴露到公网
- catch 块 e.printStackTrace() 而非日志框架
- Spring Security 部分路径被 permitAll 遗漏
- CORS allowedOrigins("*")
