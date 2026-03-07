# Flutter / Dart 语言专项审计

## Dart 专有
- 未使用的 mixin / extension
- 多余的 late 声明：构造函数或 initState 中立即赋值
- 不必要的 dynamic 类型
- 冗余的 toString()：字符串插值中对 String 调用

## Widget 与 UI
- 僵尸 Widget：定义了但未在组件树中引用
- StatefulWidget 无 setState 调用（应改 Stateless）
- build() 超 200 行未拆分
- 冗余 Container（仅做 padding 应用 Padding/SizedBox）
- 冗余 Builder/LayoutBuilder：未使用 context/constraints

## 状态管理
- 未被监听的 Provider/Bloc/GetX Controller
- 未 dispose 的 StreamSubscription / AnimationController / TextEditingController
- State/Bloc 中定义了但 UI 从未读取的字段
- BlocListener/Consumer 的 listener/builder 为空函数

## pubspec 与资源
- assets 声明了但未被引用；assets 目录有文件但未在 pubspec 声明
- 字体声明了但未通过 fontFamily 引用
- arb 中未引用的 i18n key；多 locale arb 的 key 集合不一致
- 硬编码字符串未走 i18n 流程

## 平台原生层
- AndroidManifest / Info.plist 声明了权限但代码未使用对应功能
- MethodChannel 注册了但 Dart 端无调用
- build.gradle 中 apply 了但无实际使用的 plugin

## Flutter 安全专项
- 明文存储 Token/密码（应用 flutter_secure_storage）
- HTTP 明文通信未禁用（usesCleartextTraffic）
- WebView 未限制 JS 注入
- 证书未做 pinning（中间人风险）
- release 模式未开启代码混淆（可被反编译）
- Deep Link scheme 未校验来源
