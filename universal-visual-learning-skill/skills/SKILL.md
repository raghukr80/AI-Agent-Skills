---
name: universal-visual-learning
version: 1.0.0
description: >
  Turn any learning topic into a visual-first interactive lesson and a
  presentation-ready infographic image, with creator credits. Portable across
  Claude, Gemini, ChatGPT/OpenAI-compatible tools, DeepSeek, Qwen Studio,
  and other AI tools that support custom instructions/skills/prompts.
---

# Universal Visual Learning Skill

## Purpose

When the user asks to learn, explain, visualize, teach, brush up, revise, or
create a visual lesson for a topic, transform the topic into TWO coordinated
artifacts:

1. Interactive Visual Lesson
2. Infographic / Poster Image

The two artifacts must teach the same content and use the same terminology.

## Default creator credits

Place these credits visibly but unobtrusively on generated images/posters:

- LinkedIn: https://linkedin.com/in/raghupathyk
- GitHub: https://github.com/raghukr80

Preferred placement: bottom-left and bottom-right corners. If the image has
a dark footer, use a high-contrast compact footer. Never let credits cover
important content.

For interactive lessons, put the same credits in a small footer.

## Core behavior

### Step 1 — Understand the topic

Identify:
- topic
- likely audience
- learning objective
- prerequisites
- whether the topic is conceptual, procedural, mathematical, technical,
  architectural, scientific, or interview-oriented

If the user does not specify an audience, default to practical learner /
interview-ready explanation.

Do not ask unnecessary clarification questions. Make reasonable assumptions
and state them briefly when needed.

### Step 2 — Build a visual mental model

Prefer:
- flow diagrams
- pipelines
- timelines
- layered architecture
- component diagrams
- before/after views
- cause/effect diagrams
- state transitions
- comparison tables
- annotated formulas
- worked examples
- simulations
- feedback loops

Avoid ASCII diagrams unless no visual rendering capability is available.

### Step 3 — Create the interactive visual lesson

The lesson should contain, when applicable:

1. What it is
2. Why it matters
3. Core mental model
4. Visual flow
5. Step-by-step operation
6. Parameters / components
7. Worked example or simulation
8. Common mistakes / failure modes
9. Real-world use cases
10. Comparison with related concepts
11. Interview / exam questions
12. Key takeaways

Interaction should be meaningful, not decorative. Examples:
- sliders for parameters
- play/pause simulation
- step-through flow
- clickable architecture components
- tabs for variants
- toggle for normal vs failure path
- quiz/checkpoint
- reset button

If the host AI tool cannot execute HTML/JavaScript, provide a static
interactive-style lesson using expandable sections or clearly separated
visual frames.

### Step 4 — Create the infographic image

Generate a polished landscape educational poster inspired by professional
technical-learning infographics.

Preferred structure:

HEADER
- Strong topic title
- one-line subtitle
- small visual metaphor/icon

BODY
- 6–10 numbered panels
- color-coded sections
- clear visual flows
- concise text
- diagrams/icons
- examples
- comparison where useful

FOOTER
- key takeaways
- remember box
- creator credits

Use a consistent visual language:
- dark/navy header
- light background
- rounded panels
- restrained pastel section colors
- strong typography hierarchy
- arrows and icons
- high information density without tiny text

Do not imitate a specific copyrighted infographic. Use the user's supplied
image only as a layout/style reference and create an original design.

### Step 5 — Accuracy rules

For technical topics:
- distinguish facts from examples
- do not invent APIs, benchmarks, standards, or numeric claims
- label illustrative numbers as illustrative
- preserve mathematical correctness
- show units
- explain assumptions

For architecture topics:
- show data flow
- show control flow when relevant
- identify stateful components
- show failure paths
- show security boundaries when relevant
- include scaling considerations

For algorithms:
- show state
- show inputs/outputs
- show one worked example
- show complexity when relevant
- show edge cases

For AI/ML:
- show data → model → evaluation → deployment where relevant
- distinguish training, inference, retrieval, tools, and orchestration
- include latency, cost, safety, and evaluation when production context matters

## Output strategy by host capability

Use this priority order:

A. If interactive HTML/app rendering is available:
   - Render the interactive lesson directly.
   - Generate the image using the host image-generation capability.

B. If image generation exists but interactive HTML does not:
   - Generate the infographic image.
   - Provide a structured interactive lesson in Markdown with expandable
     sections if supported.

C. If neither exists:
   - Provide a self-contained SVG/HTML lesson when allowed.
   - Otherwise provide a Markdown visual lesson and an image-generation prompt.

Never claim that an image or interactive app was generated if the host cannot
actually generate/render it.

## Image prompt blueprint

Use this internal prompt pattern:

"Create an original educational technical infographic about [TOPIC].
Landscape 16:9. Professional flat-vector learning poster. Dark navy header.
Clear numbered sections. Show [CORE FLOW]. Include [EXAMPLE], [COMPARISON],
and [KEY TAKEAWAYS]. Use readable typography, clean arrows, simple technical
icons, rounded panels, restrained pastel colors, high contrast, and generous
spacing. Make every label legible. Add a small footer with:
LinkedIn: https://linkedin.com/in/raghupathyk
GitHub: https://github.com/raghukr80
Do not obscure content with credits."

If the image model struggles with exact text, generate the visual without
dense text and produce a companion SVG/HTML version containing exact labels.

## Interactive lesson design

Use a responsive canvas with:
- topic header
- visual overview
- interactive diagram
- controls
- explanation panel
- interview/checkpoint panel
- takeaways
- credits footer

Minimum interaction:
- one parameter or step control
- one visual state change
- reset

Accessibility:
- keyboard reachable controls
- visible labels
- no hover-only information
- reduced-motion support
- readable contrast
- mobile-friendly layout

## Topic adaptation examples

For "Token Bucket":
- animated bucket
- token refill
- request consumption
- allow/reject state
- burst slider
- Token Bucket vs Leaky Bucket

For "RAG":
- ingestion → chunking → embedding → vector DB → retrieval → reranking →
  prompt → LLM
- toggle Basic / Hybrid / Agentic RAG
- show retrieval quality and hallucination controls

For "TCP 3-way handshake":
- SYN → SYN-ACK → ACK
- packet animation
- connection states
- failure/retry branch

For "Photosynthesis":
- sunlight + CO2 + water → glucose + oxygen
- chloroplast visual
- light-dependent / Calvin cycle tabs

For "Quadratic equation":
- formula
- coefficient controls
- graph
- roots
- discriminant states

## User command recognition

Treat these as triggers:
- /visualize [topic]
- /learn [topic]
- /visualize learn [topic]
- "teach me [topic] visually"
- "create an infographic for [topic]"
- "make an interactive lesson for [topic]"

If the user explicitly asks only for an image, produce only the image.
If they explicitly ask only for an interactive lesson, produce only that.
If they use /visualize or "visualize learn", produce both by default.

## Credit rules

Always preserve the exact URLs:
https://linkedin.com/in/raghupathyk
https://github.com/raghukr80

Do not replace them with guessed usernames or shortened URLs.

## Final quality checklist

Before returning:
- [ ] Topic is correct
- [ ] Visual flow matches explanation
- [ ] Example is internally consistent
- [ ] No invented facts
- [ ] Image is readable at normal zoom
- [ ] Credits are visible
- [ ] Interactive controls actually work
- [ ] Mobile layout does not break
- [ ] Reduced-motion behavior exists when animation is used
