[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 8. 拥有你的控制流

如果你拥有你的控制流，你可以做很多有趣的事情。

![180-control-flow](https://github.com/humanlayer/12-factor-agents/blob/main/img/180-control-flow.png)

构建对你特定用例有意义的自己的控制结构。具体来说，某些类型的工具调用可能是跳出循环并等待来自人类或另一个长时间运行任务（如训练管道）的响应的原因。你可能还想包含自定义实现：

- 工具调用结果的总结或缓存
- 结构化输出的LLM-as-judge
- 上下文窗口压缩或其他[内存管理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- 日志记录、跟踪和指标
- 客户端速率限制
- 持久睡眠/暂停/"等待事件"

下面的例子显示了三种可能的控制流模式：

- request_clarification：模型要求更多信息，中断循环并等待来自人类的响应
- fetch_git_tags：模型要求git标签列表，获取标签，附加到上下文窗口，并直接传递回模型
- deploy_backend：模型要求部署后端，这是高风险的事情，所以中断循环并等待人工批准

```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))
    
    # 为了清晰而内联 - 实际上你可以
    # 将其放入方法中，使用异常进行控制流，或任何你想要的
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # 异步步骤 - 中断循环，我们稍后会得到webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # 同步步骤 - 将新上下文传递给LLM以确定下一步
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # 异步步骤 - 中断循环，我们稍后会得到webhook
      break
```

这种模式允许你根据需要中断和恢复agent的流程，创建更自然的对话和工作流。

**例子** - 我对每个AI框架的第一大功能请求是我们需要能够中断
一个工作中的agent并稍后恢复，**特别是在工具**选择**和工具**调用**之间的时刻。

没有这种级别的可恢复性/粒度，就没有办法在工具调用运行之前审查/批准它，这意味着
你被迫要么：

1. 在等待长时间运行的事情完成时将任务暂停在内存中（想想`while...sleep`），如果进程被中断，从头开始重启它
2. 将agent限制为仅低风险、低风险的调用，如研究和总结
3. 给agent访问权限去做更大、更有用的事情，只是yolo希望它不会搞砸


你可能注意到这与[因子5 - 统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)和[因子6 - 使用简单API启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)密切相关，但可以独立实现。

[← 使用工具联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [紧凑错误 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md)