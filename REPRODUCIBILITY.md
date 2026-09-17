# Reproducibility

## Environment

```bash
conda env create -f environment.yml
# or
pip install -r requirements.txt
```

Developed and run on Kaggle's free-tier T4 (with `constriction` for the ANS
range coder). No dataset download beyond the 24 Kodak images.

## Data

Upload the 24 `kodim*.png` Kodak images as a Kaggle Dataset once and attach
it — the notebook searches `/kaggle/input` for them first so runs work with
internet disabled.

## Running

**Run All, then leave it.** No inputs to paste, no decisions to make: cell 8
picks the quantisation-step rate axis automatically from a single calibration
image, and cell 10 (`MAIN SWEEP`) starts sweeping. A time-budget guard
(`TIME_BUDGET_HOURS`) stops cleanly before the session is killed.

Every point is written atomically to `/kaggle/working/coin_lowrank/` as it
completes. To continue in a later session:

1. **Commit** the notebook (`Save Version -> Save & Run All`).
2. In the new session, **`Add Data -> Notebook Output -> this notebook`**
   and Run All again.

Completed points are skipped, the saved calibration is reused (a resumed
session that recalibrated to different steps would produce results under
different keys that couldn't be merged), and the sweep continues where it
left off. Repeat until cell 10 prints `SESSION COMPLETE`. Jobs run
breadth-first over trials, so exhausting the time budget mid-run still leaves
a complete single-trial experiment over all 24 images rather than a handful
of images run to completion and the rest untouched.

## Determinism

Each point is seeded deterministically from its own key
(image, config, quantisation step, trial index), so any individual point is
exactly reproducible. Diverged runs are retried once with a different seed
and flagged, not silently dropped or averaged in.

## Known limitations to disclose alongside any reported numbers

- **Resolution.** Default `MAX_SIDE=256` is not comparable to published Kodak
  results at 768×512. Either say so explicitly or set `MAX_SIDE=None` and pay
  the extra compute.
- **Entropy model.** A per-tensor factorised Gaussian is the simplest useful
  prior. A context model would compress further — that's future work, not a
  flaw in what's measured here.
- **Qualitative comparison (cell 22)** retrains 4 models per figure (the main
  sweep stores only scalars), roughly 15–20 minutes.
- **Perona–Malik post-filter (cell 24)** selects its strength by checking
  PSNR against the original image, which a real decoder does not have. As
  written it is an oracle upper bound, not a deployable filter — report it as
  one or transmit the chosen strength (2 bits) as a real result.
