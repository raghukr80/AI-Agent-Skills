# Renderer Adapter Contract

A renderer adapter consumes the normalized STEM visual response and produces
a host-specific visualization.

## Input

The adapter receives:

- `visual_plan`
- `visual.spec`
- `concept_model`
- `equations`
- optional interaction metadata
- `selected_renderer`
- optional `animation_spec`

## Output

Return:

- rendered artifact
- accessibility/alt text
- optional interaction controls
- optional source/code when the host supports it

## Rules

1. Never change scientific meaning during rendering.
2. Preserve labels and units.
3. Preserve numerical values.
4. Clearly distinguish illustrative values.
5. Make the first frame understandable without interaction.
6. If the concept benefits from motion/time/sequence and the host supports it, render animation or interaction rather than a static text representation.
7. If animation cannot execute, use a static graphical renderer before ASCII.
8. ASCII is a last-resort fallback only when no graphical renderer is executable.
9. Return the actual `selected_renderer` used.
10. Provide a non-interactive fallback where practical.

## Suggested adapters

- `image`
- `svg`
- `mermaid`
- `html`
- `python-matplotlib`
- `manim`
- `threejs`
- `native-chart`
- `canvas-html`
