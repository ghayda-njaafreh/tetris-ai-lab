# Tetris AI Lab

A reproducible Classic Tetris project that learns placement decisions from tournament footage, improves recovery through simulator-generated data, and deploys the resulting agent in an interactive Pygame application.

**Developed by Ghayda Ja'afreh**  
Powered by PyTorch and Pygame.

## Project highlights

- Video-to-dataset pipeline for 20×10 Classic Tetris boards.
- Strict code-based data cleaning and leakage-resistant video-level splits.
- CNN imitation policy with legal-action masking.
- Recovery/DAgger-style data generation from model-visited states.
- Simulator-verified horizontal reflection augmentation.
- Top-5 one-piece lookahead and a limited safety filter.
- Benchmarks across Uniform, NES-approximate, and 7-bag piece generators.
- Interactive AI, human, comparison, versus, replay, settings, audio, and screenshot modes.
- Windows packaging with PyInstaller.

## Final agent

```text
CNN policy_recovery_v3
+ Top-5 one-piece lookahead
+ Safety filter
```

The learned model predicts one of 40 final placements (`4 rotations × 10 columns`). Illegal placements are masked before selection. The final agent reranks the model's strongest candidates using a one-piece lookahead and applies a small safety override only in risky states.

## Verified results

### Offline test set

The independent test set contains 9,180 samples from videos excluded from training.

| Metric | Result |
|---|---:|
| Test loss | 1.0539 |
| Top-1 accuracy | 68.43% |
| Top-3 accuracy | 89.59% |

### Closed-loop gameplay

| Benchmark | Greedy CNN | Final agent |
|---|---:|---:|
| Uniform, 100 games, 2,000-piece cap | 143.42 mean pieces | 1,846.60 mean pieces |
| Uniform, 30 games, 10,000-piece cap | — | 5,985.67 mean pieces |
| NES-approx, 30 games, 10,000-piece cap | — | 7,964.50 mean pieces |
| 7-bag, 30 games, 10,000-piece cap | — | 10,000 mean pieces; 100% reached cap |

These are capped simulator benchmarks, not claims about official competitive Classic Tetris performance.

## Dataset preparation

The complete cleaning step is implemented in `prepare_training_data.py`.

```text
Raw samples:                 58,636
Accepted clean samples:      57,496
Removed samples:              1,140
Semantic duplicates removed:    720
Missing board files:              0
Train / validation / test: 39,367 / 8,949 / 9,180
```

Cleaning includes:

- development-test source exclusion;
- exact-match and zero-cell-error requirements;
- ambiguity rejection;
- board-file validation;
- semantic duplicate removal;
- deterministic video-level splitting with seed 42.

The v3 training set contains 79,717 samples after recovery data and simulator-verified mirror augmentation. All 26,572 requested mirrored samples were created with zero mapping failures.

## Installation

Python 3.10 or newer is required. The project was tested on Windows with Python 3.13.

```powershell
python -m venv myEnv
.\myEnv\Scripts\Activate.ps1
python -m pip install -U pip
pip install -e ".[dev,ml,gui]"
pytest
```

## Run the application

Open the main menu:

```powershell
python play_tetris.py
```

Direct modes:

```powershell
python play_tetris.py --mode ai --piece-source 7bag --seed 42
python play_tetris.py --mode human --piece-source 7bag --seed 42
python play_tetris.py --mode versus --piece-source 7bag --seed 42
python play_tetris.py --mode compare --piece-source 7bag --seed 42 --speed 2
python play_tetris.py --mode replay
```

Available generators:

```text
uniform
nes_approx
7bag
```

## Human controls

| Key | Action |
|---|---|
| Left / Right | Move piece |
| Up or X | Rotate clockwise |
| Z | Rotate counter-clockwise |
| Down | Soft drop |
| Space | Hard drop |
| P | Pause where supported |
| R | Restart with the same seed |
| S | Save replay |
| F12 | Save screenshot |
| Esc | Return or quit |

## Reproduce the model pipeline

### 1. Prepare and clean data

```powershell
python prepare_training_data.py
pytest
```

### 2. Train the baseline

```powershell
python train_policy.py --epochs 15 --batch-size 256 --output artifacts\policy_baseline
```

### 3. Evaluate offline

```powershell
python evaluate_policy.py --data data\prepared\test.jsonl --checkpoint artifacts\policy_baseline\best.pt
```

### 4. Generate recovery data and train v3

The repository includes the recovery, augmentation, and training scripts used to create `policy_recovery_v3`:

```text
generate_recovery_dataset.py
build_augmented_training_data.py
augment_training_data.py
train_policy.py
```

Final v3 training configuration:

```text
Train samples:        79,717
Validation samples:    8,949
Maximum epochs:           30
Batch size:              256
Learning rate:        0.0005
Class-weight power:      0.15
Source balancing:        0.10
Label smoothing:         0.02
Gradient clipping:        1.0
Early-stopping patience:    6
```

### 5. Gameplay evaluation

```powershell
python evaluate_gameplay.py --checkpoint artifacts\policy_recovery_v3\best.pt --episodes 100 --max-pieces 2000 --seed 42

python evaluate_gameplay_lookahead.py `
  --checkpoint artifacts\policy_recovery_v3\best.pt `
  --episodes 30 `
  --max-pieces 10000 `
  --seed 42 `
  --policies lookahead_top5 lookahead_safety heuristic
```

### 6. Piece-generator benchmark

```powershell
python compare_piece_generators.py `
  --checkpoint artifacts\policy_recovery_v3\best.pt `
  --output artifacts\policy_recovery_v3\rng_benchmark `
  --episodes 30 `
  --max-pieces 10000 `
  --seed 42 `
  --generators uniform nes_approx 7bag `
  --policies lookahead_top5 lookahead_safety heuristic
```

## Windows build

Create a portable Windows folder:

```powershell
powershell -ExecutionPolicy Bypass -File .\build_windows.ps1
```

The application is generated at:

```text
dist\TetrisAILab\TetrisAILab.exe
```

Keep the entire `dist\TetrisAILab` folder together. For transfer:

```powershell
Compress-Archive -Path .\dist\TetrisAILab -DestinationPath .\dist\TetrisAILab-Windows.zip -Force
```

## Repository structure

```text
src/tetris_imitation/    video extraction and board inference
src/tetris_ml/           datasets, model, lookahead, and policy utilities
src/tetris_game/         Pygame application and controllers
configs/                 video and batch configurations
data/                    generated datasets and manifests
artifacts/               checkpoints, analyses, benchmarks, replays, screenshots
play_tetris.py           interactive application entry point
train_policy.py          model training
evaluate_policy.py       offline evaluation
evaluate_gameplay*.py    closed-loop evaluation
```

## Responsible use and limitations

- Tournament footage should only be processed when its use is permitted.
- Source clips and broadcast-derived images should not be redistributed without appropriate permission.
- `nes_approx` is an explicitly approximate generator and not a cycle-accurate NES RNG implementation.
- Gameplay results depend on this repository's simulator, action representation, generator, caps, seed range, and safety settings.
- The final agent is hybrid: the CNN is learned, while lookahead and safety use simulator-based board evaluation.

## Documentation

- `PROJECT_REPORT.md` — methodology, experiments, and findings.
- `RELEASE_CHECKLIST.md` — final Windows and GitHub release checks.
- `FINAL_PROJECT_GUIDE.md` — quick demonstration guide.
