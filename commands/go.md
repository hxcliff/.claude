解析 $ARGUMENTS，判断输入类型并执行：

1. **如果是文件路径**（以 `/`、`./`、`~/` 开头，或以常见文档扩展名 `.md`、`.txt`、`.org`、`.todo` 结尾）：
   读取该文件，逐条实现里面的任务。

2. **如果是文案描述但引用了文件**（文案中包含文件路径）：
   先读取引用的文件获取上下文，再结合文案描述执行任务。

3. **如果是纯文案描述**（不含文件路径）：
   直接按文案内容拆解并逐条实现任务。

每完成一个任务就 git commit。
全部完成后 Output <promise>COMPLETE</promise>

使用 ralph-loop，--completion-promise "COMPLETE" --max-iterations 30
