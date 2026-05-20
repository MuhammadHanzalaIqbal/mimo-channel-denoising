# mimo-channel-denoising

**MIMO channel denoising via adaptive delay-window + soft low-rank (SVT) shrinkage. Strictly improves on the hard-window baseline at every uplink-SNR / transmission-rank combination tested.**

The final ("competition") assignment of the Skoltech MIMO course: given a noisy LS-estimated MIMO channel, design and justify a denoiser that improves downlink spectral efficiency over the course-provided hard-window baseline. The proposed method exploits two structural properties of measured MIMO channels — **delay-domain sparsity** and **per-resource low-rank structure** — and combines them in a single pipeline.

## The proposed method

```
H_noisy  (T, K, M, N)
   │
   ▼  IDFT along subcarriers
delay-domain  Z(t, τ, m, n)
   │
   ▼  adaptive circular window on averaged PDP
   │     • length & start estimated from the noise-floor-subtracted PDP
   │     • SNR-dependent energy-retention target (frac, max_len)
windowed  Z_w(t, τ, m, n)
   │
   ▼  per (t, τ) slot: SVD + soft-threshold
   │     • S_i ← max(S_i − τ, 0)   with  τ = tau_scale · median(S)
   │     • SVT-style nuclear-norm shrinkage
denoised  Z'(t, τ, m, n)
   │
   ▼  FFT back to subcarrier domain
Ĥ_denoised
```

Two ideas, composed:

1. **Adaptive delay-window** — most channel energy lives in a small set of delay bins. The hard-window baseline keeps a *fixed* slice (bins `0:13` + `-6:`); the proposed method estimates the smallest *circular* window that retains a target fraction of post-noise-floor PDP energy. Length and position are adapted per scenario; the window is wider at low UL SNR (preserve signal) and tighter at high SNR (more aggressive cut).
2. **Soft low-rank shrinkage (SVT)** — per `(t, τ)` slot the channel matrix `H(t, τ) ∈ ℂ^(M × N)` has a fast-decaying singular spectrum. Shrinking small singular values via `max(σ − τ, 0)` suppresses the noise-dominated subspace while preserving the signal-dominated one. The threshold scales with the per-slot `median(S)` so it adapts to local noise level.

## Results

Median spectral efficiency [bits/s/Hz] on the true channel, downlink SNR = 5 dB, evaluated with each method's SVD precoder built from its own estimate:

| UL SNR | Rank | Ideal CSI | Noisy CSI | Hard-window baseline | **Proposed** | **Δ over baseline** |
|---:|---:|---:|---:|---:|---:|---:|
| −20 dB | 1 | 0.774 | 0.032 | 0.314 | **0.467** | **+0.153 (+49%)** |
| −20 dB | 2 | 0.691 | 0.033 | 0.247 | **0.364** | **+0.117 (+47%)** |
| −20 dB | 3 | 0.532 | 0.034 | 0.191 | **0.270** | **+0.079** |
| −20 dB | 4 | 0.424 | 0.033 | 0.157 | **0.215** | **+0.058** |
| −15 dB | 1 | 0.774 | 0.051 | 0.546 | **0.600** | **+0.055** |
| −15 dB | 2 | 0.691 | 0.051 | 0.442 | **0.492** | **+0.049** |
| −15 dB | 3 | 0.532 | 0.049 | 0.325 | **0.357** | **+0.032** |
| −15 dB | 4 | 0.424 | 0.047 | 0.256 | **0.279** | **+0.023** |
| −10 dB | 1 | 0.774 | 0.122 | 0.684 | **0.721** | **+0.037** |
| −10 dB | 2 | 0.691 | 0.107 | 0.585 | **0.632** | **+0.048** |
| −10 dB | 3 | 0.532 | 0.093 | 0.433 | **0.474** | **+0.040** |
| −10 dB | 4 | 0.424 | 0.082 | 0.340 | **0.374** | **+0.035** |

The proposed method **strictly improves on the hard-window baseline at every (UL SNR, rank) combination**. The relative gain is largest at the hardest SNR (≈ +49% at UL = −20 dB, rank 1), where naive estimation is most noisy and the denoiser has the most to fix.

## Structural analysis

The notebook also reports three diagnostic structural visualisations of the channel used to justify the design choices:

- Average **power delay profile (PDP)** — confirms compact energy support; motivates the delay window.
- Average **beam-domain power spectrum** — confirms angular sparsity.
- Average **singular-value spectrum** of `H(t, τ)` — confirms fast decay; motivates SVT.

## What's in here

| File | Purpose |
|---|---|
| `notebooks/channel_denoising.ipynb` | Complete notebook: baseline + proposed method + evaluation + figures + structural diagnostics |
| `report.pdf` | Written report (PDF — distribution-controlled) |
| `LICENSE` | MIT |

## Data

Course-provided MIMO channel `.mat` files (`link_chan_*.mat`). **Not included in this repository** (course-restricted). Place them in the directory referenced by `FOLDER_PATH` at the top of the notebook.

## Running it

```bash
pip install numpy scipy matplotlib tqdm pandas
```

Pure NumPy/SciPy — no GPU required. Each (UL SNR, rank) sweep takes a few seconds.

## Context

Final ("competition") assignment of the **MIMO Wireless Communications** course at the **Skolkovo Institute of Science and Technology (Skoltech)**, M.Sc. in IoT and Wireless Technologies. The algorithm design, justification, and analysis are my own work; the noisy-channel scenario and the hard-window baseline are course-provided.

## License

MIT — see [`LICENSE`](LICENSE).

---

*Author: Muhammad Hanzala Iqbal.*

## Related projects

- [`mimo-spatial-filtering`](https://github.com/MuhammadHanzalaIqbal/mimo-spatial-filtering) — perfect-CSI baseline; SVD precoding and beam-domain analysis.
- [`mimo-channel-estimation`](https://github.com/MuhammadHanzalaIqbal/mimo-channel-estimation) — the SRS-based LS estimate this project denoises.
- [`nanoGPT-rf-rope-ablation`](https://github.com/MuhammadHanzalaIqbal/nanoGPT-rf-rope-ablation) — small-transformer architectural ablation (RoPE vs learned PE).
