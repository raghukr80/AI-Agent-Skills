# Universal Visual Learning Skill

A portable skill specification for turning any topic into:

1. An interactive visual lesson
2. A polished educational infographic image
3. Matching terminology and explanations
4. Creator credits

## Supported AI tools

The core package is platform-neutral and can be adapted to:
- Claude
- Gemini
- ChatGPT / OpenAI-compatible tools
- DeepSeek
- Qwen Studio
- Other assistants supporting custom instructions, skills, system prompts,
  projects, or agent instructions

The exact installation mechanism varies by product.

## Package

- `skills/SKILL.md` — main portable skill
- `templates/interactive-lesson.html` — standalone HTML starter
- `templates/image-prompt.md` — image-generation prompt template
- `templates/topic-spec.json` — optional topic schema
- `adapters/claude.md` — Claude installation notes
- `adapters/gemini.md` — Gemini installation notes
- `adapters/deepseek.md` — DeepSeek installation notes
- `adapters/qwen-studio.md` — Qwen Studio installation notes
- `adapters/openai-compatible.md` — generic OpenAI-compatible installation notes
- `examples/token-bucket.md` — example transformation

## Quick use

Install/import the skill text as the tool's custom instruction/skill, then use:

`/visualize learn token bucket algorithm`

or:

`Teach me RAG visually`

The expected result is:
- interactive lesson where supported
- infographic image
- creator credits in both

## Portability model

The skill is deliberately based on instructions rather than vendor-specific
APIs. The host decides how to render:
- HTML interactive content
- SVG
- image generation
- Markdown fallback

This makes the same teaching logic portable while allowing each AI product to
use its own native rendering features.
