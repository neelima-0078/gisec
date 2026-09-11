# Data Poisoning Detection Pipeline

GISEC School of Cybersecurity Challenge submission — a working pipeline that poisons a CIFAR-10 training set two different ways, detects the poisoned samples before training completes, and proves that removing them and retraining actually neutralizes the attack.

**Team:** Neelima, Shraddha, Jannat

Full write-up: see `gisec-submission-report.pdf` in this repo (or linked from the submission portal).

---

## What this does

1. Trains a small CNN on a clean 10,000-image CIFAR-10 subset as a baseline.
2. Injects two poisoning attacks at a 5% poison rate:
   - **Label flipping** — relabels 5% of one class's images to another class, pixels untouched.
   - **Backdoor trigger** — stamps a 3x3 pixel patch onto 5% of images and relabels them to a target class, teaching the model a hidden shortcut.
3. Confirms both attacks are stealthy (poisoned models score within ~2 points of clean baseline on standard test accuracy) while actually working (89.5% attack success rate when the trigger fires).
4. Runs two independent detectors:
   - Loss-based outlier detection (catches label-flip poisoning).
   - Activation clustering (catches backdoor poisoning).
5. Quarantines flagged samples with a recorded reason for each, retrains from scratch on what's left, and shows the attack collapses to near random-chance (7.4% ASR, down from 89.5%).
6. Stress-tests the whole cycle across the brief's specified 1-10% poison-rate range, including an honest look at where detection breaks down (~10% poison rate).

## Results summary

| Metric | Value |
|---|---|
| Clean baseline accuracy | 60.05% |
| Attack Success Rate before cleaning | 89.48% |
| Detector 1 (loss-based) recall / FPR | 72.9% / 14.7% |
| Detector 2 (activation clustering) recall / FPR | 79.0% / 0.0% |
| Post-cleaning accuracy | 59.09% |
| Post-cleaning ASR | 7.37% |

Full metrics, the stress-test table, and limitations are in the PDF report.

## How to run

**Option A — Google Colab (recommended, matches how this was built):**
1. Open `gisec_poisoning_pipeline.ipynb` in this repo.
2. Click "Open in Colab" (or upload the file to https://colab.research.google.com manually).
3. `Runtime → Change runtime type → GPU`.
4. `Runtime → Run all`. Takes roughly 6-10 minutes for the core pipeline; the poison-rate stress test cell alone takes an additional 15-25 minutes since it trains 10 models.
5. No manual dataset download needed — CIFAR-10 is fetched automatically on first run.

**Option B — Local Jupyter / VS Code:**
```bash
pip install torch torchvision numpy pandas matplotlib scikit-learn
jupyter notebook gisec_poisoning_pipeline.ipynb
```
Run all cells in order, top to bottom. A GPU is recommended but not required — training will just be slower on CPU.

## Repository structure

- `GISEC_poisoning_pipeline.ipynb` — full pipeline: baseline, attacks, detectors, cleaning, stress test
- `gisec-submission-report.pdf` — 5-page report (objective, solution, validation, results)
- `README.md` — this file

## Notes on reproducibility

The notebook was run twice independently from a cold restart; results were consistent (ASR before/after cleaning: 89.61%/9.56% and 89.48%/7.37% across the two runs). If you re-run this yourself, expect numbers within a similar range rather than an exact match, since training involves randomness (weight initialization, data shuffling, minibatch order).

## Known limitations

- Activation clustering assumes poisoned samples are a minority within the target class. Detection reliability drops sharply as poison rate approaches 10%, where this assumption breaks down.
- An unsupervised version of both detectors (not relying on ground-truth poison labels to pick thresholds) was attempted but not included as a claimed result — early numbers were inconsistent enough that we didn't want to present them as reliable.
- Detectors were tuned against a fixed trigger pattern (fixed patch, fixed position); a trigger that varies across poisoned samples would likely be harder to isolate.
