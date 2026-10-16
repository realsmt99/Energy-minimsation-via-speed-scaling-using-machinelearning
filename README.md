# Energy Minimisation via Speed Scaling with Machine Learning

**Master 1 TER — Université d’Évry – Paris-Saclay (2025/2026)**  
Students: Taha Abdelkrim Douik, Mohamed Amine Menadi  
Supervisor: Vincent Chau

## Overview

Modern processors save energy with **Dynamic Voltage Scaling (DVS)** / **speed scaling**: they change frequency over time so that work finishes before deadlines while keeping total energy low. Energy is modelled as

\[
E = \int_0^T s(t)^\alpha \, dt \quad (\alpha > 1).
\]

In the classical model of Yao, Demers and Shenker, each job \(i\) is a triple \((r_i, d_i, w_i)\) (release time, deadline, work). Offline, the optimal schedule is given by **YDS**. Online, only jobs that have already arrived are known; classic algorithms (**AVR**, **OA**, **BKP**) guarantee a competitive ratio, but that ratio is strictly greater than 1.

**Learning-augmented scheduling** improves this picture: an ML oracle predicts future jobs, and the algorithm **LAS** (Bamas et al., 2020) blends that prediction with a fixed confidence parameter \(\varepsilon\). When the prediction is good, LAS is nearly optimal; when it is wrong, a small \(\varepsilon\) can make LAS *worse* than a pure online method such as OA.

## The problem with static confidence

LAS fixes \(\varepsilon\) once and for all before execution:

- Small \(\varepsilon\) → trust the prediction heavily (good consistency, weak robustness).
- Large \(\varepsilon\) → stay safer (better robustness, wasted prediction quality).

In practice you do not know in advance whether the oracle will stay accurate. On adversarial noise, classic LAS can reach a competitive ratio around **1.45**, worse than OA (~**1.12**) on the same instance.

## Our contribution: Dynamic Trust Value

We wrap LAS with a **Dynamic Trust Engine** that never changes LAS’s internals. It maintains a live **trust value** \(c \in [c_{\min}, 1]\) and filters each predicted volume before LAS sees it:

\[
\hat{w}_i^{\mathrm{filt}} = c \cdot \hat{w}_i + (1 - c) \cdot \bar{w},
\]

where \(\bar{w}\) is a short sliding-window average of recently observed real volumes (safe fallback).

### How \(c\) is updated

1. Compare each prediction to the true work once the job arrives (relative error \(e_i\)).
2. If \(e_i > \tau\) (default \(\tau = 0.4\)): **drop trust fast** (\(c \leftarrow c - \delta_\downarrow\), \(\delta_\downarrow = 0.5\)).
3. Otherwise: **raise trust slowly** (\(c \leftarrow c + \delta_\uparrow\), \(\delta_\uparrow = 0.2\)).
4. Trust is floored at \(c_{\min} = 0.1\) so a residual signal remains.

So when the oracle is reliable, \(c \approx 1\) and LAS receives the raw prediction (consistency preserved). When the oracle fails, \(c\) falls and LAS is fed mainly \(\bar{w}\), behaving closer to a robust online policy (degradation capped).

```
ML oracle  →  Dynamic Trust Engine (c)  →  LAS  →  schedule
                 ↑ feedback: |w − ŵ|
```

## Main results

| Setting | Classic LAS | With dynamic trust |
|--------|-------------|--------------------|
| Good predictions | ~1.00 | ~1.00 (transparent) |
| Adversarial / heavy noise | up to ~1.45 | capped ~1.10 |
| Brightkite real traffic (200 days) | mean ratio 1.1261 | **1.0796** (~**4.13%** energy gain) |

On Brightkite, day-to-day variance of the competitive ratio is also reduced by about **4×** (Wilcoxon paired test, \(p < 10^{-6}\)).

## Report

Full write-up (French): [`rapport (7) (1).pdf`](./rapport%20(7)%20(1).pdf)

Topics covered: speed-scaling model, YDS / AVR / OA / BKP / LAS, limits of static \(\varepsilon\), Dynamic Trust Engine design, synthetic and Brightkite experiments, hyperparameter sensitivity.

