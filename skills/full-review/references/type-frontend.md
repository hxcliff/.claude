# 前端项目专项审计（SPA / SSR / 移动端 UI）

## L1 死代码

### 组件
- 僵尸 props / emits：定义了但父组件从未传递/监听
- 冗余 forwardRef / ref：父组件从未通过 ref 操作子组件

### React 专项（如使用）
- useState 的 setState 从未调用（应改为 const 或 useMemo）
- useRef 创建了但 .current 从未读写
- Context Provider 提供了值但无 useContext 消费者
- Redux/Zustand slice 定义了但无组件 selector
- 自定义 Hook 定义了但从未被调用

### Vue 专项（如使用）
- computed 属性未被模板引用
- Pinia/Vuex store 的 state/getter/action 无组件使用
- 自定义 directive 注册了但模板中未使用
- 事件总线注册了监听但事件从未触发

### Flutter UI 专项（如使用）
- 路由表指向已删除的 Widget；命名路由常量无 pushNamed 引用
- Deep Link 配置了但无处理逻辑

### 样式
- CSS Modules / Scoped CSS 中未引用的 class
- Tailwind extend 中未使用的自定义 color/spacing
- styled-components / emotion 定义了但未使用的 styled 组件
- CSS 变量定义了但无 var() 引用
- 全局 CSS 残留已删除组件的样式规则

### 路由
- 动态参数定义了但页面未读取
- 嵌套路由 layout 存在但子路由已全部迁走

### 数据获取
- React Query/SWR/Apollo hook 定义了但从未在组件中调用
- API 请求函数（services/*.ts）定义了但无引用
- GraphQL query/mutation 定义了但无 useQuery/useMutation
- 数据 mock 存在但对应 API 已改为真实接口

### 性能冗余
- 未使用的 polyfill（目标浏览器已原生支持）

## L2 冗余逻辑

### 组件
- 过度拆分：仅一个调用方且不足 30 行
- 多页面复制粘贴的相同 UI 结构，可提取公共组件并明确归属（通用 UI / 业务组件）

### React 专项
- useEffect 依赖数组遗漏或过多导致频繁/不触发
- useCallback/useMemo 无实际优化效果（依赖数组空或全包含）
- 不必要的 Fragment 包裹单个子元素

### Vue 专项
- watch handler 为空操作

### Flutter UI 专项
- StatelessWidget 内部需要状态但未用状态管理

### 样式
- z-index 无管理策略（999, 99999 散落各处）

### 数据获取
- WebSocket 订阅建立了但 onMessage handler 为空

### 性能冗余
- 未做 code splitting 的大型页面（React.lazy / dynamic import 缺失）
- import 整个库而非按需引入（如 import _ from 'lodash'）
- public/ 下大文件（>500KB）未压缩
- 同页面多组件独立请求相同数据但未共享缓存
- SSR/SSG 页面中客户端专用 import 未动态导入

## L5 安全

### 路由
- 路由 middleware 配置了权限但检查逻辑已移除

### 前端安全专项
- XSS：用户输入直接通过原始 HTML 注入 API（React/Vue 的不安全渲染方法）
- 敏感数据存 localStorage（应走 httpOnly cookie）
- iframe 未设置 sandbox 属性
- postMessage 未校验 origin
- 第三方脚本未设置 integrity (SRI)
