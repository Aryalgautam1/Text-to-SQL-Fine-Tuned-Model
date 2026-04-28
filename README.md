# Text-to-SQL Fine-Tuned Model

Fine-tuning an open-source 7B language model to generate accurate SQL queries from natural language, with the goal of beating GPT-4o on the BIRD benchmark while running at a fraction of the inference cost.

## Overview

Frontier models like GPT-4o and Claude are strong general-purpose SQL generators, but they make dialect-specific mistakes and are expensive to run in production. This project fine-tunes **Qwen 2.5 7B Coder** on curated text-to-SQL data using **LoRA** (Low-Rank Adaptation) to produce a specialist model that outperforms frontier models on complex, schema-heavy SQL generation tasks.

The end deliverables are:
- A fine-tuned model published on Hugging Face with reproducible weights
- A custom evaluation harness with execution-accuracy metrics on the BIRD benchmark
- A live inference demo
- A technical write-up documenting methodology, results, and lessons learned

## Why this matters

Specialist fine-tuned models are how production AI systems actually get built. Wrapping a frontier API is fine for prototypes, but real teams fine-tune smaller models for cost, latency, and accuracy on narrow tasks. This project demonstrates that workflow end-to-end: dataset curation, training, evaluation, deployment.

## Tech Stack

**Model and Training**
- **Base model:** Qwen 2.5 7B Coder (Apache 2.0)
- **Fine-tuning method:** LoRA / QLoRA via [Unsloth](https://github.com/unslothai/unsloth)
- **Framework:** PyTorch with Hugging Face `transformers` and `trl`
- **Experiment tracking:** Weights and Biases

**Data**
- **Training:** BIRD train split plus Spider train split (approximately 12K examples after filtering)
- **Evaluation:** BIRD dev set (1,534 examples) for held-out testing

**Infrastructure**
- **Training compute:** RunPod (single A100 40GB)
- **Local dev:** WSL2 Ubuntu 24.04 on RTX 4060 Laptop (8GB VRAM) for code, debugging, and inference testing
- **Environment management:** `uv` for Python dependencies

**Serving and Distribution**
- **Weights:** Hugging Face Hub
- **Demo:** Gradio Space on Hugging Face
- **Inference API:** FastAPI for programmatic access

**Evaluation**
- Custom Python eval harness running predicted SQL against actual SQLite databases
- Primary metric: execution accuracy (does the generated SQL return the same rows as ground truth)
- Secondary metrics: exact match, valid SQL rate, latency, cost per query

## 4-Week Plan

### Week 1: Foundations and Baseline

Set up the development environment and establish the baseline number we need to beat.

- WSL2 and CUDA verification, `uv` Python environment, Unsloth installation
- SQL refresher (Mode Analytics tutorial, BIRD example walkthroughs)
- Download and explore BIRD and Spider datasets
- Write evaluation script that runs predicted SQL against BIRD's SQLite databases
- Run baseline eval: un-fine-tuned Qwen 2.5 7B on BIRD dev set
- Run reference evals: GPT-4o and Claude on the same set for comparison
- **Deliverable:** Working eval harness, locked baseline numbers

### Week 2: First Fine-Tune

Get a working training pipeline and produce a model that beats the baseline.

- Set up RunPod environment, replicate local dev stack on cloud GPU
- Run Unsloth LoRA fine-tune on small subset (about 1K examples) to verify pipeline
- First full training run on combined BIRD and Spider training data
- Push checkpoint to Hugging Face, run eval, log results to Weights and Biases
- **Deliverable:** First fine-tuned model, measurable improvement over baseline

### Week 3: Iteration and Optimization

Push performance hard. This is where most of the actual model quality is won.

- Hyperparameter sweep: learning rate, LoRA rank, training epochs, batch size
- Data quality experiments: filter low-quality examples, balance schema complexity
- Prompt engineering: schema-aware prompting, few-shot examples in training data
- Optional: synthetic data generation using Claude as a labeler for hard cases
- Error analysis on failed predictions to identify systemic weaknesses
- **Deliverable:** Optimized model, target: beat GPT-4o execution accuracy on BIRD

### Week 4: Ship It

Polish, deploy, and distribute.

- Final training run with best hyperparameters, push final weights to Hugging Face
- Write thorough model card documenting training data, methodology, limitations
- Build Gradio demo and deploy as Hugging Face Space
- Build minimal FastAPI inference endpoint
- Write technical blog post covering methodology, results, and lessons learned
- Distribute: post on X, LinkedIn, Hacker News, relevant subreddits
- **Deliverable:** Public model, demo, and blog post

## Success Criteria

- Beat or match GPT-4o on BIRD dev set execution accuracy
- Inference cost at least 20x cheaper than GPT-4o per query
- Reproducible: published weights, training code, eval harness, methodology
- Demoable: anyone can try the model in a browser within 60 seconds

## Repository Structure

```
.
├── data/              # Dataset prep and loading scripts
├── training/          # Fine-tuning scripts and configs
├── eval/              # Evaluation harness and benchmark runners
├── inference/         # FastAPI server and Gradio demo
├── notebooks/         # Exploration and analysis
├── results/           # Eval outputs, comparison reports
└── README.md
```

## Status

In progress. Currently in Week 1 (environment setup and baseline eval).

## License

MIT
