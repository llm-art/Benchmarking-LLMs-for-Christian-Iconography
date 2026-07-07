# Can AI Recognize the Saints?
Benchmarking Vision-Language and Multimodal Models for Christian Iconography Classification

This repository benchmarks three families of vision models — supervised CNNs, contrastive vision-language models (CLIP, SigLIP), and generative multimodal LLMs (GPT, Gemini) — on classifying Christian saints across three Iconclass-annotated art datasets. It accompanies a study structured around four questions: how do the architectures compare (RQ1); how does accuracy change when prompts are enriched with iconographic descriptions or few-shot exemplars (RQ2); do successive LLM generations improve systematically (RQ3); and what do the residual failures of the strongest models reveal about the limits of the task (RQ4)?

## Table of Contents

- [Leaderboard](#leaderboard)
- [Results](#results)
  - [ArtDL](#artdl)
  - [ICONCLASS](#iconclass)
  - [Wikidata](#wikidata)
  - [Error Analysis](#error-analysis)
- [Repository Overview](#repository-overview)
- [Datasets](#datasets)
- [Getting Started](#getting-started)
- [Scripts Overview and Usage](#scripts-overview-and-usage)
- [Prompt Engineering](#prompt-engineering)

## Leaderboard

Best mean accuracy (averaged across Test 1, Test 2, and Test 3) per dataset. Models evaluated under three conditions: (1) zero-shot with labels, (2) zero-shot with Iconclass descriptions, and (3) few-shot with 5 exemplars.

| Rank | Model | ArtDL | ICONCLASS | Wikidata |
|------|-------|-------|-----------|----------|
| 1 | gemini-3-flash-preview | **92.67%** | **92.89%** | **92.08%** |
| 2 | gemini-3.1-pro-preview | 91.88% | 92.16% | 89.99% |
| 3 | gemini-2.5-pro | 89.88% | 86.52% | 85.96% |
| 4 | gemini-3.1-flash-lite-preview | 88.21% | 84.00% | 83.29% |
| 5 | gemini-2.5-flash | 86.52% | 78.33% | 81.98% |
| 6 | gpt-4o-2024-11-20 | 86.11% | 72.65% | 75.56% |
| 7 | gemini-2.5-flash-lite | 83.24% | 68.44% | 67.40% |
| 8 | gpt-4o-mini-2024-07-18 | 84.08% | 56.93% | 59.82% |
| 9 | gpt-5.2-2025-12-11 | 82.44% | 65.82% | 71.19% |
| 10 | gpt-5-mini-2025-08-07 | 81.47% | 73.27% | 78.57% |
| 11 | siglip-so400m-patch14-384 | 60.26% | 57.84% | 63.58% |
| 12 | siglip-large-patch16-384 | 60.05% | 45.67% | 55.05% |
| 13 | siglip-base-patch16-512 | 57.60% | 39.59% | 53.49% |
| 14 | clip-vit-large-patch14 | 35.53% | 39.42% | 56.28% |
| 15 | clip-vit-base-patch16 | 27.82% | 30.29% | 48.17% |
| 16 | clip-vit-base-patch32 | 23.04% | 27.95% | 45.33% |
| -- | *ResNet-50 baseline* | *84.44%* | *40.46%* | *43.97%* |

## Results

Performance is evaluated through **top-1 accuracy**. Three test configurations are used across all models and datasets:
- **Test 1**: Zero-shot classification using only category labels
- **Test 2**: Zero-shot classification with detailed Iconclass descriptions
- **Test 3**: Few-shot learning with 5 example images per category

### ArtDL

| Model | Test 1 | Test 2 | Test 3 | Mean +/- SD |
|-------|--------|--------|--------|-------------|
| *Contrastive VLMs* | | | | |
| clip-vit-base-patch32 | 16.15% | 31.55% | 21.41% | 23.04 +/- 6.39 |
| clip-vit-base-patch16 | 25.64% | 28.70% | 29.13% | 27.82 +/- 1.55 |
| clip-vit-large-patch14 | 30.58% | 44.31% | 31.71% | 35.53 +/- 6.22 |
| siglip-base-patch16-512 | 48.71% | 68.19% | 55.90% | 57.60 +/- 8.04 |
| siglip-large-patch16-384 | 54.45% | 72.21% | 53.49% | 60.05 +/- 8.61 |
| siglip-so400m-patch14-384 | 53.86% | 70.55% | 56.38% | 60.26 +/- 7.35 |
| *Legacy LLMs (2024-2025)* | | | | |
| gemini-2.5-flash-lite | 83.32% | 84.96% | 81.43% | 83.24 +/- 1.44 |
| gemini-2.5-flash | 87.39% | 86.86% | 85.30% | 86.52 +/- 0.89 |
| gemini-2.5-pro | 90.88% | 90.99% | 87.77% | 89.88 +/- 1.49 |
| gpt-4o-mini-2024-07-18 | 82.67% | 84.98% | 84.60% | 84.08 +/- 1.01 |
| gpt-4o-2024-11-20 | 86.21% | 86.59% | 85.52% | 86.11 +/- 0.44 |
| *Next-gen LLMs (2025-2026)* | | | | |
| gemini-3.1-flash-lite-preview | 89.75% | 89.59% | 85.30% | 88.21 +/- 2.06 |
| gemini-3-flash-preview | **93.51%** | **92.81%** | 91.68% | 92.67 +/- 0.75 |
| gemini-3.1-pro-preview | 91.85% | 91.95% | **91.85%** | 91.88 +/- 0.05 |
| gpt-5-mini-2025-08-07 | 81.22% | 80.85% | 82.35% | 81.47 +/- 0.64 |
| gpt-5.2-2025-12-11 | 81.06% | 83.96% | 82.30% | 82.44 +/- 1.19 |
| *Baseline* | 84.44% | | | |

### ICONCLASS

| Model | Test 1 | Test 2 | Test 3 | Mean +/- SD |
|-------|--------|--------|--------|-------------|
| *Contrastive VLMs* | | | | |
| clip-vit-base-patch32 | 24.74% | 29.30% | 29.82% | 27.95 +/- 2.28 |
| clip-vit-base-patch16 | 30.00% | 27.37% | 33.51% | 30.29 +/- 2.52 |
| clip-vit-large-patch14 | 40.00% | 35.44% | 42.81% | 39.42 +/- 3.04 |
| siglip-base-patch16-512 | 43.51% | 33.33% | 41.93% | 39.59 +/- 4.47 |
| siglip-large-patch16-384 | 48.95% | 38.77% | 49.30% | 45.67 +/- 4.88 |
| siglip-so400m-patch14-384 | 59.47% | 53.16% | 60.88% | 57.84 +/- 3.36 |
| *Legacy LLMs (2024-2025)* | | | | |
| gemini-2.5-flash-lite | 67.56% | 69.06% | 68.71% | 68.44 +/- 0.64 |
| gemini-2.5-flash | 77.40% | 77.98% | 79.61% | 78.33 +/- 0.94 |
| gemini-2.5-pro | 86.67% | 86.21% | 86.67% | 86.52 +/- 0.22 |
| gpt-4o-mini-2024-07-18 | 55.74% | 59.56% | 55.50% | 56.93 +/- 1.86 |
| gpt-4o-2024-11-20 | 72.42% | 74.28% | 71.26% | 72.65 +/- 1.24 |
| *Next-gen LLMs (2025-2026)* | | | | |
| gemini-3.1-flash-lite-preview | 83.66% | 84.24% | 84.11% | 84.00 +/- 0.25 |
| gemini-3-flash-preview | **93.28%** | **92.24%** | **93.16%** | 92.89 +/- 0.46 |
| gemini-3.1-pro-preview | 92.24% | **92.24%** | 92.00% | 92.16 +/- 0.11 |
| gpt-5-mini-2025-08-07 | 73.46% | 74.28% | 72.07% | 73.27 +/- 0.91 |
| gpt-5.2-2025-12-11 | 63.96% | 65.93% | 67.56% | 65.82 +/- 1.47 |
| *Baseline (ResNet-50 trained)* | 40.46% | | | |

### Wikidata

| Model | Test 1 | Test 2 | Test 3 | Mean +/- SD |
|-------|--------|--------|--------|-------------|
| *Contrastive VLMs* | | | | |
| clip-vit-base-patch32 | 45.95% | 44.52% | 45.52% | 45.33 +/- 0.60 |
| clip-vit-base-patch16 | 50.78% | 46.66% | 47.08% | 48.17 +/- 1.85 |
| clip-vit-large-patch14 | 56.76% | 56.61% | 55.48% | 56.28 +/- 0.57 |
| siglip-base-patch16-512 | 57.47% | 46.94% | 56.05% | 53.49 +/- 4.67 |
| siglip-large-patch16-384 | 60.03% | 43.95% | 61.17% | 55.05 +/- 7.86 |
| siglip-so400m-patch14-384 | 66.29% | 59.60% | 64.86% | 63.58 +/- 2.88 |
| *Legacy LLMs (2024-2025)* | | | | |
| gemini-2.5-flash-lite | 66.03% | 68.08% | 68.08% | 67.40 +/- 0.97 |
| gemini-2.5-flash | 79.45% | 83.97% | 82.51% | 81.98 +/- 1.88 |
| gemini-2.5-pro | 85.71% | 86.01% | 86.15% | 85.96 +/- 0.18 |
| gpt-4o-mini-2024-07-18 | 57.87% | 62.10% | 59.48% | 59.82 +/- 1.74 |
| gpt-4o-2024-11-20 | 75.51% | 75.95% | 75.22% | 75.56 +/- 0.30 |
| *Next-gen LLMs (2025-2026)* | | | | |
| gemini-3.1-flash-lite-preview | 85.28% | 82.07% | 82.51% | 83.29 +/- 1.42 |
| gemini-3-flash-preview | **91.68%** | **90.96%** | **93.59%** | 92.08 +/- 1.11 |
| gemini-3.1-pro-preview | 90.09% | 89.20% | 90.67% | 89.99 +/- 0.60 |
| gpt-5-mini-2025-08-07 | 78.28% | 79.30% | 78.13% | 78.57 +/- 0.52 |
| gpt-5.2-2025-12-11 | 70.70% | 72.89% | 69.97% | 71.19 +/- 1.24 |
| *Baseline (ResNet-50 trained)* | 43.97% | | | |

### Error Analysis

To probe what the strongest models still get wrong, ICONCLASS (Test 1, label-only) is analyzed layer by layer, comparing the top model (Gemini-3-Flash), a mid-tier model (GPT-5.2), and the two fine-tuned ResNet-50 baselines:

1. **Per-class confusion** — the ResNet-50 baselines scatter errors diffusely across the matrix (undertrained on ~600-700 images); Gemini-3-Flash (93.3% accuracy, 58 errors) and GPT-5.2 (64.0%, 311 errors) concentrate their far fewer errors on a small set of interpretable off-diagonal cells.
2. **Confusable pairs** — for each ordered class pair (k → l), the confusion rate CR(k→l) = N(true k, pred l) / N(true k) is computed. Both models fail on the same boundaries — Magdalene↔Catherine, Antony Abbot→Jerome, Paul↔Peter, Peter→John — despite a five-fold gap in error count.
3. **Image-property taxonomy** — every misclassified image is assigned exactly one cause by a most-proximal-cause precedence rule: information loss (greyscale or colourfulness below 15), low figure saliency (principal figure below 5% of canvas area), or iconographic overlap (predicted class forms a confusable pair with the true class, CR≥1%). Both models land in almost the same proportions (~50% low saliency, ~30% information loss, ~20% iconographic overlap) despite the accuracy gap.

The convergence of failure modes across two architecturally distinct models indicates the residual errors are governed by intrinsic iconographic ambiguity rather than model capacity. Scripts: `dataset/shift_analysis/perclass_layer.py`, `confusable_pairs.py`, `error_taxonomy.py`; full write-up in `dataset/reports/error_taxonomy.md`.

## Repository Overview

This repository presents a comprehensive benchmarking framework for evaluating three families of models — supervised convolutional networks, contrastive vision-language models (VLMs), and generative multimodal large language models (LLMs) — on image classification tasks in the cultural heritage domain, using Christian iconography as a case study.

### Model Categories

- **Generative Multimodal LLMs** — Legacy (GPT-4o, GPT-4o-mini, Gemini 2.5 Flash-Lite/Flash/Pro) and state-of-the-art (GPT-5-mini, GPT-5.2, Gemini 3-Flash, Gemini 3.1-Flash-Lite, Gemini 3.1-Pro)
- **Contrastive Vision-Language Models** — CLIP (ViT-B/32, ViT-B/16, ViT-L/14) and SigLIP (Base-Patch16-512, Large-Patch16-384, SO400M-Patch14-384)
- **Supervised Baselines** — ResNet-50: the originally published model for ArtDL, fine-tuned from scratch on an 80/20 train/test split for ICONCLASS and Wikidata

## Datasets

1. **[ArtDL](https://artdl.org/)** — 1,864 test images, 10 saint classes (published test split)
2. **[ICONCLASS AI test set](https://iconclass.org/testset/)** — 863 images, 10 saint classes (filtered from ~87.5K)
3. **[Wikidata](https://www.wikidata.org/)** — 717 images, 10 saint classes (curated via SPARQL with Iconclass annotations)

Class overlap across the three sets is incomplete: John the Baptist, Paul, Jerome, Francis, Peter, and Mary Magdalene are shared by all three; Catherine and Antony Abbot are shared only by ICONCLASS and Wikidata; the remaining classes (Dominic, Mary, Sebastian, Antony of Padua, Matthew, Luke, Joseph, John) are dataset-specific.

## Getting Started

### Prerequisites and Installation

```bash
git clone https://github.com/llm-art/ai-recognize-saints.git
cd ai-recognize-saints
pip install -r requirements.txt
```

### Configure API Access

**OpenAI GPT Models:**
```ini
# Create script/gpt_data/psw.ini
[openai]
api_key=your_openai_api_key_here
```

**Google Gemini Models:**
```ini  
# Create script/gemini_data/config.ini
[gemini]
api_key=your_gemini_api_key_here
```

### Quick Start

```bash
# Zero-shot evaluation with labels
python script/execute_gpt.py --models gpt-5.2-2025-12-11 --datasets ArtDL --folders test_1
python script/execute_gemini.py --models gemini-3-flash-preview --datasets ArtDL --folders test_1

# Zero-shot evaluation with descriptions  
python script/execute_gpt.py --models gpt-5.2-2025-12-11 --datasets ArtDL --folders test_2
python script/execute_gemini.py --models gemini-3-flash-preview --datasets ArtDL --folders test_2

# Few-shot evaluation
python script/execute_gpt.py --models gpt-5.2-2025-12-11 --datasets ArtDL --folders test_3
python script/execute_gemini.py --models gemini-3-flash-preview --datasets ArtDL --folders test_3

# Generate evaluation metrics
python script/evaluate.py --models gpt-5.2-2025-12-11 gemini-3-flash-preview gemini-3.1-pro-preview --datasets ArtDL ICONCLASS wikidata --folders test_1 test_2 test_3
```

To reproduce the paper's reported accuracy, pass `--temperature 0.1` for models where manual control is supported (legacy Gemini and GPT snapshots). State-of-the-art snapshots differ by provider default: Gemini 3.x recommends `--temperature 1`, while GPT-5.x deprecates manual temperature control owing to its reasoning behaviour.

### Output Structure

```
test_1/                           # Zero-shot with labels
├── ArtDL/
│   ├── gemini-3-flash-preview/
│   │   ├── probs.npy             # Classification probabilities
│   │   ├── confusion_matrix.png
│   │   ├── confusion_matrix.csv
│   │   └── class_metrics.csv
│   ├── gemini-3.1-pro-preview/
│   ├── gpt-5.2-2025-12-11/
│   ├── clip-vit-large-patch14/
│   ├── siglip-so400m-patch14-384/
│   └── ...
├── ICONCLASS/
└── wikidata/

test_2/                           # Zero-shot with descriptions  
test_3/                           # Few-shot learning
```

## Scripts Overview and Usage

The `script/` directory contains the core evaluation framework.

### Multimodal LLM Scripts

#### `execute_gpt.py` - OpenAI GPT Models
**Purpose:** Executes GPT models (GPT-4o, GPT-4o-mini, GPT-5-mini, GPT-5.2) for zero-shot and few-shot image classification.

```bash
Options:
  --folders TEXT            List of folders to use
  --models TEXT             List of model names to use
  --limit INTEGER           Limit the number of images to process
  --batch_size INTEGER      Number of images per batch
  --save_frequency INTEGER  How often to save cache (in batches)
  --datasets TEXT           List of datasets to use
  --verbose                 Enable verbose logging (DEBUG level)
  --temperature FLOAT       Temperature for generation (default: 0.0, min:
                            0.0)
  --top_p FLOAT             Top-p (nucleus sampling) for generation (default:
                            0.1)
  --seed INTEGER            Seed for deterministic results (default: 12345)
  --clean                   Remove cache and logs from previous runs before
                            starting
  --help                    Show this message and exit.
```

```bash
# Basic execution for GPT models
python script/execute_gpt.py --models gpt-4o-2024-11-20 gpt-4o-mini-2024-07-18 gpt-5-mini-2025-08-07 gpt-5.2-2025-12-11 --datasets ArtDL ICONCLASS wikidata --folders test_1 test_2 test_3

# With custom parameters
python script/execute_gpt.py --models gpt-5.2-2025-12-11 --datasets ArtDL --folders test_1 --limit 100 --batch_size 5
```

**Configuration Required:** API key in `script/gpt_data/psw.ini`

#### `execute_gemini.py` - Google Gemini Models  
**Purpose:** Executes Gemini models (2.5 Flash/Pro legacy and 3.x state-of-the-art) for multimodal classification tasks.

```bash
Options:
  --folders TEXT            List of folders to use
  --models TEXT             List of model names to use
  --limit INTEGER           Limit the number of images to process
  --batch_size INTEGER      Number of images per batch
  --save_frequency INTEGER  How often to save cache (in batches)
  --datasets TEXT           List of datasets to use
  --verbose                 Enable verbose logging (DEBUG level)
  --temperature FLOAT       Temperature for generation (default: 0.0, min:
                            0.0)
  --top_k INTEGER           Top-k for sampling (default: 32)
  --clean                   Remove cache and logs from previous runs before
                            starting
  --help                    Show this message and exit.
```

```bash
# Run Gemini models
python script/execute_gemini.py --models gemini-2.5-pro gemini-2.5-flash gemini-3-flash-preview gemini-3.1-pro-preview --datasets ArtDL ICONCLASS wikidata --folders test_1 test_2

# Few-shot evaluation
python script/execute_gemini.py --models gemini-3-flash-preview --datasets ICONCLASS --folders test_3
```

**Configuration Required:** API key in `script/gemini_data/config.ini`

### Vision-Language Encoder Scripts

#### `execute_clip.py` - CLIP Models
**Purpose:** Implements CLIP variants for contrastive learning-based classification.

```bash
python script/execute_clip.py --models clip-vit-base-patch32 --datasets ArtDL --folders test_1
```

#### `execute_siglip.py` - SigLIP Models 
**Purpose:** Runs SigLIP models with sigmoid-based contrastive learning.

```bash
python script/execute_siglip.py --models siglip-base-patch16-512 --datasets ArtDL --folders test_1
```

### Evaluation and Analysis Scripts

#### `evaluate.py` - Results Analysis
**Purpose:** Generates standardized evaluation metrics (top-1 accuracy, confusion matrices) for all models.

```bash
python script/evaluate.py --models gpt-5.2-2025-12-11 gemini-3-flash-preview --datasets ArtDL ICONCLASS --folders test_1 test_2
```

#### `few-shot.py` - Few-Shot Learning Framework
**Purpose:** Implements few-shot learning approaches for vision-language models.

```bash
python script/few-shot.py --models clip-vit-base-patch32 --datasets ArtDL --folders test_3 --num_epochs 150
```

#### `generation_compare.py` - Cross-Generation Accuracy Comparison
**Purpose:** Visualizes generational accuracy shifts by model family across datasets (produces Fig. 1 in the paper).

#### `analyze_prediction_shifts.py` - Cross-Generation Misclassification Analysis
**Purpose:** Generates heatmaps of cross-generation misclassifications between legacy and state-of-the-art models.

#### `consistency_analysis.py` - Cross-Dataset Consistency
**Purpose:** Evaluates prediction stability across the 45 matched cross-dataset image pairs identified via perceptual hashing.

#### `compute_overlap.py` - Dataset Overlap Detection
**Purpose:** Identifies duplicate images across datasets using perceptual hashing (Hamming distance threshold of 8).

### Error-Analysis Scripts (`dataset/shift_analysis/`)

These scripts implement the error-analysis protocol described in the paper: a structured comparison of the ICONCLASS Test 1 predictions from Gemini-3-Flash, GPT-5.2, and the two fine-tuned ResNet-50 baselines.

#### `perclass_layer.py` - Per-Class Confusion and Error Entropy
**Purpose:** Builds the side-by-side confusion-matrix figure (Fig. 2 in the paper) and a normalized off-diagonal entropy table contrasting the diffuse baseline errors with the concentrated frontier-model errors.

```bash
python dataset/shift_analysis/perclass_layer.py
```

#### `confusable_pairs.py` - Confusable-Pair Detection
**Purpose:** Computes the directional confusion rate CR(k→l) = N(true k, pred l)/N(true k) per ordered class pair and flags pairs shared by both frontier models (CR≥1% in each).

```bash
python dataset/shift_analysis/confusable_pairs.py
```

#### `error_taxonomy.py` - Image-Property Taxonomy of Misclassifications
**Purpose:** Assigns each misclassified image to exactly one category — information loss, low figure saliency, or iconographic overlap — via a fixed precedence rule, producing Fig. 3 in the paper.

```bash
python dataset/shift_analysis/error_taxonomy.py
# skip the Faster R-CNN person detector (Cat. 2 will be unknown):
python dataset/shift_analysis/error_taxonomy.py --skip-detector
```

Full write-up of the protocol, thresholds, and findings: `dataset/reports/error_taxonomy.md`.

### Baseline Scripts

#### `baseline/resnet50_baseline.py` - Supervised Learning Baseline
**Purpose:** Traditional CNN-based classification using ResNet-50 architecture.

```bash
python baseline/resnet50_baseline.py --dataset ArtDL --train_split 0.8 --epochs 100
```

#### `baseline/artdl_baseline.py` - Dataset-Specific Baseline  
**Purpose:** Specialized baseline implementation optimized for ArtDL dataset characteristics.

### Dataset Processing (`dataset/`)
- **Data preprocessing notebooks** — Jupyter notebooks for dataset preparation and analysis
- **`dataset/consistency/`** — Cross-dataset consistency evaluation results and analysis
- **`dataset/consistency_trends/`** — Temporal consistency trends across model generations
- **`dataset/shift_analysis/`** — Cross-generation misclassification shift data, plus the ICONCLASS error-analysis pipeline (per-class confusion, confusable pairs, image-property taxonomy) behind Fig. 2–3 and RQ4
- **`dataset/reports/error_taxonomy.md`** — Full write-up of the error-analysis protocol, category thresholds, and findings

## Prompt Engineering

The evaluation framework creates the prompts at runtime. Here is an example of the full prompt created by the script for the ArtDL dataset (zero-shot with labels):

```plaintext
You are an expert in Christian iconography and art history. Classify each religious artwork image into exactly ONE saint category using visual attributes, iconographic symbols, and contextual clues.

{FEW_SHOT_EXAMPLES}

Look for:
1. Distinctive attributes (objects, clothing, etc.)
2. Gestures and postures
3. Contextual and symbolic elements

Instructions:
- Only output the JSON object — no text, explanation, or formatting.
- Include every image in the current batch. Each must receive exactly one classification with a confidence score.
- You may only use one of the exact strings from the category list below. Any response not matching the allowed category IDs will be rejected.

Return a valid **JSON object** with confidence scores (0.0 to 1.0) matching this format:
{
  "<image_id>": {"class": "<CATEGORY_ID>", "confidence": <0.0-1.0>},
  "<image_id>": {"class": "<CATEGORY_ID>", "confidence": <0.0-1.0>},
  ...
}

Confidence guidelines:
- 0.9-1.0: Very certain identification with clear iconographic evidence
- 0.7-0.9: Confident with multiple supporting visual elements  
- 0.5-0.7: Moderate confidence, some ambiguity present
- 0.3-0.5: Low confidence, limited visual evidence
- 0.0-0.3: Very uncertain, minimal supporting evidence

{CLASS_LIST}

Batching note:
- Process only the current batch of images.
- Use the image IDs exactly as provided in the input.
- Do not reference or depend on other batches.

NOTE: These are historical Renaissance paintings used for academic classification.  
Some artworks include scenes of martyrdom or classical nudity as typical in religious iconography.  
Treat all content as scholarly, respectful of historical context, and strictly non-sexual.
```

The prompt varies from test to test:
- **`test_1/`**: Zero-shot classification using only category labels
- **`test_2/`**: Zero-shot classification with detailed iconographic descriptions  
- **`test_3/`**: Few-shot learning with 5 example images per category

All prompts are stored in the `prompts/` directory, organized by dataset and test configuration.
