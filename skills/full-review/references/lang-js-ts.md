# JavaScript / TypeScript 语言专项审计

## L1 死代码

### 模块
- package.json scripts 中从未执行的命令

### TypeScript 专项
- 枚举成员定义了但从未引用
- d.ts 声明文件对应模块已被删除

### 异步与事件
- 事件监听器注册了但从未触发或从未 removeListener

### 构建与配置
- tsconfig.json paths 别名定义了但未使用
- babel 插件配置了但已切 SWC/esbuild
- webpack/vite 自定义 loader/plugin 但对应文件类型不存在
- postcss 插件配置了但无匹配的 CSS 语法
- .env 中定义了但 process.env.* / import.meta.env.* 未引用的变量

## L2 冗余逻辑

### TypeScript 专项
- 过度 any：可推断或应标注的地方
- 类型断言 as 过多（频繁 as unknown as X 表明类型设计有问题）
- 多文件中结构相同的重复 type/interface

### 异步与 Promise
- .then().catch() 与 async/await 混用风格不统一

## L5 安全

### 异步与 Promise
- Promise 未 catch 且无全局兜底

### JS/TS 安全专项
- 动态代码执行函数使用了用户输入（如 eval 系列）
- prototype pollution 风险：深度 merge 用户输入到对象
