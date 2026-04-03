> _"大任务拆小, 每个小任务干净的上下文"_-- Subagent 用独立 messages[], 不污染主对话。
> 
> **Harness 层**: 上下文隔离 -- 守护模型的思维清晰度。

```
Parent agent                     Subagent
+------------------+             +------------------+
| messages=[...]   |             | messages=[]      | <-- fresh
|                  |  dispatch   |                  |
| tool: task       | ----------> | while tool_use:  |
|   prompt="..."   |             |   call tools     |
|                  |  summary    |   append results |
|   result = "..." | <---------- | return last text |
+------------------+             +------------------+

Parent context stays clean. Subagent context is discarded.
```

任务太大，分而治之，单一职责。

父 Agent 有一个`task`工具。Subagent 拥有除`task`外的所有基础工具 (禁止递归生成)。


<br/>

| 组件       | 之前 (s03) | 之后 (s04)            |
| -------- | -------- | ------------------- |
| Tools    | 5        | 5 (基础) + task (仅父端) |
| 上下文      | 单一共享     | 父 + 子隔离             |
| Subagent | 无        | `run_subagent()`函数 |
| 返回值      | 不适用      | 仅摘要文本