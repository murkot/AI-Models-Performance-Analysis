# Data Cleaning Log

**AAI-500 Team 1: AI Model Performance Analysis** (draft for review)

This log records how the raw benchmark log was cleaned for analysis. Every row and every column is kept. The cleaning only standardizes names, labels and data types, so each row of the clean file is the same row of the raw file.

## Research Question

Examine how response time varies across models and request characteristics, and identify patterns that may be useful for model selection in a multi-model workflow.

## Source Files

| File | Role |
| --- | --- |
| `Log_Template.csv` | Working file for cleaning |
| `Log_Template.xlsx` | Used to verify the CSV |
| `Readme.md` | Source of the input-type definitions |

## Summary of Results

| Item | Raw file | Clean file |
| --- | --- | --- |
| Rows | 600 | 600 (all kept) |
| Columns | 30 | 30 (none added, none removed) |
| `success` values | Yes 460, No 12, blank 128 | Yes 460, No 12, Not executed 128 |
| Completed rows per model | 115 | 115 (balanced) |
| `input_type` labels | 5 | 4 (Image + Text merged into Text + Image) |
| Scenario name spellings | Hyphen and en dash mixed | Hyphen only |

## Cleaning Steps

| Step | What was done | Why |
| --- | --- | --- |
| 1 | Loaded the CSV and compared every cell with the XLSX | Both hold the same 600 rows and 30 columns. Only 2 cells differ, one in each HH:mm:ss.ms column, where the CSV shows a shortened time (for example `0:04:34` vs `00:04:34.018`). The CSV is used as the working file. |
| 2 | Renamed all 30 columns to snake_case | Names with spaces, slashes and brackets break Python code. The two HH:mm:ss.ms columns are kept (team decision). |
| 3 | Standardized text: en dash to hyphen in 596 scenario names; `viaTelegram` to `via Telegram` in 4 rows; stray quote marks removed from `workflow_end_time`; spaces trimmed in six text columns | One scenario written two ways would be counted as two scenarios. The quote marks stopped the end time from being read as a date. |
| 4 | Merged `Image + Text` into `Text + Image` in 8 rows | The source README defines three input types and lists "image + text input" under Text+Image. Both labels use the same command with one prompt and one image. Test ID still tells the two groups apart. |
| 5 | Set the 128 blank `success` values to `Not executed` | These rows have no timing, and the notes show the requests never reached a model. They are kept for success-rate comparisons. |
| 6 | Converted the 4 timestamp columns from text to UTC dates | Text timestamps cannot be sorted or grouped reliably. No values failed to parse beyond the 140 blank rows. Durations are not recalculated; the logged seconds are kept. |
| 7 | Checked completeness, ranges and balance (no changes) | No missing model times among completed rows. Response time ranges from 2 s to 246 s, with no zero or negative values. Each model has 115 completed rows. |

### Things to know about the clean file

- **The 140 rows without timing or resource values** are exactly the 12 failed and 128 not-executed rows. Filter on `success == 'Yes'` for any latency or resource statistic.
- **124 of the 128 not-executed rows are exact copies of another row.** They are kept because each one stands for a separate planned request.
- **The merged `Text + Image` label covers two different tests.** Tests 5-8 are single requests; Test 18 is a parallel load test with different prompts and images. Test 18's former `Image + Text` rows are slower (median 43.5 s vs 21.0 s), but load and prompt also differ, so input order cannot be tested with this data.
- **In batch runs, `workflow_time_seconds` is a running total**, not a per-request time.

## Analysis Guide

| Analysis | Measure columns | Group by | Row filter |
| --- | --- | --- | --- |
| Response time by model | `model_time_seconds` | `model` | `success == 'Yes'` |
| CPU usage by model | `cpu_avg_pct`, `cpu_per_second_pct` | `model` | `success == 'Yes'` |
| Memory usage by model | `memory_avg_mb`, `memory_per_second_mb` | `model` | `success == 'Yes'` |
| Response time by request characteristics | `model_time_seconds` | `input_type`, `test_id` or `scenario_name`, `how_to_start` | `success == 'Yes'` |
| GPU usage by request characteristics | `gpu_avg_pct`, `gpu_per_second_pct` | `input_type`, `test_id` | `success == 'Yes'` |
| Workflow latency by request characteristics | `workflow_time_seconds` | `input_type`, `test_id` | `success == 'Yes'`; one row per workflow |
| Success / failure rate | `success` | `model`, `test_id` | All rows |

## Data Dictionary

| Column | Role | Meaning |
| --- | --- | --- |
| `chat_id` | Identifier | Telegram chat; one constant value |
| `test_id` | Design | Test number 1-18 |
| `scenario_name` | Design | Scenario, dashes standardized |
| `scenario_purpose` | Design | Purpose of the scenario |
| `how_to_start` | Design / request characteristic | Trigger method |
| `input_type` | Request characteristic | Text Only, Text + Image, Image Only, Mixed |
| `model` | Main grouping | One of four models |
| `image_input` | Reference | Image cell, shortened in the source |
| `text_input` | Reference | Prompt or bot command |
| `model_output` | Reference | Model answer |
| `workflow_name` | Design | CSBI Bot 1 or CSBI Bot 2 |
| `workflow_start_time` | Context | Workflow start, UTC date |
| `workflow_end_time` | Context | Workflow end, UTC date |
| `workflow_time_hhmmss` | Reference | Workflow time as logged, HH:mm:ss.ms text |
| `workflow_time_seconds` | Measure: workflow latency | As logged, whole seconds; running total in batch runs |
| `model_start_time` | Context | Model call start, UTC date |
| `model_end_time` | Context | Model call end, UTC date |
| `model_response_time_hhmmss` | Reference | Model response time as logged, HH:mm:ss.ms text |
| `model_time_seconds` | **Measure: response time (main)** | As logged, whole seconds |
| `success` | Filter | Yes / No / Not executed |
| `error_message` | Reference | Error text, if any |
| `cpu_per_second_pct` | Measure: CPU (raw list) | One value per second |
| `cpu_avg_pct` | Measure: CPU | Average CPU during the call |
| `gpu_per_second_pct` | Measure: GPU (raw list) | One value per second |
| `gpu_avg_pct` | Measure: GPU | Average GPU during the call |
| `memory_per_second_mb` | Measure: memory (raw list) | One value per second |
| `memory_avg_mb` | Measure: memory | Average memory during the call |
| `throughput_per_minute` | Measure (mirrors response time) | About 60 / `model_time_seconds` |
| `throughput_per_second` | Measure (mirrors response time) | Per-minute value / 60 |
| `notes` | Reference | Tester notes |

<details>
<summary><strong>Original column names</strong></summary>

| Raw column | Clean column |
| --- | --- |
| Chat ID | `chat_id` |
| Test ID | `test_id` |
| Scenario Name | `scenario_name` |
| Description/Purpose | `scenario_purpose` |
| How To Start Testing | `how_to_start` |
| Input Type | `input_type` |
| Model | `model` |
| Image Input (Base64 format) | `image_input` |
| Text Input | `text_input` |
| Model Result (Output) | `model_output` |
| Workflow Name | `workflow_name` |
| Workflow Start Time | `workflow_start_time` |
| Workflow End Time | `workflow_end_time` |
| Workflow Time (HH:mm:ss.ms) | `workflow_time_hhmmss` |
| Workflow Time (in second) | `workflow_time_seconds` |
| Model Start Time | `model_start_time` |
| Model End Time | `model_end_time` |
| Model Response Time (HH:mm:ss.ms) | `model_response_time_hhmmss` |
| Model Time (in second) | `model_time_seconds` |
| Success (Y/N) | `success` |
| Error Message (If Any) | `error_message` |
| CPU Usage per second (%) | `cpu_per_second_pct` |
| Avg CPU Usage (%) | `cpu_avg_pct` |
| GPU Usage per second (%) | `gpu_per_second_pct` |
| Avg GPU Usage (%) | `gpu_avg_pct` |
| Memory Usage per second (MB) | `memory_per_second_mb` |
| Avg Memory Usage (MB) | `memory_avg_mb` |
| Throughput (req/min) - per min | `throughput_per_minute` |
| Throughput (req/sec) - per sec | `throughput_per_second` |
| Notes / Observations | `notes` |

</details>
