> _"大目标要拆成小任务, 排好序, 记在磁盘上"_-- 文件持久化的任务图, 为多 agent 协作打基础。
> 
> **Harness 层**: 持久化任务 -- 比任何一次对话都长命的目标。

**问题**：s03 的 TodoManager 只是内存中的扁平清单: 没有顺序、没有依赖、状态只有做完没做完。真实目标是有结构的 -- 任务 B 依赖任务 A, 任务 C 和 D 可以并行, 任务 E 要等 C 和 D 都完成。

没有显式的关系, Agent 分不清什么能做、什么被卡住、什么能同时跑。而且清单只活在内存里, 上下文压缩 (s06) 一跑就没了。

—— 大白话：需要支持复杂任务DAG、持久化

**解决**：把扁平清单升级为持久化到磁盘的任务图。每个任务是一个 JSON 文件, 有状态、前置依赖 (`blockedBy`)。任务图随时回答三个问题:

-   什么可以做?-- 状态为`pending`且`blockedBy`为空的任务。
-   什么被卡住?-- 等待前置任务完成的任务。
-   什么做完了?-- 状态为`completed`的任务, 完成时自动解锁后续任务。

```
.tasks/
  task_1.json  {"id":1, "status":"completed"}
  task_2.json  {"id":2, "blockedBy":[1], "status":"pending"}
  task_3.json  {"id":3, "blockedBy":[1], "status":"pending"}
  task_4.json  {"id":4, "blockedBy":[2,3], "status":"pending"}

任务图 (DAG):
                 +----------+
            +--> | task 2   | --+
            |    | pending  |   |
+----------+     +----------+    +--> +----------+
| task 1   |                          | task 4   |
| completed| --> +----------+    +--> | blocked  |
+----------+     | task 3   | --+     +----------+
                 | pending  |
                 +----------+

顺序:   task 1 必须先完成, 才能开始 2 和 3
并行:   task 2 和 3 可以同时执行
依赖:   task 4 要等 2 和 3 都完成
状态:   pending -> in_progress -> completed
```
<br/>

**工作原理**：

需要 4个工具 tools：任务创建、任务更新、任务列表、任务获取

提供给 LLM 使用。


```
TOOL_HANDLERS = {
    # ...base tools...
    "task_create": lambda **kw: TASKS.create(kw["subject"]),
    "task_update": lambda **kw: TASKS.update(kw["task_id"], kw.get("status")),
    "task_list":   lambda **kw: TASKS.list_all(),
    "task_get":    lambda **kw: TASKS.get(kw["task_id"]),
}
```

<br/>

| 组件    | 之前 (s06)   | 之后 (s07)                                  |
| ----- | ---------- | ----------------------------------------- |
| Tools | 5          | 8 (`task_create/update/list/get`)         |
| 规划模型  | 扁平清单 (仅内存) | 带依赖关系的任务图 (磁盘)                            |
| 关系    | 无          | `blockedBy`边                             |
| 状态追踪  | 做完没做完      | `pending`->`in_progress`->`completed` |
| 持久化   | 压缩后丢失      | 压缩和重启后存活