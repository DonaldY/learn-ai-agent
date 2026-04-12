> "One loop & Bash is all you need"-- 一个工具 + 一个循环 = 一个 Agent。

<img src="./img/s01.png">

伪代码如下：

```py
while True:
    response = client.messages.create(messages=messages, tools=tools)
    if response.stop_reason != "tool_use":
        break
    for tool_call in response.content:
        result = execute_tool(tool_call.name, tool_call.input)
        messages.append(result)
```

现在理解的 Agent 为：Agent = LLM + tools

-   tools：工具，是文件读写、Shell、网络、数据库、浏览器等，是手脚，去感知获取周遭信息。


<br/>


**变更如下**：

| 组件           | 之前  | 之后                          |
| ------------ | --- | --------------------------- |
| Agent loop   | (无) | `while True` + stop_reason  |
| Tools        | (无) | `bash` (单一工具)               |
| Messages     | (无) | 累积式消息列表                     |
| Control flow | (无) | `stop_reason != "tool_use"`


**小结：**
1. Agent 核心是感知、决策、行动、反馈的稳定循环，控制流基本不变，新能力主要通过工具扩展、提示结构调整和状态外化实现。
2. Harness，也就是验收基线、执行边界、反馈信号、回退手段，往往比模型本身更决定系统能否收敛，高质量自动化验证和清晰目标缺一不可。
3. 上下文工程的重点是防 Context Rot，通过分层管理常驻信息、按需知识、运行时信息和记忆，再配合滑动窗口、LLM 摘要、工具结果替换和 Skills 延迟加载，才能把信号质量稳定住。