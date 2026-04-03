```
+--------+      +-------+      +---------+
|  User  | ---> |  LLM  | ---> | Tools   |
| prompt |      |       |      | + todo  |
+--------+      +---+---+      +----+----+
                    ^                |
                    |   tool_result  |
                    +----------------+
                          |
              +-----------+-----------+
              | TodoManager state     |
              | [ ] task A            |
              | [>] task B  <- doing  |
              | [x] task C            |
              +-----------------------+
                          |
              if rounds_since_todo >= 3:
                inject <reminder> into tool_result
```

<font color=red>**问题**</font>：多步任务中, 模型会丢失进度 -- 重复做过的事、跳步、跑偏。对话越长越严重: 工具结果不断填满上下文, 系统提示的影响力逐渐被稀释。一个 10 步重构可能做完 1-3 步就开始即兴发挥, 因为 4-10 步已经被挤出注意力了。

<font color=red>**解决**</font>：存储带状态。同一时间只允许一个 `in_progress`。

"同时只能有一个 in_progress" 强制顺序聚焦。nag reminder 制造问责压力 -- 你不更新计划, 系统就追着你问。

<br/>

| 组件         | 之前 (s02) | 之后 (s03)                 |
| ---------- | -------- | ------------------------ |
| Tools      | 4        | 5 (+todo)                |
| 规划         | 无        | 带状态的 TodoManager         |
| Nag 注入     | 无        | 3 轮后注入 `<reminder>`      |
| Agent loop | 简单分发     | + rounds_since_todo 计数器