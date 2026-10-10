# AI Model Performance Analysis
# Exploring Response Time, CPU and Memory

## Research Question

*How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?*

## Answer in Brief

| Part of the Question | What the Data Shows |
|---|---|
| **Across models** | Average response time ranges from **11.8 s for Qwen** to **71.8 s for Gemma**, a roughly sixfold gap. Confidence intervals and bootstrap estimates support this ranking, and the slower models also show greater variability from call to call. |
| **Across request characteristics** | Answer length is strongly associated with response time (**r = 0.91** across 460 completed calls and **r = 0.82 to 0.98** within individual models). The structure search found no direct link from input type, trigger, or bot instance to response time. |
| **Patterns for model selection** | Qwen is the leading candidate for short, time-sensitive tasks, while MiniCPM may be better suited for longer answers, subject to answer-quality testing. Expected answer length is a promising input for model-routing rules. |

## Key Findings

- **6x gap in average response time:** 11.8 s for the fastest model, Qwen, compared with 71.8 s for the slowest model, Gemma.
- **r = 0.91 correlation** between answer length and response time, the strongest request-level association identified.
- **About 2x faster per 1,000 characters of answer:** MiniCPM and Qwen compared with Pixtral and Gemma.
- **5.07x estimated processing work:** a typical Gemma call required approximately 5.07 times the CPU-seconds of a typical Qwen call.
- **19.7 CPU-hours estimated savings per 1,000 calls** moved from Gemma to Qwen.
- **33 s vs. 246 s longest observed call:** Qwen's longest completed call was 33 seconds, compared with 246 seconds for Gemma.

## What We Tested

We benchmarked four locally hosted vision-language models using **600 logged requests** sent through an n8n workflow operating across two bot instances.

Of those requests, **460 completed successfully** and were included in the performance analysis.

Each request was logged with:

- Response time
- Average CPU utilization during the call
- Average memory utilization during the call
- Total workflow time
- Bot instance
- Model response text
- Request characteristics

The completed benchmark was balanced across models. Each model completed exactly **115 runs** and received the same mix of input types:

- Text only
- Image only
- Text plus image
- Mixed input

## Models Evaluated

| Model | Developer | Size |
|---|---|---|
| **qwen2-vl-7b-instruct** | Alibaba Qwen Team | 7B parameters |
| **minicpm-o-2_6** | OpenBMB | 8B parameters, multimodal |
| **pixtral-12b** | Mistral AI | 12B decoder plus 400M vision encoder |
| **gemma-3-12b-it** | Google DeepMind | 12B parameters, 128,000-token context |

## Reliability

Reliability did not meaningfully separate the four models in this dataset.

Each model had:

- **115 completed calls**
- **3 logged failures**
- **32 requests that did not execute**

The available logs do not record why individual requests failed or did not execute. Because the counts were identical across all four models, completion rates do not provide evidence for favoring one model over another.

## Main Interpretation

The results suggest that **model choice has a substantial effect on response time**, while answer length is one of the strongest observable predictors of how long a request will take.

Qwen delivered the lowest average response time and the shortest maximum observed response time, making it a strong candidate for workflows where latency is important.

MiniCPM also performed efficiently when response time was considered relative to answer length, suggesting that it may be useful when longer responses are expected.

Gemma showed substantially higher response time and estimated processing work in this benchmark. However, performance alone should not determine model selection. A production routing strategy should also evaluate answer quality, task type, accuracy, and other model-specific capabilities.

## Practical Implication

A multi-model workflow could potentially route requests based on expected task complexity and response length.

For example:

```text
Short / latency-sensitive request
        ↓
      Qwen

Longer expected response
        ↓
    MiniCPM

Specialized task requiring other model capabilities
        ↓
 Pixtral or Gemma
```

These routing recommendations should be validated against **answer-quality metrics** before being used as final model-selection rules.

## Dataset Scope

The conclusions in this repository describe the behavior of these four models under the tested local infrastructure, workflow configuration, request mix, and hardware environment.

The results should therefore be interpreted as **benchmark-specific findings**, rather than universal performance rankings for the models.
