# Tetris AI Lab — Demonstration Guide

**Developed by Ghayda Ja'afreh**  
Powered by PyTorch and Pygame.

## Fast demonstration

```powershell
python play_tetris.py
```

Recommended sequence:

1. Open **About** to introduce the project.
2. Run **Greedy vs Final AI** with seed 42.
3. Run **AI Agent** using `7bag`.
4. Open **Human Play** to demonstrate real-time controls.
5. Open **Replay Viewer**.
6. Show benchmark results from the README or project report.

## Key result to explain

The learned CNN alone achieved 68.43% Top-1 and 89.59% Top-3 accuracy on the held-out test set. Recovery training improved closed-loop play, and the final Top-5 lookahead plus safety agent reached the 10,000-piece benchmark cap in all 30 tested 7-bag games.

## Accurate wording

Use:

> The final agent is a hybrid system that combines a CNN trained from expert video placements with one-piece lookahead and a limited safety filter.

Avoid claiming that the CNN alone achieved the final lookahead benchmark.

## Windows demonstration

```powershell
powershell -ExecutionPolicy Bypass -File .\build_windows.ps1
.\dist\TetrisAILab\TetrisAILab.exe
```
