### 因子13 - 预取你可能需要的所有上下文

如果你的模型有很大概率会调用工具X，不要浪费token往返告诉模型去获取它，也就是说，不要像这样的伪提示：

```jinja
当查看部署时，你可能会想要获取已发布git标签的列表，
这样你就可以用它来部署到生产环境。

到目前为止发生了什么：

{{ thread.events }}

下一步是什么？

用以下意图之一的JSON格式回答：

{
  intent: 'deploy_backend_to_prod',
  tag: string
} OR {
  intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}
```

你的代码看起来像

```python
thread = {"events": [initial_message]}
next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'list_git_tags':
      tags = await fetch_git_tags()
      thread["events"].append({
        type: 'list_git_tags',
        data: tags,
      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

你不妨直接获取标签并将它们包含在上下文窗口中，像这样：

```diff
- 当查看部署时，你可能会想要获取已发布git标签的列表，
- 这样你就可以用它来部署到生产环境。

+ 当前的git标签是：

+ {{ git_tags }}


到目前为止发生了什么：

{{ thread.events }}

下一步是什么？

用以下意图之一的JSON格式回答：

{
  intent: 'deploy_backend_to_prod',
  tag: string
- } OR {
-   intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}

```

你的代码看起来像

```diff
thread = {"events": [initial_message]}
+ git_tags = await fetch_git_tags()

- next_step = await determine_next_step(thread)
+ next_step = await determine_next_step(thread, git_tags)

while True:
  switch next_step.intent:
-    case 'list_git_tags':
-      tags = await fetch_git_tags()
-      thread["events"].append({
-        type: 'list_git_tags',
-        data: tags,
-      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

甚至可以将标签包含在线程中并从提示模板中删除特定参数：

```diff
thread = {"events": [initial_message]}
+ # 添加请求
+ thread["events"].append({
+  "type": 'list_git_tags',
+ })

git_tags = await fetch_git_tags()

+ # 添加结果
+ thread["events"].append({
+  "type": 'list_git_tags_result',
+  "data": git_tags,
+ })

- next_step = await determine_next_step(thread, git_tags)
+ next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append(deploy_result)
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

总的来说：

> #### 如果你已经知道你希望模型调用哪些工具，只需确定性地调用它们，让模型做困难的部分，即弄清楚如何使用它们的输出

再次，AI工程就是关于[上下文工程](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)。

[← 无状态化简器](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) | [进一步阅读 →](https://github.com/humanlayer/12-factor-agents/blob/main/README.md#related-resources)