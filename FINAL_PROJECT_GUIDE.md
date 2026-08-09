# Tetris AI Lab — Demonstration Guide

**Developed by Ghayda Ja'afreh**  
Powered by PyTorch and Pygame.

## Download the Windows release

1. Open the latest release page:
   [Tetris AI Lab v1.0.0](https://github.com/ghayda-njaafreh/tetris-ai-lab/releases/tag/v1.0.0)
2. Download `TetrisAILab-Windows-CPU.zip`.
3. Extract the complete ZIP file.
4. Open the extracted `TetrisAILab` folder.
5. Run `TetrisAILab.exe`.

Python and a dedicated NVIDIA GPU are not required.

## Recommended demonstration sequence

1. Open **About** to introduce the project and development credit.
2. Run **Greedy vs Final AI** with seed 42 to show the effect of lookahead and safety filtering.
3. Run **AI Agent** using the `7bag` generator.
4. Open **Human Play** to demonstrate real-time controls.
5. Open **Human vs Final AI** to demonstrate a fair shared-seed comparison.
6. Save a replay and open **Replay Viewer**.
7. Show the benchmark summary from the README or technical report.

## Main result to explain

The learned CNN achieved **68.43% Top-1** and **89.59% Top-3** accuracy on the held-out test set. Recovery training improved closed-loop play, while Top-5 one-piece lookahead and a limited safety filter produced the strongest final agent.

In the 7-bag benchmark, the final agent reached the **10,000-piece cap in all 30 tested games**.

## Accurate wording

Use:

> The final agent is a hybrid system that combines a CNN trained from expert video placements with one-piece lookahead and a limited safety filter.

Avoid claiming that the CNN alone achieved the final lookahead benchmark.

## Suggested presentation points

- The dataset was extracted from tournament footage and cleaned entirely through code.
- Splits were made by complete videos to reduce leakage.
- Recovery training exposed the model to damaged and high-risk boards.
- Mirror augmentation was accepted only when the simulator verified the transformed label exactly.
- The CNN frequently ranked good moves highly, while lookahead improved final ranking.
- Safety changed only a small fraction of decisions but prevented rare catastrophic placements.

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

## Public repository scope

The public repository contains documentation, benchmark results, application screenshots, and downloadable Windows releases. The proprietary source code, datasets, model-development pipeline, and internal development artifacts are maintained privately.


## Included media

Application screenshots are available under `screenshots/`, and the benchmark figure is available under `results/`.


A short demonstration video is included in the public repository:

`demo/tetris-ai-lab-demo-v1.0.0.mp4`
