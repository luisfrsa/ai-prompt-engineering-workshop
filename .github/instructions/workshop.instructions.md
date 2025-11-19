---
applyTo: "**"
---

# Workshop Instructions (Project-wide)

You are assisting in a **prompt engineering workshop**.

## 1. Scope & File Access

- You may **only** read from:
  - `transcript.md`
  - `prompt.md`
  - files under `.github/instructions/` (like this one)
- Do **not** open or use any other files in this repository unless the user explicitly asks for a specific path under `results/` or `ignored/`.
- Do **not** infer or use content from other project files.
- If the user asks you to use other files, reply:
  - "Per workshop rules I can only use `transcript.md`, `prompt.md`, `.github/instructions/*`, and any files you explicitly name under `results/`."
  - I should **only** create or edit files within `./resuls`.
    - You are **not allowed** to make changes to any other file or directory.

## 2. Output Location

- For each `transcript.md` analysis using `prompt.md` (summaries, structured notes, shrunk versions, style variants):
  - Write the output to a markdown file under the `results/` directory.
  - Prefix the file name with incremental numbers.
  - Suffix the file name choosing an unique name, max 6 words combined by `_` that represent the prompt from `prompt.md`.
    - For example, but not limited to:
      - `01_baseline_simple_summarize_meeting.md`
      - `01_persona_summary_from_pm_perspective.md`
      - `01_structured_what_heard_means_next.md`
- Never write outside the `results/` directory.

## 3. Memory

- Do not use memory from previous conversations.
- Each meeting transcript analysis and prompt should start fresh.
- Treat each task independently without referencing prior sessions or learned patterns from other analyses in this workshop.
