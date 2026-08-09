# Changelog

All notable public release changes are documented in this file.

## v1.0.0 — 2026-08-05

### Added

- First public Windows release of Tetris AI Lab.
- Portable CPU-only Windows build that does not require Python or a dedicated NVIDIA GPU.
- CNN placement policy trained from Classic Tetris tournament footage.
- Strict automated data cleaning and video-level train/validation/test splitting.
- Recovery training using simulator-generated difficult states.
- Simulator-verified horizontal reflection augmentation.
- Top-5 one-piece lookahead.
- Limited safety filter for preventing catastrophic placements.
- AI Agent, Human Play, Human vs Final AI, and Greedy vs Final AI modes.
- Replay viewer, screenshots, themes, audio, settings, high scores, help, and about screens.
- Benchmarking across Uniform, NES-approximate, and 7-bag piece generators.

### Verified results

- Test Top-1 accuracy: 68.43%.
- Test Top-3 accuracy: 89.59%.
- Uniform RNG: 5,985.67 average pieces in the 10,000-piece stress benchmark.
- NES Approx RNG: 7,964.50 average pieces.
- 7-bag RNG: reached the 10,000-piece cap in 100% of 30 benchmark games.

### Distribution

- Published `TetrisAILab-Windows-CPU.zip` as the public Windows release asset.
- Proprietary source code, datasets, and internal development artifacts remain private.

### Updated

- Added a public 16:9 demonstration video to the repository.
- Replaced the performance-evolution figure with the version generated directly from project summary files.
