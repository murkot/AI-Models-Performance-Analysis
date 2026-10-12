# Response Time in a Multi-Model VLM Workflow

**Author:** Olena McMurtrey

This project models how long four vision-language models (VLMs) take to answer requests in an automated multi-model workflow. It asks which request and workflow characteristics explain response time, and what those patterns mean for choosing a model.

## Research Question

> How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?

## Hypotheses

| Test | Null hypothesis (H0) | Alternative hypothesis (HA) |
| --- | --- | --- |
| Main effects | No candidate request or workflow characteristic is associated with response time. | At least one candidate characteristic is associated with response time. |
| Interaction | The relationship between answer length and response time is the same across models. | The relationship differs across models. |
| Model reduction | The predictors removed in Candidate Model 3 jointly contribute nothing. | At least one removed predictor contributes. |

## Data

`Olena_Log_Template_Proposed.csv` is a log of 600 requests sent through an n8n workflow (two bot instances, triggered by Telegram, Google Sheets or webhook) to four locally hosted VLMs. The 460 completed runs, 115 per model, form the analysis sample.

| Model | Developer |
| --- | --- |
| `gemma-3-12b-it` | Google DeepMind |
| `pixtral-12b` | Mistral AI |
| `minicpm-o-2_6` | OpenBMB |
| `qwen2-vl-7b-instruct` | Alibaba Qwen team |

**Response variable:** `model_time_seconds`, the time the model took to answer.

**Candidate predictors:** model, input type, trigger, workflow instance, answer length, request length, image attached, average CPU, average GPU and average memory. Columns derived from response time (such as throughput) are excluded to avoid leakage.

## Methods

All models are ordinary least squares regressions fitted with `statsmodels`, following the textbook's Python Web Appendix (Agresti & Kateri, 2022).

| Model | Specification |
| --- | --- |
| Candidate Model 1 | All candidate predictors, main effects only |
| Candidate Model 2 | Model 1 plus the model by answer-length interaction |
| Candidate Model 3 | Model 2 with five non-contributing terms removed |
| Candidate Model 4 | Model 3's structure with log-transformed response time |

The notebook also covers a leakage check, a correlation matrix, nested-model F tests, a comparison of five specifications (R-squared, adjusted R-squared, AIC), multicollinearity, residual diagnostics and influential observations.

## Key Results

| Hypothesis | Test | Statistic | Decision |
| --- | --- | --- | --- |
| Main effects | Global F, Candidate Model 3 | F(21, 438) = 490.39, p < .001 | H0 rejected |
| Interaction | Nested F, Model 1 against Model 2 | F(3, 433) = 136.18, p < .001 | H0 rejected |
| Reduction | Nested F, Model 3 against Model 2 | F(5, 433) = 0.2284, p = .950 | H0 not rejected |

- **Candidate Model 3 is the preferred model:** adjusted R-squared 0.9572 and the lowest AIC (3289.50) of the five specifications, with 22 parameters.
- **Characteristics that mattered:** model choice, input type, trigger type, answer length and request length. Answer length dominated, with a sum of squares about nine times that of model choice.
- **Characteristics that did not:** workflow instance, image attached, CPU, GPU and memory. Response time is governed by how much text is produced, not by machine contention.
- **Per-model generation rates** (seconds per character, Candidate Model 3):

| Model | Seconds per character |
| --- | --- |
| gemma-3-12b-it | 0.0303 |
| pixtral-12b | 0.0263 |
| qwen2-vl-7b-instruct | 0.0131 |
| minicpm-o-2_6 | 0.0107 |

- **Diagnostics:** Candidate Model 3's residual spread grows with the fitted value. Candidate Model 4 (log response) makes the spread nearly constant across fitted values, at the cost of expressing rates as percentage changes instead of seconds.

## Repository Contents

| File | Description |
| --- | --- |
| `Olena_Draft2_Draft_VLM_Full_Candidate_Model_polished.ipynb` | The analysis notebook |
| `Olena_Log_Template_Proposed.csv` | The workflow log (add it to the repository if it may be shared) |
| `README.md` | This file |

## How to Run

1. Install Python 3 and the required libraries:

   ```bash
   pip install pandas numpy matplotlib statsmodels scipy jupyter
   ```

2. Place `Olena_Log_Template_Proposed.csv` in the same folder as the notebook.

3. In the Data Preparation cell, change the file path to the local file name:

   ```python
   vlm_log = pd.read_csv('Olena_Log_Template_Proposed.csv')
   ```

4. Open the notebook and choose **Kernel → Restart & Run All**.

## Reproducibility Note

One run (index 304) is the only run with its trigger category, which gives it a leverage of 1 in Candidate Model 3. Its Cook's distance is therefore numerically unstable and can come out as missing or as a very large value depending on the machine. As a result, the influential-observation section may identify a different run than the one reported (run 369) when the notebook is re-run elsewhere. All other results are unaffected.
