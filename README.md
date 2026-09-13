# Privacy-Preserving Federated Learning for IoT Intrusion Detection

A comparative study of Centralized, FedAvg, and DP-FedAvg training on the RT-IoT2022 dataset, quantifying how much accuracy federated learning and differential privacy cost in exchange for not centralizing sensitive IoT traffic data.

Read Full Report here: [Report.pdf](./Report.pdf)

## Overview

This project asks two questions: can Federated Learning (FL) match centralized intrusion detection performance without pooling raw IoT network data, and what does adding formal Differential Privacy (DP) guarantees cost on top of that? A compact feed-forward neural network is trained under three regimes — centralized, standard FedAvg, and DP-FedAvg (Opacus-based gradient clipping + Gaussian noise) — and evaluated across client-count, non-IID severity, and privacy-noise sweeps, with multi-seed statistical testing throughout.

## Dataset

- **Source:** RT-IoT2022 (real IoT testbed network flow records)
- **Size:** 123,117 rows × 85 columns, no missing/infinite/duplicate values
- **Classes:** 12 original classes (normal device traffic + attack traffic) reduced to **11 classes** by merging two ultra-rare classes (37 and 28 rows) into a single `Rare_Attack` bucket
- Heavily imbalanced, spanning almost four orders of magnitude (94,659 vs. as few as 28 rows)
- Train/test: 98,493 / 24,624 rows, stratified split before scaling

## Methodology

1. **Model architecture** — small MLP (83 → 64 → 32 → 11, 7,819 parameters) used across all three settings so results are comparable.
2. **Centralized baseline** — 20 epochs, full training set.
3. **FedAvg** — hand-rolled training loop with IID and Dirichlet non-IID (α = 100, 1.0, 0.1) client partitioning.
4. **DP-FedAvg** — Opacus `PrivacyEngine` with per-sample gradient clipping and calibrated Gaussian noise (σ = 0.5, 1.0, 1.5), formal (ε, δ)-DP accounting via moments accountant.
5. **Experimental matrix** — 12 total runs covering headline comparison, client-count sweep (5/10/20), non-IID severity sweep, privacy-utility sweep, and 3-seed statistical robustness checks.

## Key Findings

- **FedAvg nearly matches centralized under IID data:** macro-F1 0.950 vs. 0.968 centralized ceiling — federation costs almost nothing in accuracy when client data is evenly distributed, at a wall-clock cost of ~133s vs. ~47s.
- **Non-IID data is the real risk, not federation itself:** IID, α=100, and α=1.0 all cluster around macro-F1 0.93–0.95, but severe label skew (α=0.1) collapses performance to macro-F1 = 0.397 — less than half of moderate-skew performance, and the model stays flat rather than slowly catching up.
- **More clients isn't automatically better:** at a fixed 30-round budget, 5 clients reached 0.957, 10 clients reached 0.950, but 20 clients dropped to 0.849 — communication cost doubles while accuracy drops, since each client sees proportionally less data per round.
- **DP incurs a steep, consistent utility cost:** all three noise levels score far below plain FedAvg (0.644–0.727 vs. 0.950), a 0.22–0.28 gap even at the weakest noise setting.
- **DP hurts "confusable" classes more than rare ones:** Wipro_bulb (0.98→0.72) and ARP_poisioning (0.98→0.78) suffered the largest drops, while the smallest class (`Rare_Attack`, 65 samples) was essentially unaffected — DP's cost tracks class separability from confusable neighbors more than raw class size.
- **DP noise dominates run-to-run variance:** across 3 seeds, DP-FedAvg's std (≈0.0002) was far tighter than FedAvg's (0.017) or centralized's (0.007), suggesting injected noise — not partition randomness — drives DP-FedAvg's behavior.
- **DP adds zero communication overhead** relative to plain FedAvg at the same client count — its cost is purely computational (gradient clipping/noise), not communicative.
- Statistical significance testing (Wilcoxon, n=3) hit its mathematical floor (p=0.25) and could not confirm significance — the study reports mean ± std as primary evidence rather than relying on underpowered p-values.

## Limitations

- The feature scaler was fit once on all data before splitting across clients — a simplification a real federated system wouldn't make.
- Reported privacy budget (ε) is per-round, not summed across the full training run.
- Federated learning was simulated on a single CPU; real network delays or dropped clients aren't captured.
- Only 3 random seeds were tested — not enough for the Wilcoxon test to reach conventional significance regardless of true effect size.
- A small, simple MLP was used for speed and clarity; a larger or different architecture might shift exact numbers.
- The client-count accuracy drop was only tested at a fixed round budget — it's untested whether more rounds would close the gap.

## Conclusion

Federated learning can match centralized performance almost exactly under IID data, removing the need to centralize sensitive IoT traffic in favorable conditions. But that advantage is fragile: realistic non-IID data can cut performance by more than half, and adding more clients doesn't automatically help. Formal differential privacy provides a real, measurable guarantee but comes at a steep and uneven accuracy cost. FedAvg is a strong, low-cost option under favorable conditions, but both data heterogeneity and privacy protection carry real costs that need to be planned for.

## References

- McMahan, H. B., Moore, E., Ramage, D., Hampson, S., & y Arcas, B. A. (2017). *Communication-Efficient Learning of Deep Networks from Decentralized Data.* AISTATS.
- Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., & Zhang, L. (2016). *Deep Learning with Differential Privacy.* ACM CCS.
- Beutel, D. J., et al. (2020). *Flower: A Friendly Federated Learning Research Framework.*
- Sharmila, S. B., & Nagapadma, R. (2023). *RT-IoT2022 (Real Time Internet of Things).* Kaggle.

## Author

Muhammad Adil (Independent research project)
