[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 5. 统一执行状态和业务状态

即使在AI世界之外，许多基础设施系统也试图将"执行状态"与"业务状态"分开。对于AI应用，这可能涉及复杂的抽象来跟踪诸如当前步骤、下一步、等待状态、重试计数等内容。这种分离创造了复杂性，可能值得，但可能对你的用例来说是过度设计。

一如既往，由你决定什么对你的应用程序是正确的。但不要认为你**必须**单独管理它们。

更清楚地说：

- **执行状态**：当前步骤、下一步、等待状态、重试计数等
- **业务状态**：agent工作流中到目前为止发生了什么（例如OpenAI消息列表、工具调用和结果列表等）

如果可能，**简化** - 尽可能统一它们。

[![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)](https://github.com/user-attachments/assets/e5a851db-f58f-43d8-8b0c-1926c99fc68d)


<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif">GIF版本</a></summary>

![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)]

</details>

实际上，你可以设计你的应用程序，以便你可以从上下文窗口推断所有执行状态。在许多情况下，执行状态（当前步骤、等待状态等）只是到目前为止发生的事情的元数据。

你可能有一些不能放入上下文窗口的东西，如会话ID、密码上下文等，但你的目标应该是最小化这些东西。通过拥抱[因子3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)，你可以控制实际进入LLM的内容

这种方法有几个好处：

1. **简单性**：所有状态的一个真实来源
2. **序列化**：线程可以简单地序列化/反序列化
3. **调试**：整个历史在一个地方可见
4. **灵活性**：通过添加新事件类型轻松添加新状态
5. **恢复**：可以通过加载线程从任何点恢复
6. **分叉**：可以通过将线程的子集复制到新的上下文/状态ID中来在任何点分叉线程
7. **人工接口和可观察性**：可以简单地将线程转换为人类可读的markdown或丰富的Web应用UI

[← 工具是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [启动/暂停/恢复 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)