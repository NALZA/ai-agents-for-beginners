<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "8164484c16b1ed3287ef9dae9fc437c1",
  "translation_date": "2025-07-23T08:16:58+00:00",
  "source_file": "10-ai-agents-production/README.md",
  "language_code": "zh"
}
-->
# AI代理在生产中的可观测性与评估

随着AI代理从实验性原型转向实际应用，理解其行为、监控其性能以及系统性地评估其输出的能力变得尤为重要。

## 学习目标

完成本课程后，您将了解如何：
- 掌握代理可观测性和评估的核心概念
- 提升代理性能、成本和效率的技术
- 系统性地评估AI代理的内容和方法
- 控制AI代理在生产环境中的部署成本
- 为使用AutoGen构建的代理添加监控功能

目标是让您掌握将“黑箱”代理转变为透明、可管理且可靠系统的知识。

_**注意：** 部署安全且值得信赖的AI代理非常重要。请查看[构建值得信赖的AI代理](./06-building-trustworthy-agents/README.md)课程。_

## 跟踪与跨度

可观测性工具（如[Langfuse](https://langfuse.com/)或[Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry)）通常将代理运行表示为跟踪和跨度。

- **跟踪（Trace）**：表示从开始到结束的完整代理任务（例如处理用户查询）。
- **跨度（Spans）**：是跟踪中的单个步骤（例如调用语言模型或检索数据）。

![Langfuse中的跟踪树](https://langfuse.com/images/cookbook/example-autogen-evaluation/trace-tree.png)

没有可观测性时，AI代理就像一个“黑箱”——其内部状态和推理过程不透明，难以诊断问题或优化性能。有了可观测性，代理变成了“玻璃箱”，提供了透明性，这对建立信任和确保其按预期运行至关重要。

## 为什么可观测性在生产环境中很重要

将AI代理迁移到生产环境会带来一系列新的挑战和需求。此时，可观测性不再是“可有可无”的功能，而是关键能力：

* **调试与根本原因分析**：当代理失败或产生意外输出时，可观测性工具提供的跟踪数据可以帮助定位错误来源。这在涉及多个LLM调用、工具交互和条件逻辑的复杂代理中尤为重要。
* **延迟与成本管理**：AI代理通常依赖于按令牌或调用计费的LLM和其他外部API。可观测性可以精确跟踪这些调用，帮助识别过于缓慢或昂贵的操作，从而优化提示词、选择更高效的模型或重新设计工作流，以控制运营成本并确保良好的用户体验。
* **信任、安全与合规**：在许多应用中，确保代理行为安全且符合道德规范非常重要。可观测性提供了代理行为和决策的审计记录，可用于检测和缓解问题，例如提示注入、生成有害内容或错误处理个人身份信息（PII）。例如，您可以通过查看跟踪数据了解代理为何提供某个响应或使用某个工具。
* **持续改进循环**：可观测性数据是迭代开发过程的基础。通过监控代理在实际环境中的表现，团队可以识别改进领域，收集数据以微调模型，并验证更改的影响。这创造了一个反馈循环，在线评估的生产洞察力为离线实验和优化提供信息，从而逐步提升代理性能。

## 关键指标追踪

为了监控和理解代理行为，需要追踪一系列指标和信号。尽管具体指标可能因代理的用途而异，但有些指标是普遍重要的。

以下是可观测性工具通常监控的一些常见指标：

**延迟**：代理响应的速度如何？长时间等待会对用户体验产生负面影响。您应该通过跟踪代理运行来测量任务和单个步骤的延迟。例如，一个代理在所有模型调用上花费20秒，可以通过使用更快的模型或并行运行模型调用来加速。

**成本**：每次代理运行的费用是多少？AI代理依赖于按令牌计费的LLM调用或外部API。频繁的工具使用或多次提示可能会迅速增加成本。例如，如果一个代理为了微小的质量提升调用LLM五次，您需要评估成本是否合理，或者是否可以减少调用次数或使用更便宜的模型。实时监控还可以帮助识别意外的成本激增（例如，由于错误导致的过多API循环）。

**请求错误**：代理失败了多少次请求？这可能包括API错误或工具调用失败。为了使代理在生产中更具鲁棒性，您可以设置备用方案或重试机制。例如，如果LLM提供商A宕机，您可以切换到LLM提供商B作为备选。

**用户反馈**：实施直接的用户评价可以提供宝贵的见解。这可以包括显式评分（👍/👎，⭐1-5星）或文本评论。持续的负面反馈应引起警觉，因为这表明代理未按预期工作。

**隐式用户反馈**：用户行为即使没有显式评分，也能提供间接反馈。这可能包括立即重新措辞问题、重复查询或点击重试按钮。例如，如果您发现用户反复提出相同的问题，这表明代理未按预期工作。

**准确性**：代理生成正确或期望输出的频率是多少？准确性的定义可能不同（例如，解决问题的正确性、信息检索的准确性、用户满意度）。第一步是定义代理成功的标准。您可以通过自动检查、评估分数或任务完成标签来追踪准确性。例如，将跟踪标记为“成功”或“失败”。

**自动评估指标**：您还可以设置自动评估。例如，您可以使用LLM对代理的输出进行评分，例如是否有帮助、准确或无害。还有一些开源库可以帮助您评估代理的不同方面。例如，[RAGAS](https://docs.ragas.io/)用于RAG代理，[LLM Guard](https://llm-guard.com/)用于检测有害语言或提示注入。

实际上，这些指标的组合可以最好地覆盖AI代理的健康状况。在本章的[示例笔记本](../../../10-ai-agents-production/code_samples/10_autogen_evaluation.ipynb)中，我们将展示这些指标在实际案例中的表现，但首先，我们将学习典型的评估工作流程。

## 为代理添加监控功能

为了收集跟踪数据，您需要为代码添加监控功能。目标是为代理代码添加监控，使其能够生成可被可观测性平台捕获、处理和可视化的跟踪和指标。

**OpenTelemetry (OTel)：** [OpenTelemetry](https://opentelemetry.io/)已成为LLM可观测性的行业标准。它提供了一套API、SDK和工具，用于生成、收集和导出遥测数据。

有许多封装现有代理框架的监控库，可以轻松地将OpenTelemetry跨度导出到可观测性工具。以下是使用[OpenLit监控库](https://github.com/openlit/openlit)为AutoGen代理添加监控的示例：

```python
import openlit

openlit.init(tracer = langfuse._otel_tracer, disable_batch = True)
```

本章的[示例笔记本](../../../10-ai-agents-production/code_samples/10_autogen_evaluation.ipynb)将演示如何为AutoGen代理添加监控功能。

**手动创建跨度**：尽管监控库提供了良好的基础，但在某些情况下可能需要更详细或自定义的信息。您可以手动创建跨度以添加自定义应用逻辑。更重要的是，您可以通过自定义属性（也称为标签或元数据）丰富自动或手动创建的跨度。这些属性可以包括业务特定数据、中间计算或任何可能对调试或分析有用的上下文，例如`user_id`、`session_id`或`model_version`。

以下是使用[Langfuse Python SDK](https://langfuse.com/docs/sdk/python/sdk-v3)手动创建跟踪和跨度的示例：

```python
from langfuse import get_client
 
langfuse = get_client()
 
span = langfuse.start_span(name="my-span")
 
span.end()
```

## 代理评估

可观测性为我们提供了指标，而评估是分析这些数据（并进行测试）以确定AI代理表现如何以及如何改进的过程。换句话说，一旦您拥有这些跟踪和指标，如何利用它们来评估代理并做出决策？

定期评估很重要，因为AI代理通常是非确定性的，并且可能会随着更新或模型行为的漂移而演变——如果没有评估，您无法知道您的“智能代理”是否真的在做好工作，或者是否出现了性能退化。

AI代理的评估分为两类：**离线评估**和**在线评估**。两者都很有价值，并且相辅相成。我们通常从离线评估开始，因为这是部署任何代理之前的最低必要步骤。

### 离线评估

![Langfuse中的数据集项](https://langfuse.com/images/cookbook/example-autogen-evaluation/example-dataset.png)

离线评估是在受控环境中评估代理，通常使用测试数据集，而不是实时用户查询。您使用经过精心挑选的数据集，这些数据集的预期输出或正确行为是已知的，然后让代理在这些数据集上运行。

例如，如果您构建了一个数学文字题代理，您可能会有一个[测试数据集](https://huggingface.co/datasets/gsm8k)，其中包含100个问题及其已知答案。离线评估通常在开发过程中进行（并且可以成为CI/CD管道的一部分），以检查改进或防止性能退化。其优点是**可重复，并且由于有明确的基准答案，可以获得清晰的准确性指标**。您还可以模拟用户查询，并将代理的响应与理想答案进行比较，或者使用上述的自动化指标。

离线评估的关键挑战是确保测试数据集全面且保持相关性——代理可能在固定测试集上表现良好，但在生产中遇到完全不同的查询。因此，您应该不断更新测试集，添加新的边缘案例和反映真实场景的示例。混合使用小型“冒烟测试”案例和大型评估集是有用的：小型集用于快速检查，大型集用于更广泛的性能指标。

### 在线评估

![可观测性指标概览](https://langfuse.com/images/cookbook/example-autogen-evaluation/dashboard.png)

在线评估是指在实时、真实环境中评估代理，即在生产中实际使用时进行评估。在线评估涉及监控代理在真实用户交互中的表现，并持续分析结果。

例如，您可能会跟踪成功率、用户满意度评分或其他实时流量指标。在线评估的优势在于它**捕捉到实验室环境中可能无法预见的情况**——您可以观察到模型随时间的漂移（例如，随着输入模式的变化，代理的有效性是否下降），并捕捉测试数据中未包含的意外查询或情况。它提供了代理在实际环境中表现的真实画面。

在线评估通常涉及收集隐式和显式用户反馈（如前所述），并可能运行影子测试或A/B测试（即新版本代理与旧版本并行运行以进行比较）。挑战在于为实时交互获取可靠的标签或评分可能很困难——您可能需要依赖用户反馈或下游指标（例如用户是否点击了结果）。

### 两者结合

在线评估和离线评估并不是互斥的；它们高度互补。在线监控的洞察（例如代理在某些新类型用户查询中的表现不佳）可以用于扩充和改进离线测试数据集。反之，在离线测试中表现良好的代理可以更有信心地部署并进行在线监控。

事实上，许多团队采用一个循环：

_离线评估 -> 部署 -> 在线监控 -> 收集新的失败案例 -> 添加到离线数据集 -> 优化代理 -> 重复_。

## 常见问题

在将AI代理部署到生产环境时，您可能会遇到各种挑战。以下是一些常见问题及其潜在解决方案：

| **问题**    | **潜在解决方案**   |
| ------------- | ------------------ |
| AI代理无法一致地完成任务 | - 优化提供给AI代理的提示词；明确目标。<br>- 确定是否可以将任务分解为子任务，并由多个代理分别处理。 |
| AI代理陷入连续循环 | - 确保您设置了明确的终止条件，以便代理知道何时停止流程。<br>- 对于需要推理和规划的复杂任务，使用专门用于推理任务的更大模型。 |
| AI代理的工具调用表现不佳 | - 在代理系统之外测试并验证工具的输出。 |

- 优化已定义的参数、提示和工具命名。  |
| 多代理系统表现不一致 | - 优化分配给每个代理的提示，确保它们具体且彼此区分明确。<br>- 构建一个分层系统，使用“路由”或控制器代理来确定哪个代理是正确的选择。 |

通过引入可观测性，许多问题可以更有效地被识别。我们之前讨论的跟踪和指标有助于精确定位代理工作流中问题发生的位置，从而使调试和优化更加高效。

## 成本管理

以下是一些管理将 AI 代理部署到生产环境成本的策略：

**使用小型模型：** 小型语言模型（SLM）在某些代理任务中表现良好，并能显著降低成本。如前所述，构建一个评估系统来确定并比较小型模型与大型模型的性能，是了解 SLM 在您的用例中表现如何的最佳方式。可以考虑将 SLM 用于诸如意图分类或参数提取等简单任务，而将大型模型保留用于复杂推理。

**使用路由模型：** 类似的策略是使用多样化的模型和规模。您可以使用 LLM/SLM 或无服务器函数，根据复杂性将请求路由到最合适的模型。这不仅有助于降低成本，还能确保在正确的任务上获得良好的性能。例如，将简单查询路由到更小、更快的模型，而仅在复杂推理任务中使用昂贵的大型模型。

**缓存响应：** 识别常见请求和任务，并在它们进入代理系统之前提供响应，是减少类似请求量的好方法。您甚至可以实现一个流程，使用更基础的 AI 模型来识别请求与缓存请求的相似度。这种策略可以显著降低针对常见问题或工作流的成本。

## 实践中的应用

在[本节的示例笔记本](../../../10-ai-agents-production/code_samples/10_autogen_evaluation.ipynb)中，我们将看到如何使用可观测性工具来监控和评估我们的代理。

## 上一课

[元认知设计模式](../09-metacognition/README.md)

## 下一课

[MCP](../11-mcp/README.md)

**免责声明**：  
本文档使用AI翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 进行翻译。尽管我们努力确保翻译的准确性，但请注意，自动翻译可能包含错误或不准确之处。原始语言的文档应被视为权威来源。对于重要信息，建议使用专业人工翻译。我们不对因使用此翻译而产生的任何误解或误读承担责任。