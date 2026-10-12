# Dataset: AI Chat Models Performance in n8n Notification Workflows

This dataset provides structured performance logs of four open-source AI chat models 
(Minicpm-o-2_6, Gemma-3-12B-IT, Pixtral-12B, Qwen2-VL-7B-Instruct) deployed locally 
through LM Studio and orchestrated via n8n.

## File Contents
- `data/Log_Template.xlsx` : Main dataset (Excel format).
- `data/Log_Template.csv` : CSV version of the dataset for easy analysis.
- `workflow/n8n_workflow.json` : Importable n8n workflow design used in this study.

## Dataset Columns
- **Test ID** : Unique identifier for each run
- **Scenario Name** : Describes the task (e.g., tone analysis, scene description)
- **Purpose** : The evaluation goal of the scenario
- **Input Type** : Text-only, Text+Image, or Image-only
- **How to Start** : Trigger method (Telegram, Webhook)
- **Model** : AI chat model tested
- **Response Time (s)** : Model-only latency
- **Workflow Time (s)** : End-to-end latency (including orchestration)
- **Success Status** : Whether the run was successful
- **CPU/GPU/Memory per second** : Resource usage logs per second during execution

## Scenarios
The dataset covers three input categories:
1. **Text-only**
   - Simple factual queries
   - Medium reasoning tasks
   - Complex analytical prompts
2. **Text+Image**
   - Scene description
   - Visual reasoning (image + text input)
3. **Image-only**
   - Object detection
   - Caption generation

## Usage
- Analyze performance differences across models and modalities.
- Evaluate workflow orchestration overhead.
- Compare efficiency trade-offs (response time vs. CPU/GPU/memory).

## License
CC BY 4.0 – You are free to share and adapt with proper attribution.

## Citation
If you use this dataset, please cite the companion paper and dataset DOI.


