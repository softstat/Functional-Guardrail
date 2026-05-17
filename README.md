# Functional Guardrails

**Layer-wise Trajectory Analysis for Adaptive-Robust Defense of Vision-Language Models**

---

## 1. Overview

This project investigates whether **layer-wise hidden state trajectories** of Vision-Language Models (VLMs), viewed as multivariate functional data, carry sufficient signal to detect adversarial multimodal inputs in a way that remains robust under an *adaptive* attacker who knows the detector.

### Research question

> Given a VLM processing a multimodal input $(I, T)$ (image, text), can we treat the layer-wise hidden states as a multivariate functional object and detect adversarial inputs by comparing trajectory **shape** rather than activation magnitude — and does shape-based detection survive an adaptive attacker who optimizes against it?

### Core hypotheses

- **H1**: For benign vs adversarial multimodal inputs, layer-wise hidden state trajectories differ in *shape* (SRVF distance), not only in magnitude. The shape difference is statistically significant and yields detection AUC > 0.80.

- **H2**: SRVF's **re-parametrization invariance** means an attacker cannot evade detection by relocating the attack into a different layer subset. Adaptive attackers using PGD/GCG with detector-aware objectives achieve materially lower attack success than against static-representation detectors (HiddenDetect, RCS).

- **H3**: Multivariate functional PCA (MFPCA) over text/image/cross-modal trajectories captures the *cross-modal co-variation pattern* — benign inputs show coordinated trajectory evolution across modalities, while adversarial inputs disrupt this coordination. This modality-attribution capability is novel.

### Why this might be a contribution

Existing internal-representation defenders for VLMs (HiddenDetect, RCS/MCD/KCD, PIShield for LLM-only) analyze *static* hidden states at safety-critical layers. Existing trajectory analyses (TaT, Transformer Alignment, Bridging the Dimensional Chasm) use LSTM, Jacobian, or intrinsic-dimension tools. To the best of our knowledge, no published work applies FDA tools (MFPCA + SRVF) to VLM hidden state trajectories for adversarial detection. SRVF's re-parametrization invariance provides a *formal* basis for adaptive robustness claims that are otherwise empirical-only.

---

## 2. Status

**Current phase**: Month 0 — pre-lit-review. Plan and infrastructure setup in progress. No experiments yet.

Major decisions still open:
- Final base VLM (LLaVA-1.5 7B is the current default)
- Cloud GPU provider
- Whether to use R `fdasrvf` for prototyping or commit fully to Python from the start

---

## 3. Threat Model (working draft)

We target the strongest practical adversary; weaker adversaries follow automatically.

| Dimension | Setting |
|---|---|
| Attacker knowledge of VLM | White-box (open-weight model, weights known) |
| Attacker knowledge of defender | Full — knows MFPCA basis and SRVF metric |
| Query budget | Effectively unlimited (gradient access) |
| Attack modality | Both image and text, including coordinated bi-modal |
| Attack goal | Jailbreak — induce a harmful or policy-violating generation |
| Detector deployment | At inference time, on the VLM's own forward pass |

This is the Carlini-style threat model — the attacker has every advantage except control of the detector itself. If Functional Guardrails survives here, it survives in deployment.

---

## 4. Reading List (18 papers)

Read in order. One-page note per paper. Total time: 8 weeks.

### Group A — ML Security foundations (Week 1–2)
1. Goodfellow, Shlens, Szegedy (2015) — *Explaining and Harnessing Adversarial Examples*. ICLR.
2. Madry et al. (2018) — *Towards Deep Learning Models Resistant to Adversarial Attacks*. ICLR. PGD adversarial training.
3. Athalye, Carlini, Wagner (2018) — *Obfuscated Gradients Give a False Sense of Security*. ICML.
4. Carlini et al. (2019) — *On Evaluating Adversarial Robustness*. arXiv:1902.06705. **Mandatory.**

### Group B — LLM Security (Week 3–4)
5. Greshake et al. (2023) — *Not what you've signed up for*. AISec workshop.
6. Zou et al. (2023) — *Universal and Transferable Adversarial Attacks on Aligned Language Models*. GCG attack.
7. Hackett et al. (2025) — *Bypassing LLM Guardrails*. ACL LLMSec.
8. Zhan et al. (2025) — *Adaptive Attacks Break Defenses Against IPI on LLM Agents*. arXiv:2503.00061.

### Group C — VLM Security (Week 5–6)
9. Qi et al. (2024) — *Visual Adversarial Examples Jailbreak Aligned Large Language Models*. AAAI.
10. Bagdasaryan et al. (2023) — *Abusing Images and Sounds for Indirect Instruction Injection in Multimodal LLMs*. arXiv:2307.10490.
11. Shayegani, Dong, Abu-Ghazaleh (2024) — *Jailbreak in Pieces*. ICLR.
12. Yang et al. (2025) — *HiddenDetect*. arXiv:2502.14744. **Closest baseline.**
13. Liu et al. (2025) — *Rethinking Jailbreak Detection of LVLMs with Representational Contrastive Scoring* (RCS/MCD/KCD). arXiv:2512.12069. **SOTA baseline.**

### Group D — Trajectory analysis (Week 7)
14. *Truth as a Trajectory (TaT)* — arXiv:2603.01326 (2026).
15. *Transformer Alignment in Large Language Models* — arXiv:2407.07810 (2024).
16. *Bridging the Dimensional Chasm* — arXiv:2503.22547 (2025).
17. *From Latent Signals to Reflection Behavior* — arXiv:2602.01999 (2026).

### Group E — FDA on neural networks (Week 8)
18. *Nonlinear Functional PCA Using Neural Networks (nFunNN)* — arXiv:2306.14388 (2023).

### Per-paper note template
- What problem
- What method
- What limitation
- Relation to our work (complement / baseline / contrast)

---

## 5. Twelve-Month Plan

| Month | Phase | Deliverable |
|---|---|---|
| 1–2 | Lit review (18 papers) | One-page notes; contribution statement finalized |
| 3 | Infrastructure setup | Cloud GPU setup; LLaVA-1.5 7B running; hidden-state extraction pipeline; FDA toolchain in Python |
| 4 | Pilot (H1 test) | AUC for benign vs MM-SafetyBench using SRVF-distance scoring; **GO/NO-GO decision** |
| 5–6 | Method | Detector class defined; cross-VLM transfer (LLaVA → Qwen2-VL) |
| 7–8 | Static evaluation | Comparison vs HiddenDetect, RCS, MCD, KCD on standard benchmarks |
| 9–10 | Adaptive evaluation | Custom adaptive PGD/GCG against Functional Guardrails; H2 test |
| 11 | Paper draft | First full draft; internal review |
| 12 | Paper submission | Polish; submit to USENIX Security 2027 / IEEE S&P 2027 |

### Decision points
- **End of Month 4**: H1 pilot result. AUC > 0.80 → proceed. AUC < 0.65 → re-design or switch to plan B.
- **End of Month 10**: H2 adaptive evaluation. If detector collapses under adaptive attack → publish as honest negative result + lessons.

---

## 6. Compute and Cloud Budget

### Recommended base model
**LLaVA-1.5 7B** — most baseline coverage (HiddenDetect, RCS evaluated on LLaVA), ~16 GB VRAM in FP16, fits A100 40 GB easily.

Alternative for cross-VLM transfer experiments: **Qwen2-VL 7B**, **LLaVA-NeXT 7B**.

### GPU cost estimate (cloud)

| Phase | GPU needed | Hours (approx) | Cost @ $1.10/hr (A100 40GB) |
|---|---|---|---|
| Month 3 (setup, smoke tests) | A100 40 GB | 100 | $110 |
| Month 4 (pilot) | A100 40 GB | 200 | $220 |
| Month 5–6 (method) | A100 40 GB | 500 | $550 |
| Month 7–8 (static eval) | A100 40 GB | 600 | $660 |
| Month 9–10 (adaptive eval) | A100 80 GB | 1,500 | ~$3,000 (80GB ~$2.00/hr) |
| Month 11–12 (paper revisions) | A100 40 GB | 200 | $220 |
| **Total** | | ~3,100 hrs | **~$4,800** |

This is a single-researcher solo budget. Numbers will swell if cross-VLM and ablation breadth expand.

### Cloud provider candidates
- **Lambda Labs** — cheapest A100 ($1.10/hr); reasonable availability
- **RunPod** — flexible spot pricing
- **Vast.ai** — cheapest but variable reliability
- **Google Colab Pro+** — useful for early prototyping only; not suitable for adaptive eval

### Recommended pattern
1. Use a **persistent volume** (~500 GB) for model weights and intermediate activations. Cheaper than re-downloading each session.
2. Spin up the GPU instance only for active work. Detach when not running.
3. For Month 9–10 long-running adaptive attack jobs, consider reserved/committed-use discounts.

---

## 7. Software Stack

### Core dependencies
- Python 3.11
- PyTorch 2.4+ with CUDA 12.x
- `transformers` (HuggingFace) for LLaVA / Qwen2-VL
- `accelerate` for multi-GPU if needed
- `bitsandbytes` for quantization (if memory pressure)

### Hidden-state extraction
- `nnsight` (preferred — clean API for activation extraction) **or**
- `transformer_lens` (more mature, but VLM support is limited)

If neither supports our exact VLM, fall back to PyTorch forward hooks directly. This is straightforward but more code.

### Functional data analysis
- **Python**: `scikit-fda` (general FDA), or custom MFPCA implementation
- **R**: `fdasrvf` (Srivastava's official package — gold standard for SRVF), `fda`, `MFPCA`
- **Bridge**: `rpy2` to call R from Python, OR export NumPy arrays as CSV and run R offline

**Decision**: Prototype in R for first MFPCA/SRVF experiments (Month 4), then port to Python (Month 5) for full integration with PyTorch pipeline. Rationale: the R `fdasrvf` package is more battle-tested, and Month 4 is a GO/NO-GO point where correctness matters more than pipeline speed.

### Attack frameworks (Month 9+)
- `nanogcg` — clean GCG implementation
- Custom PGD on images (write from scratch using PyTorch autograd)
- `torchvision.transforms` for image preprocessing

### Evaluation
- `scikit-learn` for ROC/AUC
- `seaborn`, `matplotlib` for visualization
- Custom statistical tests (permutation tests on SRVF distances)

---

## 8. Data

### Benign multimodal data
- **COCO Captions** (2017 split) — standard image-caption pairs. ~5,000 sample subset.
- **VQA-v2** — image + question pairs.

### Adversarial multimodal data
- **MM-SafetyBench (Liu et al. 2024, ECCV)** — primary attack benchmark. 13 unsafe scenarios.
- **VLSafe (Chen 2024)** — VLM jailbreak benchmark.
- **HADES (Li 2024)** — image-side hidden adversarial.
- **Visual adversarial (Qi 2024)** — gradient-based image adversarial.
- **MM-PoisonRAG (Ha et al. 2025)** — if multimodal RAG variant of the threat model is included.

### Generated data (Month 9–10)
- Custom adaptive PGD images: optimize $\delta$ on image such that (a) jailbreak succeeds, (b) SRVF distance to benign manifold is minimized
- Custom adaptive GCG suffixes on text: same dual objective

### Storage requirements
- VLM weights: ~30 GB (FP16)
- Benign + attack image corpus: ~50 GB
- Extracted hidden states (32 layers × ~500 positions × 4096 dim × float16 × ~10,000 samples): **~2.6 TB raw**.
- **Strategy**: do *not* store all layers for all samples. Subsample positions, store as compressed `.npz`. Plan for ~200 GB working storage.

### Data layout (planned)
```
data/
  raw/
    coco/             # original images
    mm_safety_bench/  # attack corpus
  cache/
    embeddings/       # vision encoder outputs
    hidden_states/    # layer-wise activations, compressed npz
  processed/
    trajectories/     # functional-data form
```

---

## 9. Pilot Experiment (Month 4)

This is the GO/NO-GO point. Single most important experiment of Month 1–4.

### Setup
- VLM: LLaVA-1.5 7B
- Benign: 100 COCO image + caption samples
- Attack: 100 MM-SafetyBench samples
- Hidden states: all 32 transformer layers, last-token position for now

### Procedure
1. Forward pass each sample, hook every layer, save hidden state $h_\ell \in \mathbb{R}^{4096}$ for $\ell = 1, \ldots, 32$
2. For each sample, construct trajectory $\mathcal{T}_i = (h_1, h_2, \ldots, h_{32})$ as a discrete functional sample on $[0, 1]$ (re-parameterized layer index)
3. Compute pairwise SRVF distances within and across benign/attack groups
4. Fit MFPCA on the combined sample; project each $\mathcal{T}_i$ onto first $K$ PCs
5. Logistic regression on MFPCA scores → benign/attack classifier
6. Report ROC AUC, confusion matrix, per-PC explained variance

### Success criterion
**AUC > 0.80** → proceed to Month 5 method build.
**AUC 0.65–0.80** → marginal; investigate trajectory of intermediate positions, not just last token.
**AUC < 0.65** → re-design (e.g., use multivariate FPCA over multiple positions per sample; or pivot to plan B).

### Notes
- This pilot uses **single-position trajectories** for simplicity. Full method (Month 5+) uses **multivariate trajectories** over text/image/cross-modal positions — this is where H3 lives.
- The pilot is in **R** using `fdasrvf` to keep correctness simple. Python port comes in Month 5.

---

## 10. Risks and Plan B

| Risk | Likelihood | Plan B |
|---|---|---|
| H1 fails (no shape separation) | Medium | Pivot to *attention* trajectory analysis instead of hidden-state trajectory |
| Adaptive attack breaks detector (H2 fails) | Medium-High | Publish honest negative result + analysis of *why* SRVF invariance wasn't enough |
| Cross-VLM transfer fails | Medium | Restrict scope to single-model paper (still publishable, lower-tier venue) |
| Cloud cost overrun | Medium | Reduce ablation breadth; single attack family for adaptive eval |
| Reviewer doesn't understand FDA | High | Write tutorial section in paper; this is also a contribution opportunity |

Plan B exists in writing for a reason: ML security is a field where ~70% of defense papers are broken within a year. If our detector breaks under adaptive evaluation, that result is *itself* publishable as a contribution to the field's honest evaluation culture (Athalye 2018 lineage).

---

## 11. Target Venues

| Venue | Deadline (approx) | Fit | Notes |
|---|---|---|---|
| USENIX Security 2027 | Feb / Jun 2026 | Strong | Top fit for security focus |
| IEEE S&P 2027 | Jun 2026 | Strong | Top tier |
| NDSS 2027 | Jul 2026 | Strong | Top tier |
| CCS 2026/2027 | May / Jan | Good | |
| NeurIPS 2026 | May 2026 | Possible | ML angle |
| CVPR 2026 | Nov 2025 | Possible | Multimodal angle (deadline tight) |
| AISec workshop 2026 | Aug 2026 | Fallback | Shorter paper |
| ACL LLMSec 2026 | Apr/May 2026 | Fallback | |

Primary: USENIX Security 2027. Submission target Feb 2026 cycle (= Month 12 deadline).

---

## 12. References (working bibliography)

To be filled in during Month 1–2 reading. Use BibTeX file `refs.bib`, one entry per paper read.

---

## 13. Open Questions

These need answers before Month 3:

1. Single-position or multi-position trajectory in the pilot? (Currently: single, last-token. Reconsider after Week 7 reading.)
2. Are LLaVA and Qwen2-VL architecturally similar enough for SRVF cross-model transfer? (Investigate during Group C reading.)
3. R-Python bridge or commit fully to Python? (Decide at end of Month 2.)
4. Is the threat model in §3 the right starting point, or should we begin with a weaker adversary and escalate? (Discuss with advisor / collaborator if any.)

---

## 14. Honest Acknowledgments

- This plan is a single-researcher solo project. A 1-year cycle is tight; missing the H1 GO/NO-GO point at Month 4 by even a month compounds across the rest of the schedule.
- The author's strength is functional data analysis (background in MFPCA + SRVF on gait kinematics and survival analysis). VLM internals, mechanistic interpretability, and adversarial ML are *new* domains and the reading list is mandatory, not optional.
- The novelty claim — "FDA tools on VLM trajectories" — rests on a literature search current to early 2026. Re-check before submission. The field is moving fast.

---

*Project lead: BLDA Lab*
*Started: 2026*
*Plan version: 0.1 — Month 0 draft*
