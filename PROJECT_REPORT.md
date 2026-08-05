# Tetris AI Lab — Technical Project Report

## 1. Objective

The project builds a Classic Tetris placement agent from tournament video data. It covers the complete workflow: video acquisition, board calibration, stable-state extraction, move inference, code-based cleaning, supervised learning, recovery-data generation, augmentation, closed-loop evaluation, and interactive deployment.

## 2. Data pipeline

For each pair of stable boards, the engine tests all tetrominoes, unique rotations, and legal columns. It simulates hard drop and line clearing, then compares the simulated result with the observed next board. High-confidence exact matches become labeled placement samples.

The final strict cleaning report is:

| Item | Count |
|---|---:|
| Raw samples | 58,636 |
| Accepted samples | 57,496 |
| Removed samples | 1,140 |
| Development-test samples removed | 143 |
| Non-exact or cell-error samples removed | 277 |
| Semantic duplicates removed | 720 |
| Missing board files | 0 |

The split is performed by complete source video rather than by individual frame, reducing leakage from temporally adjacent states.

## 3. Model

The policy input contains board occupancy, hole information, and a learned piece embedding. The model outputs 40 placement logits representing four rotations and ten columns. A legal-action mask removes impossible placements.

The baseline was trained with cross-entropy and evaluated on two held-out videos. Recovery versions add states visited by the model itself and labels produced by the repository's heuristic expert.

## 4. Recovery and augmentation

Two recovery rounds exposed the model to high, damaged, and hole-heavy boards that were rare in professional gameplay footage. Horizontal reflection augmentation was accepted only when the mirrored label could be exactly verified by the simulator.

The final v3 dataset contains:

| Item | Count |
|---|---:|
| Input training samples before mirror augmentation | 53,145 |
| Verified mirrored samples | 26,572 |
| Final training samples | 79,717 |
| Mapping failures | 0 |

## 5. Training safeguards

The final loss uses label smoothing only over legal actions. This avoids assigning probability mass to masked actions and prevents infinite loss. Additional safeguards include gradient clipping, early stopping, learning-rate decay, light class weighting, and source balancing.

## 6. Offline evaluation

On 9,180 independent test samples:

| Metric | Result |
|---|---:|
| Loss | 1.0539 |
| Top-1 | 68.43% |
| Top-3 | 89.59% |

Top-3 is relevant because multiple placements can be viable even when only the demonstrated human placement is labeled.

## 7. Closed-loop findings

The greedy learned policy substantially outperformed random play but still accumulated errors. Recovery training improved the greedy policy, while one-piece lookahead produced the largest improvement by reranking strong CNN candidates. A small safety filter prevented rare catastrophic decisions.

### Uniform generator, 100 games, 2,000-piece cap

| Policy | Mean pieces | Mean lines | Cap rate |
|---|---:|---:|---:|
| Greedy CNN | 143.42 | 40.96 | 0% |
| Lookahead Top-3 | 908.76 | 348.25 | 13% |
| Lookahead Top-5 | 1,716.85 | 679.39 | 71% |
| Lookahead + Safety | 1,846.60 | 732.68 | 83% |
| Heuristic | 1,256.90 | 488.95 | 31% |

### Stress benchmark, 30 games, 10,000-piece cap

| Generator | Final-agent mean pieces | Cap rate |
|---|---:|---:|
| Uniform | 5,985.67 | 26.67% |
| NES approximate | 7,964.50 | 63.33% |
| 7-bag | 10,000.00 | 100% |

The 7-bag result is cap-limited: it establishes survival through 10,000 pieces for all 30 tested seeds, not the uncapped lifetime.

## 8. Interpretation

The results suggest that the CNN frequently ranks useful moves highly but greedy argmax is vulnerable to compounding errors. Top-5 lookahead exploits the learned ranking while considering the immediate next-piece consequence. Safety changes only a small fraction of decisions but can remove fatal outliers.

The final system should therefore be described as a hybrid agent, not as a pure CNN policy.

## 9. Application

The Pygame application provides:

- AI mode;
- real-time human mode;
- human versus final AI;
- greedy versus final AI;
- replay saving and viewing;
- piece-generator, seed, speed, theme, sound, and fullscreen settings;
- live decision diagnostics;
- screenshots, help, about, high scores, and Windows packaging.

## 10. Limitations and future work

- Video labels are limited to final placements rather than frame-level controller inputs.
- The simulator does not claim exact emulation of every Classic Tetris timing rule.
- `nes_approx` is not cycle-accurate.
- Lookahead uses one next piece rather than a deeper search.
- The heuristic teacher can influence recovery behavior.
- Evaluation should be repeated across additional seed ranges and machine configurations.
- A signed Windows installer, automated CI, and release checksums would improve distribution.
