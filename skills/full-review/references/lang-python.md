# Python 语言专项审计

## 类型标注
- 公共函数缺少类型标注或标注为 `Any`（外部 JSON 边界除外）
- 返回类型标注与实际返回值不一致
- `TypeVar` / `ParamSpec` 定义了但未使用
- `Protocol` / `ABC` 定义了但无实现类
- `@overload` 签名与运行时实现不匹配

## 异常处理
- 裸 `except:` 捕获所有异常（包括 KeyboardInterrupt / SystemExit）
- `except Exception` 未附 log 或 raise（静默吞掉异常）
- 过宽的异常捕获：应精确到具体异常类型
- 自定义异常 raise 了但从未在 except 中被捕获
- `__exit__` 返回 True 吞掉异常但未记录

## 异步
- 异步上下文中调用阻塞 IO（未用 `asyncio.to_thread` 隔离）
- `asyncio.create_task` 创建了但未 await 结果（fire-and-forget 无错误处理）
- async generator 定义了但从未被 `async for` 消费

## 模块与导入
- 同一模块下 ≥ 3 个符号仍逐个 import 而非 `from xxx import a, b, c`
- 循环导入（通过延迟导入掩盖）
- `__init__.py` 导出了但包外未引用的符号
- `__all__` 列表与实际导出不一致

## 类与数据结构
- `dataclass` / pydantic model 字段定义了但未被读写
- 继承链中间类无自身逻辑（纯透传）
- `metaclass` / `__init_subclass__` 定义了但无子类触发
- `property` 定义了 setter 但从未被赋值（或反之）
- `__slots__` 声明与实际使用的属性不一致

## 包管理
- optional dependencies 组定义了但无 extras 引用
- 版本约束过松（无上限）或过紧（锁定 patch 版本）

## 冗余模式
- 生产代码中使用 `print()` 而非 `logging`
- 裸 `dict` 承载配置数据（应用 pydantic / dataclasses）
- 手写循环可用列表推导式 / 生成器表达式替代
- 冗余 `isinstance` 检查：类型已由上游保证

## Python 安全专项
- `eval` / `exec` 使用了用户输入
- `subprocess` 使用 `shell=True` 且参数含用户输入
- `yaml.load` 而非 `yaml.safe_load`
- tempfile 使用可预测路径（竞态条件风险）
- 密码学使用了弱算法（MD5 / SHA1 用于安全场景）
