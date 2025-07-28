[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

## 更长的版本：我们如何走到这里

### 你不必听我的

无论你是 AI Agent 的新手还是像我一样脾气暴躁的老兵，我都会试图说服你抛弃对 AI Agent 的大部分认知，退后一步，从第一性原理重新思考它们。（如果你几周前没有注意到 OpenAI responses 的发布，剧透警告：将更多 Agent 逻辑推送到 API 后面并不是答案）

## Agent 是软件，以及软件的简要历史

让我们谈谈我们是如何走到这里的

### 60 年前

我们将大量讨论有向图（DGs）及其无环朋友 DAGs。我首先指出...嗯...软件是一个有向图。我们过去用流程图表示程序是有原因的。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 年前

大约 20 年前，我们开始看到 DAG 编排器变得流行。我们谈论的是经典如[Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，一些前身，以及一些较新的如 ([dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/))。这些遵循相同的图模式，增加了可观察性、模块化、重试、管理等好处。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 年前

当 ML 模型开始变得足够好用时，我们开始看到 DAGs 中散布着 ML 模型。你可能会想象诸如"将此列中的文本总结为新列"或"按严重程度或情感分类支持问题"等步骤。

![020-dags-with-ml](https://github.com/humanlayer/12-factor-agents/blob/main/img/020-dags-with-ml.png)

但归根结底，这仍然是大部分相同的经典确定性软件。

### Agent 的承诺

我不是第一个[这样说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但当我开始学习 agent 时，我最大的收获是你可以扔掉 DAG。而不是软件工程师编码每个步骤和边缘情况，你可以给 agent 一个目标和一组转换：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

让 LLM 实时做出决定来找出路径

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是你编写更少的软件，你只需给 LLM 图的"边缘"，让它找出节点。你可以从错误中恢复，你可以编写更少的代码，你可能会发现 LLM 找到问题的创新解决方案。

### Agent作为循环

换句话说，你有这个由3个步骤组成的循环：

1. LLM确定工作流中的下一步，输出结构化json（"工具调用"）
2. 确定性代码执行工具调用
3. 结果被附加到上下文窗口
4. 重复直到下一步被确定为"完成"

```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

我们的初始上下文只是起始事件（可能是用户消息，可能是cron触发，可能是webhook等），
我们要求llm选择下一步（工具）或确定我们完成了。

这是一个多步示例：

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)]

</details>

生成的"物化"DAG看起来像这样：

![027-agent-loop-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### 这种"循环直到解决"模式的问题

这种模式的最大问题：

- Agent在上下文窗口太长时迷失方向 - 它们反复尝试相同的错误方法
- 这就是全部，但这足以削弱方法

即使你还没有手工制作agent，你可能也已经在使用agent编码工具时看到了这个长上下文问题。它们过一段时间后就会迷失方向，你需要开始新的聊天。

我甚至会提出一些我无意中听到很多的东西，你可能已经形成了自己的直觉：

> ### **即使模型支持越来越长的上下文窗口，你总会用小的、集中的提示和上下文获得更好的结果**

我交谈过的大多数构建者**把"工具调用循环"的想法推到一边**，当他们意识到超过10-20轮就会变成LLM无法恢复的大混乱时。即使agent 90%的时间都是正确的，这也远未达到"足够好以至于可以交到客户手中"。你能想象一个网页应用在10%的页面加载时崩溃吗？

**更新2025-06-09** - 我真的很喜欢[@swyx](https://x.com/swyx/status/1932125643384455237)的表述方式：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="Screenshot 2025-07-02 at 11 50 50 AM" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### 真正有效的东西 - 微agent

我在野外**确实**看到过很多的一件事是将agent模式洒入更广泛的确定性DAG中。

![micro-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

你可能会问 - "在这种情况下为什么要使用agent？" - 我们很快就会讲到，但基本上，让语言模型管理范围明确的任务集使得无需陷入上下文错误循环即可轻松整合实时人类反馈。（[因子1](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)、[因子3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)、[因子7](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)）。

> #### 让语言模型管理范围明确的任务集使得无需陷入上下文错误循环即可轻松整合实时人类反馈

### 真实生活中的微agent

这里是一个确定性代码如何运行一个负责处理人工参与步骤部署的微agent的示例。

![029-deploybot-high-level](https://github.com/humanlayer/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **人类**将PR合并到GitHub主分支
* **确定性代码**部署到暂存环境
* **确定性代码**针对暂存运行端到端（e2e）测试
* **确定性代码**交给agent进行生产部署，初始上下文："将SHA 4af9ec0部署到生产环境"
* **Agent**调用`deploy_frontend_to_prod(4af9ec0)`
* **确定性代码**请求人工批准此操作
* **人类**拒绝操作并反馈"你能先部署后端吗？"
* **Agent**调用`deploy_backend_to_prod(4af9ec0)`
* **确定性代码**请求人工批准此操作
* **人类**批准操作
* **确定性代码**执行后端部署
* **Agent**调用`deploy_frontend_to_prod(4af9ec0)`
* **确定性代码**请求人工批准此操作
* **人类**批准操作
* **确定性代码**执行前端部署
* **Agent**确定任务成功完成，我们完成了！
* **确定性代码**针对生产运行端到端测试
* **确定性代码**任务完成，或传递给回滚agent审查失败并可能回滚

[![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif">GIF版本</a></summary>

![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)]

</details>

这个示例基于我们[在Humanlayer发布以管理我们部署的](https://github.com/got-agents/agents/tree/main/deploybot-ts)真实[OSS agent](https://github.com/got-agents/agents/tree/main/deploybot-ts) - 这是上周我与它的真实对话：

![035-deploybot-conversation](https://github.com/humanlayer/12-factor-agents/blob/main/img/035-deploybot-conversation.png)

我们没有给这个agent大量的工具或任务。LLM中的主要价值是解析人类的明文反馈并提出更新的行动方案。我们尽可能隔离任务和上下文，以保持LLM专注于小的、5-10步工作流。

这是另一个[更经典的支持/聊天机器人演示](https://x.com/chainlit_io/status/1858613325921480922)。

### 那么agent到底是什么？

- **提示** - 告诉LLM如何表现，以及它有哪些"工具"可用。提示的输出是一个描述工作流中下一步（"工具调用"或"函数调用"）的JSON对象。（[因子2](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)）
- **switch语句** - 基于LLM返回的JSON，决定如何处理它。（[因子8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)的一部分）
- **累积上下文** - 存储已发生的步骤列表及其结果（[因子3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)）
- **for循环** - 直到LLM发出某种"终端"工具调用（或明文响应），将switch语句的结果添加到上下文窗口并要求LLM选择下一步。（[因子8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)）

![040-4-components](https://github.com/humanlayer/12-factor-agents/blob/main/img/040-4-components.png)

在"deploybot"示例中，我们通过拥有控制流和上下文累积获得了几个好处：

- 在我们的**switch语句**和**for循环**中，我们可以劫持控制流以暂停人工输入或等待长时间运行任务的完成
- 我们可以轻松序列化**上下文**窗口以暂停+恢复
- 在我们的**提示**中，我们可以优化我们如何传递指令和"到目前为止发生了什么"给LLM


[第二部分](https://github.com/humanlayer/12-factor-agents/blob/main/README.md#12-factor-agents)将**形式化这些模式**，以便它们可以应用于向任何软件项目添加令人印象深刻的AI功能，而无需完全投入传统实现/定义"AI agent"。


[因子1 - 自然语言到工具调用 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)