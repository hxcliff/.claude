# Python 语言专项审计

## L1 死代码

### 类型标注
- `TypeVar` / `ParamSpec` 定义了但未使用
- `Protocol` / `ABC` 定义了但无实现类

### 异常处理
- 自定义异常 raise 了但从未在 except 中被捕获

### 异步
- async generator 定义了但从未被 `async for` 消费

### 模块与导入
- `__init__.py` 导出了但包外未引用的符号

### 类与数据结构
- `dataclass` / pydantic model 字段定义了但未被读写
- `metaclass` / `__init_subclass__` 定义了但无子类触发
- `property` 定义了 setter 但从未被赋值（或反之）

### 包管理
- optional dependencies 组定义了但无 extras 引用

## L2 冗余逻辑

### 类型标注
- 公共函数缺少类型标注或标注为 `Any`（外部 JSON 边界除外）
- 返回类型标注与实际返回值不一致
- `@overload` 签名与运行时实现不匹配

### 模块与导入
- 同一模块下 ≥ 3 个符号仍逐个 import 而非 `from xxx import a, b, c`
- `__all__` 列表与实际导出不一致

### 类与数据结构
- 继承链中间类无自身逻辑（纯透传）
- `__slots__` 声明与实际使用的属性不一致

### 冗余模式
- 生产代码中使用 `print()` 而非 `logging`
- 裸 `dict` 承载配置数据（应用 pydantic / dataclasses）
- 手写循环可用列表推导式 / 生成器表达式替代
- 冗余 `isinstance` 检查：类型已由上游保证

## L3 依赖健康

### 模块与导入
- 循环导入（通过延迟导入掩盖）

### 包管理
- 版本约束过松（无上限）或过紧（锁定 patch 版本）

## L5 安全

### 异常处理
- 裸 `except:` 捕获所有异常（包括 KeyboardInterrupt / SystemExit）
- `except Exception` 未附 log 或 raise（静默吞掉异常）
- 过宽的异常捕获：应精确到具体异常类型
- `__exit__` 返回 True 吞掉异常但未记录

### 异步
- 异步上下文中调用阻塞 IO（未用 `asyncio.to_thread` 隔离）
- `asyncio.create_task` 创建了但未 await 结果（fire-and-forget 无错误处理）

### 资源管理
- open() / 数据库连接 / socket 未使用 with 语句（资源泄漏风险）

### Python 安全专项
- `eval` / `exec` 使用了用户输入
- `subprocess` 使用 `shell=True` 且参数含用户输入
- `yaml.load` 而非 `yaml.safe_load`
- tempfile 使用可预测路径（竞态条件风险）
- 密码学使用了弱算法（MD5 / SHA1 用于安全场景）
