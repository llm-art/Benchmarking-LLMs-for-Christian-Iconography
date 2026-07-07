# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Benchmarking framework that evaluates three families of vision models on Christian iconography classification across three art datasets. The framework compares generative multimodal LLMs (GPT, Gemini, Bedrock), contrastive VLMs (CLIP, SigLIP), and supervised CNN baselines (ResNet-50) under zero-shot and few-shot conditions.

**Datasets (all use the same 10 saint classes):**
- `ArtDL` — 1,864 test images (published split from artdl.org)
- `ICONCLASS` — 863 images filtered from the Iconclass AI test set
- `wikidata` — 717 images curated via SPARQL with Iconclass annotations

**Test configurations:**
- `test_1` — zero-shot with category labels only
- `test_2` — zero-shot with detailed Iconclass descriptions
- `test_3` — few-shot with 5 example images per category

## Installation

```bash
pip install -r requirements.txt
# venv is at venv/ if already initialized
source venv/bin/activate
```

## Running Scripts

All scripts are in `script/` and use Click CLI with consistent flags: `--models`, `--datasets`, `--folders`, `--limit`, `--batch_size`, `--save_frequency`, `--verbose`, `--clean`.

```bash
# Generative LLMs
python script/execute_gpt.py --models gpt-4o-2024-11-20 --datasets ArtDL ICONCLASS wikidata --folders test_1 test_2 test_3
python script/execute_gemini.py --models gemini-2.5-pro --datasets ArtDL --folders test_1
python script/execute_bedrock.py --models claude-sonnet-4-6 --datasets ArtDL --folders test_1

# Contrastive VLMs
python script/execute_clip.py --models clip-vit-large-patch14 --datasets ArtDL --folders test_1 test_2
python script/execute_siglip.py --models siglip-so400m-patch14-384 --datasets ArtDL --folders test_1 test_2

# Few-shot fine-tuning for CLIP/SigLIP (test_3)
python script/few-shot.py --models clip-vit-base-patch32 --datasets ArtDL --folders test_3 --num_epochs 150

# Evaluate and generate metrics
python script/evaluate.py --models gpt-4o-2024-11-20 gemini-2.5-pro --datasets ArtDL ICONCLASS --folders test_1 test_2 test_3

# ResNet-50 baseline
python baseline/resnet50_baseline.py --dataset ArtDL --train_split 0.8 --epochs 100
```

## API Configuration

API keys are stored in INI files (gitignored; templates provided):

```bash
# OpenAI (GPT)
script/gpt_data/psw.ini        # [openai] api_key=...

# Google Gemini
script/gemini_data/config.ini  # [gemini] api_key=...

# Harvard Bedrock gateway (execute_bedrock.py)
# Uses go.apis.huit.harvard.edu — configure credentials per that gateway's spec
```

## Architecture

### Script Pattern (all `execute_*.py`)

Each execute script follows the same class hierarchy:
- **`ModelConfig`** — model names, pricing per token, API identifiers
- **`CacheManager`** — loads/saves JSON cache at `{provider}_data/cache/{dataset}/{test}/{model}.json` to avoid redundant API calls
- **`ImageClassifier`** — loads prompts from `prompts/{dataset}/{test}.txt`, encodes images as base64, batches requests, parses JSON responses, writes `probs.npy`
- **`JSONExtractor`** / **`JSONResponseParser`** / **`ClassResolver`** — shared utilities for parsing model output; `ClassResolver` handles fuzzy matching of Iconclass IDs (e.g., `11H(SEBASTIAN)` aliases)

### Prompts

Templates live at `prompts/{dataset}/{test}.txt` (e.g., `prompts/ArtDL/test_1.txt`). The scripts fill in `{FEW_SHOT_EXAMPLES}` and `{CLASS_LIST}` at runtime. All prompts instruct the model to return a JSON object `{ "<image_id>": {"class": "<CATEGORY_ID>", "confidence": 0.0–1.0} }`.

### Output Structure

```
test_1/ArtDL/gemini-2.5-pro/
  probs.npy             # shape: (N_images, N_classes) float array
  confusion_matrix.csv
  confusion_matrix.png
  class_metrics.csv
  summary_metrics.csv
```

`evaluate.py` reads `probs.npy` and ground-truth labels to compute top-1 accuracy, per-class metrics, and confusion matrices.

### Logging

`script/logger_utils.py` provides `setup_logger(dataset, test, model, output_folder, verbose)` used by all scripts. Logs go to `{output_folder}/{dataset}/{test}/{model}.log`. Pass `--verbose` for DEBUG level.

## Key Data Files

- `dataset/` — Jupyter notebooks for dataset construction and analysis; `dataset/consistency/` holds cross-dataset matched pairs (45 image pairs found via perceptual hashing, Hamming distance ≤ 8)
- `prompts/base_prompt_template.txt` — base template used to generate per-dataset prompt files
- `models.md` — reference table of model architectures
- `datasets.md` — reference table of available art datasets
