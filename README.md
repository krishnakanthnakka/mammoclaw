<div align="center">

<img src="https://raw.githubusercontent.com/krishnakanthnakka/mammoclaw/gh-pages/images/breasticon.png" width="90" alt="MammoClaw">

# MammoClaw

### Towards Skill-Evolving Agent Harness for Breast Cancer Mammography Analysis

**[Krishna Kanth Nakka](https://scholar.google.com/citations?hl=en&user=g_21RKoAAAAJ)**

DeepBreath Workshop, **MICCAI 2026**

[![Project Page](https://img.shields.io/badge/Project-Page-f68946)](https://krishnakanthnakka.github.io/mammoclaw/)
[![Paper](https://img.shields.io/badge/Paper-PDF-b3261e)](https://krishnakanthnakka.github.io/mammoclaw/docs/main.pdf)
[![Evolved Skills](https://img.shields.io/badge/Evolved-Skills-0f766e)](https://krishnakanthnakka.github.io/mammoclaw/#skills)
[![Trajectories](https://img.shields.io/badge/Agent-Trajectories-d2601f)](https://krishnakanthnakka.github.io/mammoclaw/trajectories/index.html)

</div>

---

> ### 🚧 Code release
>
> **The codebase will be released soon.** The agent harness, mammography tool suite, and skill-evolution
> pipeline are being prepared for release. In the meantime, the [paper](https://krishnakanthnakka.github.io/mammoclaw/docs/main.pdf),
> all [evolved skills](https://krishnakanthnakka.github.io/mammoclaw/#skills), and
> [full agent trajectories for 100 examples](https://krishnakanthnakka.github.io/mammoclaw/trajectories/index.html)
> are already available on the project page.
>
> ⭐ Star or watch this repository to be notified when the code lands.

---

## Overview

<div align="center">
<img src="https://raw.githubusercontent.com/krishnakanthnakka/mammoclaw/gh-pages/media/overview_v5.png" width="100%" alt="MammoClaw overview">
</div>

*Top:* the agent alternates between reasoning, calls to deterministic mammography tools, and observations before
producing a final prediction. *Bottom:* the offline skill-evolution workflow, where failed trajectories on a labeled
reference set are distilled into reusable skills that are retrieved in later runs.

## Abstract

In this work, we explore **MammoClaw**, a training-free agent framework that leverages frozen MLLMs for mammography
analysis. To support agentic investigation, we equip the agent with lightweight mammography-specific tools for targeted
image analysis, including ROI, paired-view, and contralateral-breast examination. MammoClaw iteratively gathers
evidence through these tools, while skill evolution enables non-parametric adaptation by transforming failed
trajectories into reusable guidance for later runs. We evaluate the framework on BI-RADS assessment and breast density
estimation tasks. In our experiments, we find that tools alone do not reliably improve performance, whereas evolved
skills can improve tool-use behavior and performance in some settings. Beyond these results, MammoClaw enables
transparent inspection of evidence acquisition, tool interactions, and failure modes, facilitating the analysis and
auditing of agent behavior. We view this work as an exploratory study of training-free, self-evolving agentic
approaches for mammography and hope it provides a concrete starting point for future work on mammography-specific
tools and self-evolution mechanisms.

## Highlights

- **Framework.** A training-free agent harness for agentic mammography analysis with frozen MLLMs, deterministic
  mammography tools, and an automated offline skill-evolution workflow. No weight updates.
- **Tool suite.** Lightweight, deterministic, model-free tools for retrieving task knowledge, inspecting regions of
  interest, and comparing paired and contralateral views — without auxiliary segmentation or detection networks.
- **Skill evolution.** Failed trajectories are distilled into reusable reasoning guidance, retrieved in subsequent
  cases, enabling experience-driven adaptation without touching the backbone.
- **Evaluation.** Tool augmentation alone does not necessarily help (BI-RADS macro-F1 0.108 → 0.106), while skill
  evolution improves both performance (→ 0.148) and tool-use behavior.
- **Auditability.** Every tool call and piece of evidence behind a decision is logged and inspectable.

## Method

| Step | Description |
|---|---|
| **1. Agent orchestration** | Following ReAct, the agent alternates between reasoning, tool invocation, and observation. At each step it either produces a final prediction or selects a tool call. |
| **2. Skill retrieval** | Before the first reasoning step, a lightweight retriever selects relevant skills, which are prepended to the context. |
| **3. Mammography tools** | Deterministic, model-free tools return crops, side-by-side comparisons, and structured measurements. |
| **4. Collecting failures** | The agent is run on a labeled reference set and failed trajectories are collected. |
| **5. Distilling skills** | A teacher model analyzes the failed trajectories jointly and proposes candidate skills, which are merged into a reusable skill bank. |

## Mammography Tool Suite

All tools are deterministic and model-free. One representative call per tool is shown on the
[project page](https://krishnakanthnakka.github.io/mammoclaw/#tools), drawn from logged trajectories.

| Tool | Category | What it does |
|---|---|---|
| `inspect_current_example` | Task context | Returns current example metadata (laterality, view, age, paired-view availability). |
| `retrieve_knowledge` | Task context | Retrieves task-specific domain knowledge, e.g. BI-RADS category definitions (0–6, incl. 4A/4B/4C) or density-band definitions. |
| `inspect_mammogram_roi` | Single-image inspection | Crops and enlarges a region using normalized 0–1000 grid coordinates, with optional contrast enhancement. |
| `measure_finding_size` | Single-image measurement | Measures width and height of a finding by drawing annotated measurement bars on a contextual crop. |
| `inspect_paired_mammogram_view` | Cross-view comparison | Returns the paired CC and MLO projections of the same breast side by side. |
| `inspect_contralateral_breast` | Cross-breast comparison | Compares the current breast with the contralateral breast from the same subject and view. |
| `estimate_breast_density` | Single-image density | Estimates fibroglandular density via dual Otsu thresholding, with a caveat against assigning A/B/C/D from the percentage alone. |
| `measure_image_sharpness` | Single-image quality | Scores technical sharpness via Laplacian variance and Tenengrad gradient energy. |
| `compute_tissue_statistics` | Single-image texture | Intensity mean, standard deviation, entropy, and skewness over the breast foreground. |

## Results

**Tools alone vs. evolved skills** — adding the tool suite leaves BI-RADS macro-F1 essentially unchanged, while
evolved skills on top of tools give a statistically significant gain over both the no-tool and tools-only settings
(paired bootstrap, Holm-corrected *p* < 0.001). Backbone: frozen Qwen3.5-35B-A3B. Dataset: KAU-BCMD.

| Method | Tools | Skills | Macro-F1 [95% CI] |
|---|:--:|:--:|---|
| Baseline | ✗ | ✗ | 0.108 [0.103, 0.113] |
| MammoClaw | ✓ | ✗ | 0.106 [0.093, 0.122] |
| **MammoClaw** | ✓ | ✓ | **0.148 [0.121, 0.180]** |

**Statistical significance across both tasks** — skill evolution improves macro-F1 on both tasks, whereas adding tools
alone does not lead to a significant change.

| Task | Comparison | ΔMacro-F1 | Bootstrap *p* | McNemar *p* |
|---|---|---|---|---|
| BI-RADS | No tools → Tools | −0.002 | 0.70 | 0.0017 |
| BI-RADS | No tools → Skills | +0.040 | <0.001 | <0.001 |
| BI-RADS | Tools → Skills | +0.043 | <0.001 | <0.001 |
| Density | No tools → Tools | +0.019 | 0.70 | 0.062 |
| Density | No tools → Skills | +0.093 | 0.18 | <0.001 |
| Density | Tools → Skills | +0.074 | 0.017 | 0.062 |

## Evolved Skills

Skills are distilled from failed reasoning trajectories on the reference set. Rather than encoding explicit
tool-selection policies, they capture reusable reasoning strategies. The full text of each skill is on the
[project page](https://krishnakanthnakka.github.io/mammoclaw/#skills).

1. `verify-suspicious-findings-with-multi-view-comparison`
2. `systematic-quadrant-inspection-for-subtle-findings`
3. `tool-execution-before-conclusion`
4. `calibrate-birads-using-feature-checklist`
5. `rule-out-artifact-with-contralateral-comparison`
6. `distinguish-benign-from-probably-benign-calcifications`
7. `avoid-characterizing-findings-without-tool-confirmation`
8. `structured-reasoning-grounded-in-tool-outputs`
9. `use-uncertainty-resolution-sequence`

## Agent Trajectories

Full trajectories for **100 examples**, with every reasoning step, tool call, and tool output, are browsable here:

**➡️ [Browse all trajectories](https://krishnakanthnakka.github.io/mammoclaw/trajectories/index.html)** (large page, ~60 MB)

A [walkthrough of one suspicious BI-RADS 4 case](https://krishnakanthnakka.github.io/mammoclaw/#trajectory) shows the
agent localizing architectural distortion with an ROI call, confirming it persists across projections with the paired
view, and ruling out a symmetric normal variant with the contralateral comparison.

## Limitations

- **Outcome-based, not process-based, evaluation.** Metrics score only the final prediction, not whether the reasoning
  and evidence gathering were clinically sound.
- Evaluation covers two tasks, a single frozen backbone, and one dataset per task.
- Evolved skills come from a labeled reference set and have **not been validated by clinical experts**.
- The effects of the system prompt, tool descriptions, teacher LLM, and retrieval mechanism have not been ablated.
- Skill evolution uses a **single iteration** on a 100-example reference set.

## Collaboration

I am looking for collaborators to extend this work, in particular on **multi-round skill evolution**, and for feedback
from clinical researchers on the evolved skills. Details on the
[project page](https://krishnakanthnakka.github.io/mammoclaw/#collaboration).

> **Intended use.** Research artifact only — not a medical device, not for diagnosis or clinical decision-making.

## Citation

```bibtex
@InProceedings{Nakka_2026_MICCAI,
  author    = {Nakka, Krishna Kanth},
  title     = {MammoClaw: Towards Skill-Evolving Agent Harness for Breast Cancer Mammography Analysis},
  booktitle = {Proceedings of the Deep Breath Workshop on AI and Imaging for Diagnostic and Treatment Challenges in Breast Care, MICCAI 2026},
  year      = {2026},
}
```
