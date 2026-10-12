# AI Model Performance Analysis
# Exploring Response Time, CPU and Memory

**Author:** Olena McMurtrey

## Research Question

*How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?*

## Answer in Brief

| Part of the Question | What the Data Shows |
|---|---|
| **Across models** | Average response time ranges from **11.8 s for Qwen** to **71.8 s for Gemma**, a roughly sixfold gap. Confidence intervals and bootstrap estimates support this ranking, and the slower models also show greater variability from call to call. |
| **Across request characteristics** | Answer length is strongly associated with response time (**r = 0.91** across 460 completed calls and **r = 0.82 to 0.98** within individual models). In the regression models, answer length is the dominant predictor and its effect differs by model. The structure search found no direct link from input type, trigger, or bot instance to response time, though the regression detected smaller effects of input type and trigger after adjustment. |
| **Patterns for model selection** | Qwen is the leading candidate for short, time-sensitive tasks, while MiniCPM may be better suited for longer answers, subject to answer-quality testing. Expected answer length is a promising input for model-routing rules. |

## Key Findings

- **6x gap in average response time:** 11.8 s for the fastest model, Qwen, compared with 71.8 s for the slowest model, Gemma.
- **r = 0.91 correlation** between answer length and response time, the strongest request-level association identified.
- **About 2x faster per 1,000 characters of answer:** MiniCPM and Qwen compared with Pixtral and Gemma.
- **5.07x estimated processing work:** a typical Gemma call required approximately 5.07 times the CPU-seconds of a typical Qwen call.
- **19.7 CPU-hours estimated savings per 1,000 calls** moved from Gemma to Qwen.
- **33 s vs. 246 s longest observed call:** Qwen's longest completed call was 33 seconds, compared with 246 seconds for Gemma.
- **95.9% of the variation in response time explained** by the preferred regression model, which includes a model by answer-length interaction.

## Repository Structure

```text
.
├── README.md
├── data/
│   └── Olena_Log_Template_Proposed.csv
└── notebooks/
    ├── Olena_FinalProject_EDA_polished.ipynb
    └── Olena_Draft2_Draft_VLM_Full_Candidate_Model_polished.ipynb
```

| Notebook | What it covers |
|---|---|
| **Part 1: Response time, CPU and memory** (`Olena_FinalProject_EDA_polished.ipynb`) | Response time, CPU and memory by model; distribution shape; confidence and bootstrap intervals; CPU-seconds; answer length; learned dependency structure |
| **Part 2: Regression modeling** (`Olena_Draft2_Draft_VLM_Full_Candidate_Model_polished.ipynb`) | Four candidate linear models of response time, nested-model F tests, model comparison, diagnostics, influential observations and a log-transformed model |

## What We Tested

We benchmarked four locally hosted vision-language models (VLMs) using **600 logged requests** sent through an n8n workflow operating across two bot instances.

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

## Part 1: Response Time, CPU and Memory

### Response time

| Model | Mean (s) | Median (s) | Standard deviation (s) | Longest call (s) |
|---|---|---|---|---|
| qwen2-vl-7b-instruct | 11.77 | 7 | 8.20 | 33 |
| minicpm-o-2_6 | 17.87 | 17 | 10.94 | 59 |
| pixtral-12b | 42.45 | 31 | 32.35 | 122 |
| gemma-3-12b-it | 71.84 | 46 | 56.72 | 246 |

- All four 95% confidence intervals for the mean are disjoint, under both the t distribution and the bootstrap, so the ranking is not benchmark noise.
- Every model is right-skewed (skewness 0.89 to 1.16), with the mean above the median. The median describes a typical wait; the mean drives totals across many calls.

### CPU

- Average CPU is about **115% of one core for every model**. Tukey comparisons found no difference between any pair (adjusted p-values 0.989 to 1.000).
- CPU percentage is unrelated to response time (r = 0.045). The bot instance, not the model, moves the CPU reading (median 118.7% on Bot 1 against 72.4% on Bot 2).
- Model choice still shapes CPU cost through duration: a typical Gemma call uses **5.07x the CPU-seconds** of a typical Qwen call.

### Memory

- Average memory is about **7,700 MB for every model**. Tukey comparisons found no difference between any pair (adjusted p-values 0.893 to 1.000).
- Memory is unrelated to the model call (r = 0.004) but correlates at about **0.50 with total workflow time** for every model. Memory follows the workflow, not the model.

## Part 2: Regression Modeling

The modeling notebook fits four candidate linear models of response time, following the textbook's Python Web Appendix (Agresti & Kateri, 2022).

### Hypotheses

| Test | Null hypothesis (H0) | Alternative hypothesis (HA) |
|---|---|---|
| Main effects | No candidate request or workflow characteristic is associated with response time. | At least one candidate characteristic is associated with response time. |
| Interaction | The relationship between answer length and response time is the same across models. | The relationship differs across models. |
| Model reduction | The predictors removed in Candidate Model 3 jointly contribute nothing. | At least one removed predictor contributes. |

### Candidate models

| Model | Specification |
|---|---|
| Candidate Model 1 | All candidate predictors, main effects only |
| Candidate Model 2 | Model 1 plus the model by answer-length interaction |
| Candidate Model 3 | Model 2 with five non-contributing terms removed |
| Candidate Model 4 | Model 3's structure with log-transformed response time |

Candidate predictors: model, input type, trigger, bot instance, answer length, request length, image attached, average CPU, average GPU and average memory. Columns derived from response time, such as throughput, are excluded to avoid leakage.

### Results

| Hypothesis | Test | Statistic | Decision |
|---|---|---|---|
| Main effects | Global F, Candidate Model 3 | F(21, 438) = 490.39, p < .001 | H0 rejected |
| Interaction | Nested F, Model 1 against Model 2 | F(3, 433) = 136.18, p < .001 | H0 rejected |
| Reduction | Nested F, Model 3 against Model 2 | F(5, 433) = 0.2284, p = .950 | H0 not rejected |

- **Preferred model:** Candidate Model 3, with adjusted R-squared 0.9572 and the lowest AIC (3289.50) of the five specifications compared.
- **Predictors that mattered:** model choice, input type, trigger type, answer length and request length. Answer length dominated, with a sum of squares about nine times that of model choice.
- **Predictors that did not:** bot instance, image attached, CPU, GPU and memory. This matches Part 1: response time is governed by how much text is produced, not by machine contention.
- **Estimated generation rates**, holding the other predictors constant:

| Model | Seconds per character |
|---|---|
| gemma-3-12b-it | 0.0303 |
| pixtral-12b | 0.0263 |
| qwen2-vl-7b-instruct | 0.0131 |
| minicpm-o-2_6 | 0.0107 |

These rates agree with Part 1's finding that MiniCPM and Qwen write about twice as fast per character as Pixtral and Gemma.

- **Diagnostics:** Candidate Model 3's residual spread grows with the fitted value. Candidate Model 4 (log response time) makes the spread nearly constant, at the cost of expressing rates as percentage changes instead of seconds.

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

## How to Run

1. Install Python 3 and the required libraries:

   ```bash
   pip install pandas numpy matplotlib seaborn scipy statsmodels pgmpy pygraphviz jupyter
   ```

   `pgmpy` and `pygraphviz` are needed only for the dependency-structure section of Part 1. `pygraphviz` also requires Graphviz to be installed on your system.

2. In each notebook, point the data file path to the `data/` folder:

   ```python
   pd.read_csv('../data/Olena_Log_Template_Proposed.csv')
   ```

3. Open a notebook and choose **Kernel → Restart & Run All**.

## Reproducibility Note

In Part 2, one run (index 304) is the only run with its trigger category, which gives it a leverage of 1 in Candidate Model 3. Its Cook's distance is therefore numerically unstable and can come out as missing or as a very large value depending on the machine. As a result, the influential-observation section may identify a different run than the one reported (run 369) when the notebook is re-run elsewhere. All other results are unaffected.

## Dataset Scope

The conclusions in this repository describe the behavior of these four models under the tested local infrastructure, workflow configuration, request mix, and hardware environment.

The results should therefore be interpreted as **benchmark-specific findings**, rather than universal performance rankings for the models.

## Reference

Agresti, A., & Kateri, M. (2022). *Foundations of statistics for data scientists: With R and Python*. Chapman & Hall/CRC.
# AI Model Performance Analysis
# Exploring Response Time, CPU and Memory

**Author:** Olena McMurtrey
**Course:** AAI-500 Probability and Statistics for AI, University of San Diego

## Research Question

*How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?*

## Answer in Brief

| Part of the Question | What the Data Shows |
|---|---|
| **Across models** | Average response time ranges from **11.8 s for Qwen** to **71.8 s for Gemma**, a roughly sixfold gap. Confidence intervals and bootstrap estimates support this ranking, and the slower models also show greater variability from call to call. |
| **Across request characteristics** | Answer length is strongly associated with response time (**r = 0.91** across 460 completed calls and **r = 0.82 to 0.98** within individual models). In the regression models, answer length is the dominant predictor and its effect differs by model. The structure search found no direct link from input type, trigger, or bot instance to response time, though the regression detected smaller effects of input type and trigger after adjustment. |
| **Patterns for model selection** | Qwen is the leading candidate for short, time-sensitive tasks, while MiniCPM may be better suited for longer answers, subject to answer-quality testing. Expected answer length is a promising input for model-routing rules. |

## Key Findings

- **6x gap in average response time:** 11.8 s for the fastest model, Qwen, compared with 71.8 s for the slowest model, Gemma.
- **r = 0.91 correlation** between answer length and response time, the strongest request-level association identified.
- **About 2x faster per 1,000 characters of answer:** MiniCPM and Qwen compared with Pixtral and Gemma.
- **5.07x estimated processing work:** a typical Gemma call required approximately 5.07 times the CPU-seconds of a typical Qwen call.
- **19.7 CPU-hours estimated savings per 1,000 calls** moved from Gemma to Qwen.
- **33 s vs. 246 s longest observed call:** Qwen's longest completed call was 33 seconds, compared with 246 seconds for Gemma.
- **95.9% of the variation in response time explained** by the preferred regression model, which includes a model by answer-length interaction.

## Repository Structure

```text
.
├── README.md
├── data/
│   └── Olena_Log_Template_Proposed.csv
└── notebooks/
    ├── Olena_FinalProject_EDA_polished.ipynb
    └── Olena_Draft2_Draft_VLM_Full_Candidate_Model_polished.ipynb
```

| Notebook | What it covers |
|---|---|
| **Part 1: Response time, CPU and memory** (`Olena_FinalProject_EDA_polished.ipynb`) | Response time, CPU and memory by model; distribution shape; confidence and bootstrap intervals; CPU-seconds; answer length; learned dependency structure |
| **Part 2: Regression modeling** (`Olena_Draft2_Draft_VLM_Full_Candidate_Model_polished.ipynb`) | Four candidate linear models of response time, nested-model F tests, model comparison, diagnostics, influential observations and a log-transformed model |

## What We Tested

We benchmarked four locally hosted vision-language models (VLMs) using **600 logged requests** sent through an n8n workflow operating across two bot instances.

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

## Part 1: Response Time, CPU and Memory

### Response time

| Model | Mean (s) | Median (s) | Standard deviation (s) | Longest call (s) |
|---|---|---|---|---|
| qwen2-vl-7b-instruct | 11.77 | 7 | 8.20 | 33 |
| minicpm-o-2_6 | 17.87 | 17 | 10.94 | 59 |
| pixtral-12b | 42.45 | 31 | 32.35 | 122 |
| gemma-3-12b-it | 71.84 | 46 | 56.72 | 246 |

- All four 95% confidence intervals for the mean are disjoint, under both the t distribution and the bootstrap, so the ranking is not benchmark noise.
- Every model is right-skewed (skewness 0.89 to 1.16), with the mean above the median. The median describes a typical wait; the mean drives totals across many calls.

### CPU

- Average CPU is about **115% of one core for every model**. Tukey comparisons found no difference between any pair (adjusted p-values 0.989 to 1.000).
- CPU percentage is unrelated to response time (r = 0.045). The bot instance, not the model, moves the CPU reading (median 118.7% on Bot 1 against 72.4% on Bot 2).
- Model choice still shapes CPU cost through duration: a typical Gemma call uses **5.07x the CPU-seconds** of a typical Qwen call.

### Memory

- Average memory is about **7,700 MB for every model**. Tukey comparisons found no difference between any pair (adjusted p-values 0.893 to 1.000).
- Memory is unrelated to the model call (r = 0.004) but correlates at about **0.50 with total workflow time** for every model. Memory follows the workflow, not the model.

## Part 2: Regression Modeling

The modeling notebook fits four candidate linear models of response time, following the textbook's Python Web Appendix (Agresti & Kateri, 2022).

### Hypotheses

| Test | Null hypothesis (H0) | Alternative hypothesis (HA) |
|---|---|---|
| Main effects | No candidate request or workflow characteristic is associated with response time. | At least one candidate characteristic is associated with response time. |
| Interaction | The relationship between answer length and response time is the same across models. | The relationship differs across models. |
| Model reduction | The predictors removed in Candidate Model 3 jointly contribute nothing. | At least one removed predictor contributes. |

### Candidate models

| Model | Specification |
|---|---|
| Candidate Model 1 | All candidate predictors, main effects only |
| Candidate Model 2 | Model 1 plus the model by answer-length interaction |
| Candidate Model 3 | Model 2 with five non-contributing terms removed |
| Candidate Model 4 | Model 3's structure with log-transformed response time |

Candidate predictors: model, input type, trigger, bot instance, answer length, request length, image attached, average CPU, average GPU and average memory. Columns derived from response time, such as throughput, are excluded to avoid leakage.

### Results

| Hypothesis | Test | Statistic | Decision |
|---|---|---|---|
| Main effects | Global F, Candidate Model 3 | F(21, 438) = 490.39, p < .001 | H0 rejected |
| Interaction | Nested F, Model 1 against Model 2 | F(3, 433) = 136.18, p < .001 | H0 rejected |
| Reduction | Nested F, Model 3 against Model 2 | F(5, 433) = 0.2284, p = .950 | H0 not rejected |

- **Preferred model:** Candidate Model 3, with adjusted R-squared 0.9572 and the lowest AIC (3289.50) of the five specifications compared.
- **Predictors that mattered:** model choice, input type, trigger type, answer length and request length. Answer length dominated, with a sum of squares about nine times that of model choice.
- **Predictors that did not:** bot instance, image attached, CPU, GPU and memory. This matches Part 1: response time is governed by how much text is produced, not by machine contention.
- **Estimated generation rates**, holding the other predictors constant:

| Model | Seconds per character |
|---|---|
| gemma-3-12b-it | 0.0303 |
| pixtral-12b | 0.0263 |
| qwen2-vl-7b-instruct | 0.0131 |
| minicpm-o-2_6 | 0.0107 |

These rates agree with Part 1's finding that MiniCPM and Qwen write about twice as fast per character as Pixtral and Gemma.

- **Diagnostics:** Candidate Model 3's residual spread grows with the fitted value. Candidate Model 4 (log response time) makes the spread nearly constant, at the cost of expressing rates as percentage changes instead of seconds.

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

## How to Run

1. Install Python 3 and the required libraries:

   ```bash
   pip install pandas numpy matplotlib seaborn scipy statsmodels pgmpy pygraphviz jupyter
   ```

   `pgmpy` and `pygraphviz` are needed only for the dependency-structure section of Part 1. `pygraphviz` also requires Graphviz to be installed on your system.

2. In each notebook, point the data file path to the `data/` folder:

   ```python
   pd.read_csv('../data/Olena_Log_Template_Proposed.csv')
   ```

3. Open a notebook and choose **Kernel → Restart & Run All**.

## Reproducibility Note

In Part 2, one run (index 304) is the only run with its trigger category, which gives it a leverage of 1 in Candidate Model 3. Its Cook's distance is therefore numerically unstable and can come out as missing or as a very large value depending on the machine. As a result, the influential-observation section may identify a different run than the one reported (run 369) when the notebook is re-run elsewhere. All other results are unaffected.

## Dataset Scope

The conclusions in this repository describe the behavior of these four models under the tested local infrastructure, workflow configuration, request mix, and hardware environment.

The results should therefore be interpreted as **benchmark-specific findings**, rather than universal performance rankings for the models.

## Reference

Agresti, A., & Kateri, M. (2022). *Foundations of statistics for data scientists: With R and Python*. Chapman & Hall/CRC.
