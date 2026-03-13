# C++ 语言专项审计

## L1 死代码

### 类与继承
- override 虚函数从未通过基类指针调用（多态无实际效果）
- 未使用的友元声明
- 抽象类中非纯虚函数从未被子类调用

### 模板
- 显式特化从未被实例化
- 模板参数定义了但函数体内未使用

### 构建
- CMake target_link_libraries 链接了未使用的库

## L2 冗余逻辑

### 资源管理
- unique_ptr 场景误用 shared_ptr（不必要的引用计数开销）

### 模板
- SFINAE / concepts 约束恒真或恒假（约束无实际过滤作用）
- 冗余 typename / template 消歧义关键字（非依赖上下文）

### 移动语义
- 可移动对象被不必要拷贝（应 std::move）
- 完美转发遗漏 std::forward（退化为左值引用）
- 返回局部对象时多余 std::move（阻碍 NRVO 优化）

### STL 使用
- 容器选型不当（频繁中间插入用 vector 而非 list / deque）
- 可 reserve 但未预分配导致频繁扩容
- find + insert 可用 try_emplace 替代（减少重复查找）
- std::endl 滥用（应使用 '\n'，避免不必要的 flush）

### 现代 C++ 升级
- C 风格转换应改 static_cast / dynamic_cast / const_cast
- 原始数组应改 std::array / std::vector
- NULL 应改 nullptr
- 宏常量应改 constexpr
- typedef 应改 using
- 手写循环可用 range-based for 或算法库替代

## L5 安全

### 资源管理
- 原始 new / delete 未用智能指针管理
- shared_ptr 循环引用缺 weak_ptr 打破
- 缺少 Rule of Five：自定义了析构 / 拷贝 / 移动中的部分但未补全
- RAII 包装缺失：裸 FILE* / socket fd / mutex lock 未用 RAII 管理

### 类与继承
- 基类析构函数非 virtual 但存在继承（多态析构未定义行为）
- 菱形继承未用 virtual 继承（数据重复）

### 移动语义
- std::move 后仍读取源对象（moved-from 状态未定义）

### STL 使用
- 遍历中修改容器致迭代器失效

### C++ 安全专项
- 裸指针跨线程共享无同步机制
- const_cast 移除 const 后写入（未定义行为）
- reinterpret_cast 用于不相关类型（严格别名违规）
- 异常安全基本保证缺失（抛异常后资源泄漏）
- 未捕获的异常穿越 noexcept 边界（std::terminate）
- 虚函数在构造 / 析构中被调用（不会动态分派）

## L6 全局健康

### 构建
- 不同 target 的 C++ 标准版本不一致
- header-only 库被多个翻译单元重复编译（编译时间膨胀）
- PCH 包含了变动频繁的头文件
