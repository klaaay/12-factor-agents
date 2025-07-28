[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 3. 拥有你的上下文窗口

你不需要使用基于消息的标准格式来向LLM传达上下文。

> #### 在任何给定点，你在agent中对LLM的输入是"到目前为止发生了什么，下一步是什么"

<!-- todo 语法高亮 -->
<!-- ![130-own-your-context-building](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

一切都是上下文工程。[LLMs是无状态函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，将输入转换为输出。为了获得最佳输出，你需要给它们最佳输入。

创建优秀的上下文意味着：

- 你给模型的提示和指令
- 你检索的任何文档或外部数据（例如RAG）
- 任何过去的状态、工具调用、结果或其他历史记录
- 任何来自相关但独立历史/对话的过去消息或事件（记忆）
- 关于输出何种结构化数据的指令

![image](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)

### 关于上下文工程

本指南都是关于从今天的模型中获得尽可能多的东西。值得注意的是没有提到：

- 模型参数的变化，如temperature、top_p、frequency_penalty、presence_penalty等
- 训练你自己的补全或嵌入模型
- 微调现有模型

同样，我不知道将上下文交给LLM的最佳方式是什么，但我知道你想要尝试一切的灵活性。

#### 标准与自定义上下文格式

大多数LLM客户端使用基于消息的标准格式，像这样：

```yaml
[
  {
    "role": "system",
    "content": "你是一个有用的助手..."
  },
  {
    "role": "user",
    "content": "你能部署后端吗？"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

虽然这对大多数用例都很好，但如果你想真正从今天的LLM中获得**最多**，你需要以最节省标记和注意力的方式将上下文放入LLM。

作为基于消息的标准格式的替代方案，你可以构建针对你的用例优化的自己的上下文格式。例如，你可以使用自定义对象并将它们打包/分散到一个或多个用户、系统、助手或工具消息中，视情况而定。

这里是一个将整个上下文窗口放入单个用户消息的例子：
```yaml

[
  {
    "role": "system",
    "content": "你是一个有用的助手..."
  },
  {
    "role": "user",
    "content": |
            到目前为止发生的一切：
        
        <slack_message>
            来自：@alex
            频道：#deployments
            文本：你能部署后端吗？
        </slack_message>
        
        <list_git_tags>
            意图："list_git_tags"
        </list_git_tags>
        
        <list_git_tags_result>
            标签：
              - 名称："v1.2.3"
                提交："abc123"
                日期："2024-03-15T10:00:00Z"
              - 名称："v1.2.2"
                提交："def456"
                日期："2024-03-14T15:30:00Z"
              - 名称："v1.2.1"
                提交："ghi789"
                日期："2024-03-13T09:15:00Z"
        </list_git_tags_result>
        
        下一步是什么？
    }
]
```

模型可能会通过你提供的工具模式推断你在问它`下一步是什么`，但将其包含在你的提示模板中永远不会有什么坏处。

### 代码示例

我们可以用类似这样的东西构建：

```python

class Thread:
  events: List[Event]

class Event:
  # 可以只使用字符串，或者可以明确 - 由你决定
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |  
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

#### 示例上下文窗口

这里是如何用这种方法构建上下文窗口的示例：

**初始Slack请求：**
```xml
<slack_message>
    来自：@alex
    频道：#deployments
    文本：你能将最新后端部署到生产环境吗？
</slack_message>
```

**列出Git标签后：**
```xml
<slack_message>
    来自：@alex
    频道：#deployments
    文本：你能将最新后端部署到生产环境吗？
    线程：[]
</slack_message>

<list_git_tags>
    意图："list_git_tags"
</list_git_tags>

<list_git_tags_result>
    标签：
      - 名称："v1.2.3"
        提交："abc123"
        日期："2024-03-15T10:00:00Z"
      - 名称："v1.2.2"
        提交："def456"
        日期："2024-03-14T15:30:00Z"
      - 名称："v1.2.1"
        提交："ghi789"
        日期："2024-03-13T09:15:00Z"
</list_git_tags_result>
```

**错误和恢复后：**
```xml
<slack_message>
    来自：@alex
    频道：#deployments
    文本：你能将最新后端部署到生产环境吗？
    线程：[]
</slack_message>

<deploy_backend>
    意图："deploy_backend"
    标签："v1.2.3"
    环境："production"
</deploy_backend>

<error>
    运行deploy_backend时出错：连接到部署服务失败
</error>

```