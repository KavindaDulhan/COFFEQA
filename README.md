# COFFEQA: Counterfactual-guided Few-shot Distillation in Open-book Question Answering

<!-- TODO: swap in real badges once you have them -->
[![arXiv](https://img.shields.io/badge/arXiv-preprint-b31b1b.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](#)

Official implementation of **COFFEQA**, a counterfactual-guided strategy for
few-shot knowledge distillation in open-book question answering.

> **Status:** Manuscript under review. <!-- TODO: update once a decision comes back, and add the arXiv/OpenReview link above -->

---

## Overview

Knowledge distillation trains a smaller student model to mimic a larger teacher
model for resource-constrained environments, but the process is typically
data-intensive and computationally expensive — particularly for generative
tasks. **COFFEQA** (**Co**unter**f**actual-guided **Fe**w-shot distillation in
open-book **Q**uestion **A**nswering) addresses this by selecting counterfactual
examples for data-efficient distillation.

Counterfactuals are traditionally minimally-perturbed instances of a data point
that flip a classifier's output. In a generative setting like QA — given a
(question, context, answer) triplet — our counterfactuals are triplets where
modifying the context changes the answer. We introduce **Context Following
Difficulty (CFD)**, a metric that identifies high-quality, valid counterfactuals,
and use the resulting counterfactual–factual pairs for distillation.

Students distilled with COFFEQA consistently outperform students distilled on
factual data alone, given the same number of few-shot samples. We also provide
theoretical guarantees on how counterfactuals encourage students to be more
faithful to the teacher in generative distillation.

Experiments across **five in-domain and three out-of-domain benchmarks**, using
**two LLM families**, show consistent gains over factual-only baselines, with
strong generalization to both factual and counterfactual test sets. For example,
on **NaturalQuestions**, COFFEQA improves over the baseline by **+3.87 average EM**
and **+2.73 average F1** with a **T5** student under 16-shot distillation.

## Method

<p align="center">
  <img src="pipeline.png" alt="COFFEQA pipeline" width="100%">
</p>

Given the full dataset $\mathcal{D}$, we:

1. **Sub-select** a factual subset $\mathcal{D}_f$ from the full dataset $\mathcal{D}$.
2. Pass $\mathcal{D}_f$ through a **counterfactual generator** to produce a
   counterfactual dataset $\mathcal{D}_{cf}$, where the context (and therefore
   the answer) has been minimally perturbed relative to the original factual
   instance.
3. Score every counterfactual with the **Context Following Difficulty (CFD)
   Calculator**, which uses a teacher LLM to estimate how strongly a
   counterfactual instance forces context-following behavior, and sort
   $\mathcal{D}_{cf}$ by this score.
4. Select the **top-$k$ counterfactuals** $\mathcal{D}_{cf,k}$ (plus their
   paired $k$-factuals $\mathcal{D}_{f,k}$) as the few-shot distillation set.
5. Run **knowledge transfer**: both the teacher LLM and the pre-trained student
   LLM are conditioned on $\mathcal{D}_{cf,k} + \mathcal{D}_{f,k}$, and the
   student is distilled toward the teacher's behavior on these pairs, producing
   the **distilled student LLM**.

## Installation

<!-- TODO: confirm these match your actual repo/env -->
```bash
git clone https://github.com/KavindaDulhan/coffeqa.git
cd coffeqa
conda create -n coffeqa python=3.10 -y
conda activate coffeqa
pip install -r requirements.txt
```

## Quickstart

<!-- TODO: replace with your actual entry-point scripts and arguments -->
```bash
# 1. Generate counterfactuals from a factual QA dataset
python scripts/generate_counterfactuals.py \
    --dataset natural_questions \
    --output_dir data/cf/

# 2. Score and sort counterfactuals with the CFD calculator
python scripts/compute_cfd.py \
    --teacher_model <teacher-llm-name> \
    --cf_dir data/cf/ \
    --output data/cf_sorted.jsonl

# 3. Distill the student on the top-k counterfactual + factual pairs
python scripts/distill.py \
    --teacher_model <teacher-llm-name> \
    --student_model t5-base \
    --k_shots 16 \
    --cf_sorted data/cf_sorted.jsonl \
    --output_dir checkpoints/coffeqa_t5/
```

## Repository structure

<!-- TODO: update to match your actual layout -->
```
coffeqa/
├── scripts/
│   ├── generate_counterfactuals.py   # counterfactual generator
│   ├── compute_cfd.py                # Context Following Difficulty calculator
│   └── distill.py                    # few-shot teacher → student distillation
├── data/                             # datasets and generated counterfactuals
├── configs/                          # experiment configs
├── pipeline.png                      # method figure (above)
└── README.md
```

## Results

| Dataset | Student | Shots ($k$) | ΔEM vs. factual-only | ΔF1 vs. factual-only |
|---|---|---|---|---|
| NaturalQuestions | T5 | 16 | **+3.87** | **+2.73** |
<!-- TODO: add the remaining in-domain / out-of-domain rows from your full results table -->

We evaluate on five in-domain and three out-of-domain QA benchmarks across two
LLM families; COFFEQA improves over factual-only few-shot distillation across
the board, with the gains generalizing to both factual and counterfactual test
splits. <!-- TODO: list the specific datasets and LLM families here once finalized -->

## Citation

<!-- TODO: replace with the final citation once accepted / an arXiv ID is available -->
```bibtex
@misc{coffeqa2026,
  title        = {COFFEQA: Counterfactual-guided Few-shot Distillation in Open-book Question Answering},
  author       = {TODO: author list},
  year         = {2026},
  note         = {Manuscript under review},
}
```

## License

<!-- TODO: pick a license and add a LICENSE file, e.g. MIT -->
This project is released under the [MIT License](LICENSE).

## Acknowledgments

<!-- TODO: funding sources, advisors, compute grants, etc. -->
