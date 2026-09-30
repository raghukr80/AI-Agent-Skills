# STEM Visualization Copilot Skill

A portable, renderer-neutral skill for turning Science, Technology,
Engineering and Mathematics questions into visual-first explanations.

## Package

- `SKILL.md` — main drop-in skill instructions
- `schemas/stem-visual-response.schema.json` — normalized response schema
- `renderers/RENDERER_CONTRACT.md` — renderer adapter interface
- `examples/` — reference outputs
- `tests/EVALUATION.md` — evaluation prompts and quality checks

## Installation

### Generic AI agent

Load `SKILL.md` as a system/developer skill or agent capability.

### Skill-based agent framework

Expose the directory as a skill package and make `SKILL.md` the entry point.

### Renderer architecture

Keep the skill independent of the rendering engine. Connect the host's
capabilities through renderer adapters.

Recommended flow:

User question
→ STEM skill
→ normalized visual response
→ renderer adapter
→ rendered visual
→ explanation

## Important implementation detail

The skill deliberately does not require a particular image model,
JavaScript framework, chart library, or diagram language.

That makes it portable across AI platforms.

## Suggested future extensions

- `schemas/interaction.schema.json`
- `schemas/assessment.schema.json`
- domain packs for physics, chemistry, biology, CS, engineering and maths
- misconception library
- accessibility adapter
- localization layer
- simulation adapter
- benchmark/evaluation dataset


## v1.1 visual policy

This release makes the renderer policy explicit: **ANIMATED / INTERACTIVE → STATIC GRAPHICAL → ASCII**.
For motion, time, sequences, cause/effect, flows, and parameter manipulation, an executable animation or interactive visual is preferred whenever the host supports it. ASCII is only a last-resort fallback.
