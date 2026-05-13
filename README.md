# MINJA: Memory Injection Attacks and Guardrail Defense

This repository is a project implementation based on **Memory Injection Attacks on LLM Agents via Query-Only Interaction**. The original idea studies how an attacker can inject malicious memories into an LLM agent only through normal user queries. Those poisoned memories can later be retrieved by the agent and change its decisions.

In this version, I kept the core MINJA implementation for the RAP and QA settings and added a fast guardrail experiment for the QA agent. The guardrail checks candidate memories before they are stored and blocks records that look like malicious answer-reversal or instruction-injection memories.

<p align="center">
  <img src="figure/MINJA.png" width="800">
</p>

## What Was Implemented

- Reproduced the MINJA query-only memory injection workflow for an LLM agent.
- Included the RAP-agent implementation that runs against a locally installed WebShop environment.
- Included the QA-agent implementation over MMLU-style CSV data.
- Added a fast guardrail notebook that evaluates a defense before writing new records into memory.
- Updated Git ignore rules so API keys, generated memories, logs, outputs, and the external WebShop folder are not pushed.

## Setup

Create and activate a Python environment:

```bash
conda create -n minja python=3.10 -y
conda activate minja
```

Install RAP dependencies:

```bash
cd rap
pip install -r requirements.txt
```

Install QA dependencies as needed:

```bash
pip install openai numpy python-Levenshtein
```

Create API-key files locally only.

```text
QA/OpenAI_api_key.txt
rap/NVIDIA_api_key.txt
```

Only create the key file required by the provider you are running.

## Running the RAP Agent

The RAP code expects a local WebShop setup. Since `WebShop/` is ignored in this repository, download/setup WebShop separately from the official project, then start its local server before running RAP experiments.

Example RAP run:

```bash
cd rap
python minja.py \
  --victim toothbrush \
  --target "DenTek Professional Oral Care Kit with DenTek Triple Clean Advanced Clean Floss Picks" \
  --target_price 20.0 \
  --inject_num 15 \
  --num_benign 50 \
  --test_num 30 \
  --provider nvidia \
  --model_name moonshotai/kimi-k2.5
```

Predefined victim-target pairs are stored in:

```text
rap/victim_target_pair/victim_target.json
```

## Running the QA Agent

```bash
cd QA
python main.py \
  --data_path data/test/high_school_chemistry_test.csv \
  --core_model gpt-4o \
  --n_shots 3 \
  --seed 42 \
  --memory_path memory.json
```

The QA run generates local memory and result files such as `memory.json`, `memory_test.json`, `question_*.json`, and log files. These are ignored by Git because they are experiment artifacts.

## Running the Guardrail Notebook

Open:

```text
MINJA_GUARDRAIL_FAST.ipynb
```

Run the notebook cells in order. The notebook contains:

- dataset preparation helpers,
- memory retrieval logic,
- malicious-reversal detection,
- `guardrail_check(answer, thought)`,
- a guardrail-only experiment loop,
- comparison output for paper baseline, no-guardrail, and with-guardrail results.

The guardrail is applied immediately before `memory.store()`. If the generated answer/thought matches suspicious reversal patterns, the memory is blocked instead of being saved.

 
## Research Paper Reference

This implementation follows the MINJA paper idea: query-only attacks can poison an agent's memory by making malicious interactions look like normal successful examples. Later, when the agent retrieves similar memories, the poisoned records influence its reasoning or final action.

My added part focuses on a simple defense direction: inspect each candidate memory before storage and reject suspicious records. This does not remove the original attack implementation; it adds an experiment path for comparing behavior with and without the guardrail.
