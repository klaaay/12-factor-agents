[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 9. 将错误压缩到上下文窗口中

这个可能有点短，但值得一提。代理的一个好处是"自我修复"——对于短任务，LLM可能会调用一个失败的工具。好的LLM有相当大的机会读取错误消息或堆栈跟踪，并在后续工具调用中找出要更改的内容。

大多数框架都实现了这一点，但你可以在不实现其他11个因子的情况下**仅实现这一点**。这里有一个例子：

```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # 我们的switch语句
  except Exception as e:
    # 如果我们得到一个错误，我们可以将其添加到上下文窗口并重试
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # 循环，或做任何其他事情来尝试恢复
```

你可能想要为特定工具调用实现一个errorCounter，以限制单个工具的尝试次数约为3次，或任何其他对你的用例有意义的逻辑。

```python
consecutive_errors = 0

while True:

  # ... 现有代码 ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # 成功！重置错误计数器
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # 做循环并重试
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # 打破循环，重置上下文窗口的部分，升级给人类，或任何其他你想做的事情
      break
  }
```

达到某些连续错误阈值可能是一个很好的地方来[升级给人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)，无论是通过模型决定还是通过控制流的确定性接管。

[![195-factor-09-errors](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)

<details>
<summary>[GIF版本](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)</summary>

![195-factor-09-errors](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)

</details>

好处：

1. **自我修复**：LLM可以读取错误消息并在后续工具调用中找出要更改的内容
2. **持久**：即使一个工具调用失败，代理也可以继续运行

我相信你会发现，如果你**过度**这样做，你的代理将开始失控，并可能一遍又一遍地重复相同的错误。

这就是[因子8 - 拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)和[因子3 - 拥有你的上下文构建](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)发挥作用的地方——你不需要只是将原始错误放回去，你可以完全重新构建它的表示方式，从上下文窗口中删除先前的事件，或任何你发现的有助于让代理回到正轨的确定性事情。

但防止错误失控的第一种方法是拥抱[因子10 - 小型专注的代理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)。

[← 拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [小型专注的代理 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)