[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 7. 使用工具调用联系人类

默认情况下，LLM API依赖于一个根本的高风险标记选择：我们是返回明文内容，还是返回结构化数据？

![170-contact-humans-with-tools](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

你在第一个标记的选择上放了很多权重，在`东京天气`的情况下，它是

> "the"

但在`fetch_weather`的情况下，它是一些特殊标记来表示JSON对象的开始。

> |JSON>

你可能通过让LLM**始终**输出json，然后用一些自然语言标记如`request_human_input`或`done_for_now`声明其意图来获得更好的结果（与"适当"的工具如`check_weather_in_city`相反）。

同样，你可能不会从这个中获得任何性能提升，但你应该实验，并确保你有自由尝试奇怪的东西以获得最佳结果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# 人类交互的工具定义
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# agent循环中的示例用法
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # 中断循环并等待响应通过线程ID返回
else:
  # ... 其他情况
```

稍后，你可能会从处理slack、电子邮件、短信或其他事件的系统接收webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... 为了简洁而简化，你可能不想在这里阻塞web工作线程
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - 循环或中断或任何你想要的

  return {"status": "ok"}
```

上面包括了来自[因子5 - 统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)、[因子8 - 拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)、[因子3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)和[因子4 - 工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)以及其他几个的模式。

如果我们使用来自[因子3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)的XML-y格式，我们的上下文窗口在几轮后可能看起来像这样：

```xml

（为简洁而删减）

<slack_message>
    来自：@alex
    频道：#deployments
    文本：你能将后端v1.2.3部署到生产环境吗？
    线程：[]
</slack_message>

<request_human_input>
    意图："request_human_input"
    问题："你想继续将v1.2.3部署到生产环境吗？"
    上下文："这是将影响实时用户的生产部署。"
    选项：{
        紧急程度："high"
        格式："yes_no"
    }
</request_human_input>

<human_response>
    响应："是的请继续"
    已批准：true
    时间戳："2024-03-15T10:30:00Z"
    用户："alex@company.com"
</human_response>

<deploy_backend>
    意图："deploy_backend"
    标签："v1.2.3"
    环境："production"
</deploy_backend>

<deploy_backend_result>
    状态："success"
    消息："到生产环境的v1.2.3部署成功完成。"
    时间戳："2024-03-15T10:30:00Z"
</deploy_backend_result>
```

好处：

1. **清晰指令**：不同类型人类联系的工具允许来自LLM的更多特异性
2. **内环与外环**：启用传统chatGPT风格界面**之外**的agent工作流，其中控制流和上下文初始化可能是`Agent->Human`而不是`Human->Agent`（想想，由cron或事件启动的agent）
3. **多个人类访问**：可以通过结构化事件轻松跟踪和协调来自不同人类的输入
4. **多Agent**：简单的抽象可以轻松扩展以支持`Agent->Agent`请求和响应
5. **持久**：与[因子6 - 使用简单API启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)结合，这使得持久、可靠和可检查的多人工作流成为可能

[更多关于外环Agent的内容在这里](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

![175-outer-loop-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

与[因子11 - 从任何地方触发，在用户所在的地方与他们见面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)配合得很好

[← 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) | [拥有你的控制流 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)