[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 11. 从任何地方触发，在用户所在的地方相遇

如果你在等待[humanlayer](https://humanlayer.dev)的推销，你等到了。如果你在做[因子6 - 使用简单API启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)和[因子7 - 使用工具调用联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)，你就准备好整合这个因子了。

![1b0-trigger-from-anywhere](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

使用户能够从slack、电子邮件、短信或他们想要的任何其他渠道触发代理。使代理能够通过相同的渠道响应。

好处：

- **在用户所在的地方相遇**：这帮助你构建感觉像真正人类的AI应用程序，或者至少，数字同事
- **外循环代理**：使代理能够被非人类触发，例如事件、cron、中断等。它们可能工作5、20、90分钟，但当它们到达关键点时，它们可以联系人类寻求帮助、反馈或批准
- **高风险工具**：如果你能够快速引入各种人类，你可以让代理访问更高风险的操作，如发送外部电子邮件、更新生产数据等。保持明确的标准让你在代理[执行更大更好的事情](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)时获得可审计性和信心

[← 小型专注的代理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [无状态化简器 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)