# AI Agent Skills

This repository contains a collection of skills for Hermes Agent, organized by category. Each skill provides specialized capabilities for specific tasks and workflows.

## Skill Categories

### Autonomous AI Agents
- **agents-sdk** — Build AI agents on Cloudflare Workers using the Agents SDK
- **claude-code** — Delegate coding to Claude Code CLI (features, PRs)
- **codex** — Delegate coding to OpenAI Codex CLI (features, PRs)
- **hermes-agent** — Configure, extend, or contribute to Hermes Agent
- **kanban-codex-lane** — Use when a Hermes Kanban worker wants to run Codex CLI
- **opencode** — Delegate coding to OpenCode CLI (features, PR review)

### Cloudflare
- **cloudflare** — Comprehensive Cloudflare platform skill covering Workers, Pages, D1, KV, R2, Queues, and more
- **cloudflare-email-service** — Send and receive transactional emails with Cloudflare Email
- **durable-objects** — Create and review Cloudflare Durable Objects
- **turnstile-spin** — Set up Cloudflare Turnstile end-to-end in a project
- **workers-best-practices** — Reviews and authors Cloudflare Workers code against production standards
- **wrangler** — Cloudflare Workers CLI for deploying, developing, and managing Workers

### Creative
- **architecture-diagram** — Dark-themed SVG architecture/cloud/infra diagrams as HTML
- **ascii-art** — ASCII art: pyfiglet, cowsay, boxes, image-to-ascii
- **ascii-video** — ASCII video: convert video/audio to colored ASCII MP4/GIF
- **baoyu-article-illustrator** — Article illustrations: type × style × palette consistency
- **baoyu-comic** — Knowledge comics (知识漫画): educational, biography, tutorial
- **baoyu-infographic** — Infographics: 21 layouts x 21 styles (信息图, 可视化)
- **claude-design** — Design one-off HTML artifacts (landing, deck, prototype)
- **comfyui** — Generate images, video, and audio with ComfyUI
- **design-md** — Author/validate/export Google's DESIGN.md token spec files
- **excalidraw** — Hand-drawn Excalidraw JSON diagrams (arch, flow, seq)
- **humanizer** — Humanize text: strip AI-isms and add real voice
- **ideation** — Generate project ideas via creative constraints
- **manim-video** — Manim CE animations: 3Blue1Brown math/algo videos
- **p5js** — p5.js sketches: gen art, shaders, interactive, 3D
- **pixel-art** — Pixel art w/ era palettes (NES, Game Boy, PICO-8)
- **popular-web-designs** — 54 real design systems (Stripe, Linear, Vercel) as HTML/CSS
- **pretext** — Build creative browser demos with @chenglou/pretext
- **sketch** — Throwaway HTML mockups: 2-3 design variants to compare
- **songwriting-and-ai-music** — Songwriting craft and Suno AI music prompts
- **touchdesigner-mcp** — Control a running TouchDesigner instance via twozero MCP

### Data Science
- **jupyter-live-kernel** — Iterative Python via live Jupyter kernel (hamelnb)

### DevOps
- **kanban-orchestrator** — Decomposition playbook + anti-temptation rules for an orchestrator
- **kanban-worker** — Pitfalls, examples, and edge cases for Hermes Kanban workers
- **webhook-subscriptions** — Webhook subscriptions: event-driven agent runs

### Dogfood
- **dogfood** — Exploratory QA of web apps: find bugs, evidence, reports

### Email
- **himalaya** — Himalaya CLI: IMAP/SMTP email from terminal

### Gaming
- **minecraft-modpack-server** — Host modded Minecraft servers (CurseForge, Modrinth)
- **pokemon-player** — Play Pokemon via headless emulator + RAM reads

### GitHub
- **codebase-inspection** — Inspect codebases w/ pygount: LOC, languages, ratios
- **github-auth** — GitHub auth setup: HTTPS tokens, SSH keys, gh CLI login
- **github-code-review** — Review PRs: diffs, inline comments via gh or REST
- **github-issues** — Create, triage, label, assign GitHub issues via gh or REST
- **github-pr-workflow** — GitHub PR lifecycle: branch, commit, open, CI, merge
- **github-repo-management** — Clone/create/fork repos; manage remotes, releases

### MCP (Model Context Protocol)
- **native-mcp** — MCP client: connect servers, register tools (stdio/HTTP)

### Media
- **gif-search** — Search/download GIFs from Tenor via curl + jq
- **heartmula** — HeartMuLa: Suno-like song generation from lyrics + tags
- **songsee** — Audio spectrograms/features (mel, chroma, MFCC) via CLI
- **spotify** — Spotify: play, search, queue, manage playlists and devices
- **youtube-content** — YouTube transcripts to summaries, threads, blogs

### MLOps
- **huggingface-hub** — HuggingFace hf CLI: search/download/upload models, datasets

#### MLOps / Evaluation
- **evaluating-llms-harness** — lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.)
- **weights-and-biases** — W&B: log ML experiments, sweeps, model registry, dashboards

#### MLOps / Inference
- **llama-cpp** — llama.cpp local GGUF inference + HF Hub model discovery
- **obliteratus** — OBLITERATUS: abliterate LLM refusals (diff-in-means)
- **serving-llms-vllm** — vLLM: high-throughput LLM serving, OpenAI API, quantization

#### MLOps / Models
- **audiocraft-audio-generation** — AudioCraft: MusicGen text-to-music, AudioGen text-to-sound
- **segment-anything-model** — SAM: zero-shot image segmentation via points, boxes, masks

#### MLOps / Research
- **dspy** — DSPy: declarative LM programs, auto-optimize prompts, RAG

### Note Taking
- **obsidian** — Read, search, create, and edit notes in the Obsidian vault

### Productivity
- **airtable** — Airtable REST API via curl. Records CRUD, filters, upserts
- **google-workspace** — Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python
- **linear** — Linear: manage issues, projects, teams via GraphQL + curl
- **maps** — Geocode, POIs, routes, timezones via OpenStreetMap/OSRM
- **nano-pdf** — Edit PDF text/typos/titles via nano-pdf CLI (NL prompts)
- **notion** — Notion API + ntn CLI: pages, databases, markdown, Workers
- **ocr-and-documents** — Extract text from PDFs/scans (pymupdf, marker-pdf)
- **petdex** — Install and select animated petdex mascots for Hermes
- **powerpoint** — Create, read, edit .pptx decks, slides, notes, templates
- **teams-meeting-pipeline** — Operate the Teams meeting summary pipeline via Hermes CLI

### Red Teaming
- **godmode** — Jailbreak LLMs: Parseltongue, GODMODE, ULTRAPLINIAN

### Research
- **arxiv** — Search arXiv papers by keyword, author, category, or ID
- **blogwatcher** — Monitor blogs and RSS/Atom feeds via blogwatcher-cli tool
- **llm-wiki** — Karpathy's LLM Wiki: build/query interlinked markdown KB
- **polymarket** — Query Polymarket: markets, prices, orderbooks, history

### Sandbox SDK
- **sandbox-sdk** — Build sandboxed applications for secure code execution

### Smart Home
- **openhue** — Control Philips Hue lights, scenes, rooms via OpenHue CLI

### Social Media
- **xurl** — X/Twitter via xurl CLI: post, search, DM, media, v2 API

### Software Development
- **debugging-hermes-tui-commands** — Debug Hermes TUI slash commands: Python, gateway, Ink UI
- **diagram-editor-trace-flow** — Implement request trace/flow visualization in ReactFlow-based editor
- **hermes-agent-skill-authoring** — Author in-repo SKILL.md: frontmatter, validator, structure
- **hermes-s6-container-supervision** — Modify, debug, or extend the s6-overlay supervision tree
- **node-inspect-debugger** — Debug Node.js via --inspect + Chrome DevTools Protocol CLI
- **plan** — Plan mode: write an actionable markdown plan to .hermes/plans/
- **python-debugpy** — Debug Python: pdb REPL + debugpy remote (DAP)
- **requesting-code-review** — Pre-commit review: security scan, quality gates, auto-fix
- **simplify-code** — Parallel 3-agent cleanup of recent code changes
- **spike** — Throwaway experiments to validate an idea before build
- **subagent-driven-development** — Execute plans via delegate_task subagents (2-stage review)
- **system-design-simulator** — Build browser-based system design simulators with React Flow
- **systematic-debugging** — 4-phase root cause debugging: understand bugs before fixing
- **test-driven-development** — TDD: enforce RED-GREEN-REFACTOR, tests before code
- **vite-tailwind-react** — Scaffold, verify, and debug Vite + React + Tailwind CSS projects
- **writing-plans** — Write implementation plans: bite-sized tasks, paths, code

### Yuanbao
- **yuanbao** — Yuanbao (元宝) groups: @mention users, query info/members

## Custom Skills (Local)

Located in `/home/raghu/Skills/`:
- **stem-visualization-skill** — STEM visualization capabilities
- **universal-visual-learning-skill** — Universal visual learning framework

## Usage

Skills are loaded automatically based on the task at hand. To manually load a skill:

```bash
# In Hermes Agent
skill_view(name="category/skill-name")
```

Or use the skill system to discover available skills:

```bash
skills_list()
skills_list(category="cloudflare")
```

## Contributing

To add a new skill:
1. Create a new directory under the appropriate category
2. Add a `SKILL.md` file with frontmatter and documentation
3. Include any reference files, templates, or scripts in subdirectories

## Repository

- **GitHub**: https://github.com/raghukr80/AI-Agent-Skills
- **Hermes Profile**: default

---

*Generated on September 30, 2026*