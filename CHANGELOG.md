# Changelog

Method versions are tracked via `METHOD_TAG` in the notebook's checkpoint
filenames, so results from different versions can never be silently averaged
together. This log follows the version history documented in the notebook's
own changelog cell.

## v5 (current)

- **Rate axis fixed to quantisation step, not lambda.** With a learned prior,
  sigma tracks the weight scale, so sweeping lambda left bitrate flat and
  non-monotonic (measured on kodim01: a 10^6 sweep of lambda held bpp at ~27
  and non-monotonic). Quantisation step is now the swept variable.
- **Lambda fixed at 1e-4** (was 1e-2 in v4). At 1e-2 the rate penalty drove
  sigma to its floor, after which the step stopped affecting rate at all —
  a 16x cliff between adjacent steps, then a dead flat region, with 25% of
  runs diverging. At 1e-4 the rate falls smoothly with no floor hit.
- **Calibration rescales the whole ladder by one factor**, never interpolates
  a grid. v4's interpolation produced two near-identical steps (0.00388 and
  0.003881), leaving only 3 usable rate points and every BD-rate `n/a`.
- **Sigma floored at half a quantiser bin.** Without the floor, the rate term
  could push sigma below the step so the loss read weights as nearly free
  while the coder paid full price (measured: 7.0 bpp estimated vs. 10.0 bpp
  actual, 8 of 10 tensors collapsed; after the floor, 7.36 vs. 7.36).
- **All parameters coded**, including biases (previously raw float32, a
  0.38 bpp floor at 256px) and the transmitted prior.
- **BD-rate refuses to extrapolate** — requires 4 rate points, ≥1 dB of
  shared quality range, and ≥2 measured points of each curve inside it.
  Replaying v1 data under this rule correctly withholds the earlier,
  fictitious −88% figure.
- Added: SSIM alongside PSNR, divergence detection with reseeded retry,
  JPEG2000 baseline, vector figures, LaTeX BD-rate table, run card.

## v4

- Introduced the learned-prior rate model shared between loss and coder.
- Lambda set to 1e-2, which (unknown at the time) crushed sigma to its floor
  and broke rate control — fixed in v5.
- Calibration interpolated a grid rather than rescaling it — fixed in v5.

## v1

- First working pipeline: hardcoded `sigma=0.05` in the loss (equivalent to
  plain L2 weight decay, `7.97 + 288.5*mean(w^2)`) while the coder fitted its
  own sigma independently, so the loss and the coder disagreed about what a
  bit cost.
- Kept as a parameter-count ablation, not a rate-distortion result: at a
  fixed quantiser with no working rate term, bitrate tracked parameter count
  almost exactly. That observation motivated the learned prior used from v4
  onward.
