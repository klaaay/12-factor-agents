[← 返回README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 10. 小型专注的代理

与其构建试图做所有事情的单体代理，不如构建做得好的小型专注的代理。代理只是更大、大部分确定性系统中的一个构建块。

![1a0-small-focused-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)

这里的关键洞察是关于LLM限制：任务越大越复杂，需要采取的步骤就越多，这意味着更长的上下文窗口。随着上下文的增长，LLM更有可能迷失或失去焦点。通过让代理专注于特定领域，最多3-10步，也许20步，我们保持上下文窗口可管理且LLM性能高。

> #### 随着上下文的增长，LLM更有可能迷失或失去焦点

小型专注代理的好处：

1. **可管理的上下文**：较小的上下文窗口意味着更好的LLM性能
2. **清晰的职责**：每个代理都有明确定义的范围和目的
3. **更好的可靠性**：在复杂工作流中迷失的可能性较小
4. **更容易测试**：更容易测试和验证特定功能
5. **改进的调试**：出现问题时更容易识别和修复

### 如果LLM变得更智能呢？

如果LLM变得足够智能以处理100+步工作流，我们是否仍然需要这个？

简答：是的。随着代理和LLM的改进，它们**可能**自然会扩展以能够处理更长的上下文窗口。这意味着处理更大DAG的更多部分。这种小型专注的方法确保你今天就能获得结果，同时为你准备在LLM上下文窗口变得更可靠时慢慢扩展代理范围做好准备。（如果你曾经重构过大型确定性代码库，你可能正在点头）。

[![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/cd7ed52c-046e-4d5e-bab4-57657157c82f
)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif">GIF版本</a></summary>
![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)
</details>

有意识地考虑代理的大小/范围，并且只在允许你保持质量的方式下增长，这是关键所在。正如[构建NotebookLM的团队所说](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> 我觉得始终如一，AI构建中最神奇的时刻对我来说出现在我真正、真正、真正接近模型能力边缘的时候

无论边界在哪里，如果你能找到那个边界并始终如一地做对，你就会构建神奇的体验。这里有很多护城河可以建造，但像往常一样，它们需要一些工程严谨性。

[← 压缩错误](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md) | [从任何地方触发 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)