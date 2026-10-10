# AI-Models-Performance-Analysis for Response Time, CPU and Memory
AI-Models-Performance-Analysis: Choosing the Right Model for Each Task
The research question we set out to answer:
How does response time vary across models and request characteristics, and what patterns can guide model selection in a multi-model workflow?
The answer in brief
Part of the question: What the data shows
Across models:	Average response time ranges from 11.8 s (Qwen) to 71.8 s (Gemma), a sixfold gap. Confidence intervals and bootstrap estimates both support this ranking, and the slower models also vary more from call to call.
Across request characteristics:	Answer length is strongly associated with response time (r = 0.91 across 460 completed calls; 0.82 to 0.98 within each model). The structure search found no direct link from input type, trigger or bot instance to response time.
Patterns for model selection:	Qwen is the leading candidate for short, time-sensitive tasks and MiniCPM for longer answers, subject to answer-quality testing. The expected length of an answer is a promising input for routing rules.
