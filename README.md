# Entropy-constrained low-rank COIN

Low-rank factorised SIREN image codec, evaluated against a full-rank
entropy-constrained COIN baseline on all 24 Kodak images. The rate model is a
learned per-tensor Gaussian prior handed directly to an ANS range coder, so the
bitrate the loss minimises is the bitrate the coder actually spends —
estimated and realised bpp are checked to agree, not just plausible.

## Result

Per-image paired BD-rate against the full-rank baseline, mean ± SE over 10
held-out images:

| Method | Params | BD-rate | Better on |
| --- | --- | --- | --- |
| COIN baseline | — | — (anchor) | — |
| Low-rank r=16 | 13,443 | **−60.40% ± 1.65** | 10/10 |
| Low-rank r=32 | 25,731 | −35.18% ± 3.07 | 10/10 |
| Low-rank r=48 | 38,019 | −13.24% ± 4.01 | 8/10 |

Low-rank factorisation wins at the low-bitrate end and loses it back as rank
grows toward full-rank parity — the win is a regime, not a universal one. The
codec does not beat JPEG/WebP/JPEG2000; that's the known state of single-image
INR compression, and the comparison here is within the INR family, not against
standard codecs.

## What made the numbers honest

The rate axis is the quantisation step, not the loss's rate weight — sweeping
lambda instead left bitrate flat and non-monotonic, because the learned prior's
sigma tracks the weight scale rather than an absolute rate. Getting bands that
actually overlap (required for BD-rate to be valid) took fixing that axis,
flooring sigma at half a quantiser bin so the rate term can't claim weights are
free, coding every parameter including biases, and refusing to compute BD-rate
at all when curves lack genuine shared support. Full account of what broke and
what fixed it is in the notebook's changelog cell.

## Notebook

[`coin_lowrank_kodak_eval.ipynb`](coin_lowrank_kodak_eval.ipynb) — self-contained,
resumable across free-tier Kaggle sessions (checkpoints to disk, merges prior
session output, stops cleanly on a time budget). Run All; no inputs to paste,
the rate axis is calibrated automatically per session.

Caveats disclosed in the notebook itself: results are at 256px (not directly
comparable to published 768×512 Kodak numbers), divergent runs are retried and
flagged rather than silently dropped, and the entropy model is a simple
per-tensor factorised Gaussian — a context model would compress further.

Research internship, NIT Calicut. Under review, *IEEE Signal Processing
Letters*.
