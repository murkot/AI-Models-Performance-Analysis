# AI Model Performance Analysis

## Choosing the Right Model for Each Task

**Response Time, Compute Cost, and Routing Across Four Vision-Language Models**

## Research Question

*How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?*

## Answer in Brief

| Part of the Question | What the Data Shows |
|---|---|
| **Across models** | Average response time ranges from **11.8 s for Qwen** to **71.8 s for Gemma**, a roughly sixfold gap. Confidence intervals and bootstrap estimates support the same ranking, and slower models also vary more from call to call. |
| **Across request characteristics** | Answer length is strongly associated with response time (**r = 0.91** across 460 completed calls; **r = 0.82 to 0.98** within individual models). The structure search found no direct link from input type, trigger, or workflow instance to response time. |
| **Patterns for model selection** | Qwen is the leading candidate for short, time-sensitive tasks, while MiniCPM is a strong candidate for longer answers, subject to answer-quality testing. Expected answer length is a promising input for routing rules. |

## Key Findings

- **Sixfold gap in average response time:** 11.8 s for Qwen compared with 71.8 s for Gemma.
- **r = 0.91 correlation** between answer length and response time, the strongest request-level association found.
- **About 2x faster per 1,000 characters:** MiniCPM and Qwen compared with Pixtral and Gemma.
- **5.07x estimated processing work:** a typical Gemma call required about 5.07 times the CPU-seconds of a typical Qwen call.
- **19.7 CPU-hours estimated savings per 1,000 calls** moved from Gemma to Qwen.
- **33 s vs. 246 s longest observed call:** Qwen compared with Gemma.

## What We Tested

We benchmarked four locally hosted vision-language models on **600 logged requests** sent through an n8n workflow operating across two bot instances.

Of those requests, **460 completed** and were included in the performance analysis.

Each completed request included:

- Response time
- Average CPU utilization
- Average memory utilization
- Total workflow time
- Bot instance
- Model response text
- Request characteristics

The completed benchmark was balanced. Each model completed exactly **115 runs** and received the same mix of input types:

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

## Results at a Glance

| Model | Median Call | Mean Call | Longest Call | Median Answer Length | Seconds per 1,000 Characters | Median CPU-Seconds |
|---|---:|---:|---:|---:|---:|---:|
| **Qwen** | 7 s | 11.8 s | 33 s | 471 | 15.0 | 9.8 |
| **MiniCPM** | 17 s | 17.9 s | 59 s | 1,173 | 13.0 | 16.6 |
| **Pixtral** | 31 s | 42.5 s | 122 s | 1,050 | 28.0 | 34.9 |
| **Gemma** | 46 s | 71.8 s | 246 s | 1,194 | 32.8 | 49.9 |

## Response Time Across Models

Average response time ranged from **11.8 seconds for Qwen** to **71.8 seconds for Gemma**. The variability also increased as models became slower.

The 95% confidence intervals for mean response time did not overlap across the four models, and bootstrap estimates reproduced the same ranking.

| Model | 95% CI for Mean Response Time | 95% CI for Median Response Time |
|---|---|---|
| **Qwen** | 10.3 to 13.3 s | 6 to 12 s |
| **MiniCPM** | 15.9 to 19.9 s | 15 to 17 s |
| **Pixtral** | 36.5 to 48.4 s | 25 to 37 s |
| **Gemma** | 61.4 to 82.3 s | 39 to 52 s |

Response times were right-skewed for every model. This means most calls completed relatively quickly, while a smaller number took much longer.

For user expectations, the **median** is useful for describing a typical call. For capacity planning and timeout decisions, the **mean and upper tail** are also important.

## Answer Length and Response Time

Answer length was strongly associated with response time.

Across all 460 completed calls:

**r = 0.91**

The relationship also remained strong within each model:

| Model | Correlation with Response Time | Median Answer Length |
|---|---:|---:|
| **Qwen** | 0.95 | 471 characters |
| **MiniCPM** | 0.82 | 1,173 characters |
| **Pixtral** | 0.98 | 1,050 characters |
| **Gemma** | 0.95 | 1,194 characters |

The models also formed two approximate pace tiers:

- **MiniCPM:** 13.0 s per 1,000 characters
- **Qwen:** 15.0 s per 1,000 characters
- **Pixtral:** 28.0 s per 1,000 characters
- **Gemma:** 32.8 s per 1,000 characters

This suggests that expected answer length may be useful as an input to model-routing rules.

## Request Characteristics

Other logged request characteristics showed no direct link to response time in the exploratory structure search:

- Input type
- Workflow instance
- Trigger

The learned dependency structure selected **model → response time** as the only direct link into response time.

This structure is exploratory and should not be interpreted as proof of causation.

## Compute Cost

Average CPU percentage and memory usage did not differ detectably across models, but slower models held the processor for longer.

To estimate total processing work, CPU-seconds were calculated as:

```text
CPU-seconds = (CPU percentage / 100) × response time in seconds
```

On this measure, the models separated clearly.

A typical Gemma call required about **5.07 times** the estimated CPU-seconds of a typical Qwen call.

### Estimated Savings From Routing Calls to Qwen

| Model Replaced by Qwen | Median CPU-Seconds Relative to Qwen | Estimated CPU-Hours Saved per 1,000 Calls |
|---|---:|---:|
| **MiniCPM** | 1.68x | 1.8 |
| **Pixtral** | 3.55x | 10.1 |
| **Gemma** | 5.07x | 19.7 |

These are planning estimates rather than direct monetary or energy measurements.

## Model Selection Recommendations

| Task Type | Candidate Model | Rationale |
|---|---|---|
| **Short, time-sensitive tasks** | Qwen | Lowest median and mean response time, shortest observed maximum response time, and lowest estimated compute. |
| **Longer answers** | MiniCPM | Lowest response seconds per 1,000 characters and strong performance on longer outputs. |
| **Tasks with specialist quality needs** | Pixtral or Gemma | This benchmark measured speed and compute, not answer quality. These models should be selected only when task-specific quality testing justifies the additional latency and compute. |

## Reliability

Reliability did not separate the models.

Each model had:

- **115 completed calls**
- **3 logged failures**
- **32 requests that did not execute**

The logs do not record why calls failed or did not execute. Because the counts were identical across models, completion rates do not provide evidence for favoring one model over another.

## Methods

All statistical results were based on the **460 completed runs**, with **115 completed runs per model**.

Methods included:

- Descriptive statistics and box plots
- Histograms and normal quantile plots
- Skewness analysis
- Gamma distribution comparison
- Standard errors and 95% confidence intervals
- Bootstrap confidence intervals
- Tukey pairwise comparisons
- Correlation analysis
- Estimated CPU-seconds
- Response seconds per 1,000 characters
- Exploratory dependency structure learning using hill climbing and the Bayesian Information Criterion

## Main Conclusion

Model choice had a substantial effect on both latency and estimated processing work in this benchmark.

**Qwen** was the strongest candidate for short, latency-sensitive work.

**MiniCPM** was especially competitive for longer answers because it produced longer outputs at a relatively low time per 1,000 characters.

**Pixtral and Gemma** required substantially more time and estimated processing work, so their use should be justified by task-specific answer quality or specialist capabilities.

A practical multi-model workflow could use expected answer length and task type as routing inputs, but these routing rules should be validated with **answer-quality testing** before production use.

## Scope and Limitations

These findings describe the behavior of the four tested models under this specific local infrastructure, hardware configuration, n8n workflow, and request mix.

The results should therefore be interpreted as **benchmark-specific findings**, not universal performance rankings for these models.
