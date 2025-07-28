[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 2. 拥有你的提示

不要将你的提示工程外包给框架。

![120-own-your-prompts](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)

顺便说一句，[这远非新颖的建议：](https://hamel.dev/blog/posts/prompt/)

![image](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

一些框架提供这样的"黑盒"方法：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

这对于引入一些顶尖的提示工程来帮助你入门很棒，但通常很难调整和/或反向工程以将完全正确的标记放入你的模型中。

相反，拥有你的提示并将它们视为一等代码：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}
    
    You are a helpful assistant that manages deployments for frontend and backend systems.
    You work diligently to ensure safe and successful deployments by following best practices
    and proper deployment procedures.
    
    Before deploying any system, you should check:
    - The deployment environment (staging vs production)
    - The correct tag/version to deploy
    - The current system status
    
    You can use tools like deploy_backend, deploy_frontend, and check_deployment_status
    to manage deployments. For sensitive deployments, use request_approval to get
    human verification.
    
    Always think about what to do first, like:
    - Check current deployment status
    - Verify the deployment tag exists
    - Request approval if needed
    - Deploy to staging before production
    - Monitor deployment progress
    
    {{ _.role("user") }}

    {{ thread }}
    
    What should the next step be?
  "#
}
```

（上面的例子使用[BAML](https://github.com/boundaryml/baml)生成提示，但你可以使用任何你想要的提示工程工具，甚至只是手动模板化）

如果签名看起来有点奇怪，我们将在[因子4 - 工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)中讨论

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

拥有你的提示的主要好处：

1. **完全控制**：编写你的agent需要的准确指令，没有黑盒抽象
2. **测试和评估**：为你的提示构建测试和评估，就像你对任何其他代码所做的那样
3. **迭代**：根据真实世界表现快速修改提示
4. **透明度**：准确了解你的agent正在使用哪些指令
5. **角色黑客**：利用支持用户/助手角色非标准使用的API - 例如，现已弃用的OpenAI"补全"API的非聊天风格。这包括一些所谓的"模型煤气灯"技术

记住：你的提示是你的应用逻辑和LLM之间的主要接口。

完全控制你的提示为你提供了生产级agent所需的灵活性和提示控制。

我不知道什么是最好的提示，但我知道你想要尝试一切的灵活性。

[← 自然语言到工具调用](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [拥有你的上下文窗口 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)