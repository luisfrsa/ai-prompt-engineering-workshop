# How to Prompt AI for Consistent Results — Workshop

This repo is a minimal sandbox for the **How to Prompt AI for Consistent Results** workshop.

## 1. Key Files
- `transcript.md` — the messy cross‑functional planning meeting you’ll summarize.
- `prompt.md` — the main prompt template you’ll refine and reuse during the exercises.
- `prompt-examples.md` — example prompts for each step of the workshop.
- `.github/instructions/workshop.instructions.md` (in the full workshop repo) — facilitator and Copilot instructions that explain how runs should be shaped and saved.

## 2. How to Run the Exercises
- Open this repo in VS Code with Copilot Chat (or the CLI).
- Use `prompt.md` together with `transcript.md` to ask the model for meeting summaries.
- Follow the slide / workshop flow: start with a baseline run, then iterate by adding role, structure, format, and length constraints.

## 3. Examples & Outputs
- `prompt-examples.md` — ready-made prompt variants for each step (baseline, add role, add structure & intent, control format & length, build your own style).
- `results/` — drop each model run here (e.g., one file per experiment) so you can compare shape, length, and consistency across iterations and models.
