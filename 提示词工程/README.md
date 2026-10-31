# 提示词工程学习路线

这份路线聚焦如何设计、在代码中使用和评估提示词，并用 AV2T-Gen 的环境条件举例。

## 1. 先理解提示词

提示词是输入给模型的任务、背景、约束或示例。提示词工程是围绕具体任务设计输入、测试结果、分析错误并迭代的过程。

推荐顺序：

1. [Prompt Engineering Guide：Introduction](https://www.promptingguide.ai/introduction)：看 Basics of Prompting、Prompt Elements、Examples of Prompts；弄清任务、上下文、输入、输出格式。
2. [Google：Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies)：看清晰指令、提供示例和迭代设计；用同一任务对比不同写法。
3. [Anthropic：Prompting best practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-templates-and-variables)：看清晰直接、示例、结构化输入；注意不同模型的接口和推荐方式不同。
4. [OpenAI：Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering)：看消息角色、示例、补充上下文和评估；结合当前产品文档选用接口。

## 2. 练习：从模糊到明确

任务：根据用户提供的论文摘要，提取任务、方法和局限。

版本 A：
> 帮我看一下论文。

版本 B：
> 根据下面的摘要，提取研究任务、核心方法和作者提到的局限。用三条列表回答；摘要没有提供的信息写“未说明”。摘要：{abstract}

版本 C：
> [任务] 根据摘要提取研究信息。
> [输出] 用 JSON 输出 task、method、limitations 三个字段；limitations 是数组。
> [约束] 仅依据摘要；没有提到的内容使用 null。
> [摘要] {abstract}

比较三版时固定同一批摘要和模型设置，检查事实是否来自摘要、字段是否完整、有无编造。更长的提示不保证更好。

## 3. 代码中如何构造提示

以下是普通 Python 字符串示意，不依赖某一厂商 API：

```python
def build_prompt(abstract: str) -> str:
    # 提示模板固定；每条输入数据替换变量。
    return f"""根据下方摘要提取研究任务、方法和局限。
    仅依据摘要；未说明的信息标记为“未说明”。
    输出三个字段：任务、方法、局限。

    摘要：
    {abstract}
    """

abstracts = ["第一篇论文的摘要……", "第二篇论文的摘要……"]
prompts = [build_prompt(abstract) for abstract in abstracts]

# 将每条 prompt 传给所选模型的官方 SDK。
# 根据任务记录输入、模型版本、输出及人工评分，便于比较不同模板。
```

## 4. 设计小型评估

- 准备 10–20 条有代表性的样本，包含信息完整、信息缺失和难例。
- 先定义评分标准：事实正确、格式合格、遗漏情况。
- 固定模型、输入和参数；比较模板 A 与 B。
- 阅读错误例子，修改一处提示，再在同一批样本上评估。
- 在未用于调提示的新样本上复查，避免只适配这几条例子。

## 5. 与 AV2T-Gen 的关系

可以用模板将温度、湿度、时间等整理为环境文本，再编码为生成条件。下面只是教学例子，不是论文原模板：

```python
def environment_prompt(temp_c, humidity):
    return f"Temperature: {temp_c} °C; relative humidity: {humidity}%."
```

语言模型的问答提示与图像扩散模型的条件文本有共同点：都设计输入来影响模型行为；但它们的模型结构、输入接口和评价方法不同。单靠文本提示不能保证物理温度准确。

## 6. 进阶论文（选读）

- [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165)：看 in-context learning 和 few-shot 示例，理解“把示例放在提示里”。
- [Pre-train, Prompt, and Predict](https://arxiv.org/abs/2107.13586)：看提示学习的研究脉络；论文的部分方法属于早期模型，注意与当前模型实践区分。
- [A Systematic Survey of Prompt Engineering in Large Language Models](https://arxiv.org/abs/2402.07927)：需要系统分类时再读。
