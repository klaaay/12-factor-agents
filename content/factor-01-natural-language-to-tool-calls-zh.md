[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 1. 自然语言到工具调用

Agent构建中最常见的模式之一是将自然语言转换为结构化工具调用。这是一个强大的模式，允许你构建能够推理任务并执行它们的agent。

![110-natural-language-tool-calls](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)

当原子化应用时，这种模式就是将类似这样的短语进行简单翻译

> 你能为Terri创建一个750美元的付款链接，用于赞助二月的AI修补匠聚会吗？

转换为描述Stripe API调用的结构化对象，如

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "嘿Jeff - 见下文获取二月AI修补匠聚会的付款链接"
    }
  }
}
```

**注意**：实际上Stripe API更复杂一些，一个[真正做这个的agent](https://github.com/dexhorthy/mailcrew)（[视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)）会列出客户、列出产品、列出价格等来构建具有正确ID的有效载荷，或者将这些ID包含在提示/上下文窗口中（我们将在下面看到它们实际上是同一回事！）

从那里开始，确定性代码可以获取有效载荷并对其进行处理。（更多内容见[因子3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)）

```python
# LLM接受自然语言并返回结构化对象
nextStep = await llm.determineNextStep(
  """
  为Jeff创建一个750美元的付款链接
  用于赞助二月的AI修补匠聚会
  """
  )

# 根据其函数处理结构化输出
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # 或任何你想要的，见下文
elif nextStep.function == 'something_else':
    # ... 更多情况
    pass
else:  # 模型调用了我们不知道的工具
    # 做其他事情
    pass
```

**注意**：虽然一个完整的agent会随后接收API调用结果并循环处理，最终返回类似这样的内容

> 我已成功为Terri创建了750美元的付款链接，用于赞助二月的AI修补匠聚会。这是链接：https://buy.stripe.com/test_1234567890

**相反**，我们实际上将跳过这一步，把它留给另一个因子，你可能想要也可能不想要合并（由你决定！）

[← 我们如何走到这里](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md) | [拥有你的提示 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)