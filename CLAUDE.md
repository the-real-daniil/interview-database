# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A knowledge base of technical interview prep material, written in Russian, organized as plain Markdown files. There is no build, lint, or test tooling — the "product" is the content itself. Categories: `frontend/`, `backend/`, `system-design/`, `AI/`.

## Content format (required for every question)

Each question/concept is one self-contained `<details>` block, never a bare heading with prose. Follow `HOW_TO_CONTRIBUTE.md` exactly — it is the authoritative style guide, read it before adding content:

````markdown
<details>
<summary><b>Question?</b></summary>

Short 1–3 sentence direct answer first.

Key points as bullets, then optional code example, then trade-offs/comparisons if relevant, and common interview follow-up questions/pitfalls.

</details>
````

Rules distilled from `HOW_TO_CONTRIBUTE.md`:
- One `<details>` block = one question or concept. Don't merge unrelated questions into one block.
- Within a category file, group related questions under a `#`/`##` section heading, each followed by its own `<details>` blocks (see structure at the bottom of `HOW_TO_CONTRIBUTE.md`).
- Always tag code fences with a language; prefer TypeScript for frontend/AI examples.
- Use Mermaid for architecture/flow diagrams (```mermaid fences), not ASCII art — though existing files (e.g. `backend/nodejs.md`) do use ASCII diagrams for phase/pipeline illustrations; matching the surrounding file's existing convention beats a mechanical rule.
- New file names are kebab-case, e.g. `backend/database-indexes.md`, `AI/rag-architecture.md`.
- Don't add a question that duplicates one already answered elsewhere in the same file or a related file — grep the category first.
- A good answer explains *why*, not just *what*: mechanism, trade-offs, and typical interviewer follow-ups, not a copy-paste of docs.

## Repo layout

- `frontend/` — javascript.md, typescript.md, react.md, vue.md, angular.md, html_css.md, algorithms.md, other.md, plus `images/` referenced by relative path from these files.
- `backend/nodejs.md` — currently the only backend file.
- `system-design/frontend.md` — frontend-focused system design questions.
- `AI/` — meta-guidance on working with AI coding agents (best-practices.md, context.md, prompting.md), not interview Q&A.
- `HOW_TO_CONTRIBUTE.md` — the style/process guide; treat as source of truth over inference from existing files when they conflict.

## Workflow expectations

- Branch naming: `feature/some-topic-title`.
- PR titles are conventional-commit style and in English even though content is Russian, e.g. `feat: Add article about React rendering lifecycle`.
- Before adding a question, check the target file (and sibling files in the same category) for an existing entry on the same topic.
