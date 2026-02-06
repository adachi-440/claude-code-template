# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- ============================================================
  PROJECT-SPECIFIC SECTION
  After cloning, fill in the sections below for your project.
============================================================ -->

<!-- ============================================================
  /init INSTRUCTIONS
  When running /init, follow these rules:
  1. Only modify the PROJECT-SPECIFIC SECTION (Project Overview,
     Build & Development, Architecture). Do NOT modify or remove
     the TEMPLATE FRAMEWORK SECTION below (Skills, Agents,
     Conventions).
  2. After updating Build & Development, also update
     `.claude/settings.local.json`:
     - Add build/test/lint/format commands to "permissions.allow"
       (e.g. "Bash(npm run build:*)", "Bash(npm test:*)")
     - Add formatter/linter hooks to "hooks.PostToolUse" so they
       run automatically after Edit/Write operations.
  3. Keep existing entries in settings.local.json (git, gh, codex,
     WebSearch, etc.) — only append project-specific ones.
============================================================ -->

## Project Overview

<!-- Describe your project in 1-3 sentences. -->

## Build & Development

<!--
  List the build, test, lint, and format commands used in your project.
  NOTE: Also add these commands to `.claude/settings.local.json`:
    - "permissions.allow" — so that agents can execute them without prompting.
    - "hooks" — e.g. run linter/formatter automatically after file edits.
  Example permissions:
    "Bash(cargo build:*)",
    "Bash(cargo test:*)",
    "Bash(cargo clippy:*)",
    "Bash(cargo fmt:*)"
  Example hooks:
    "hooks": {
      "PostToolUse": [
        {
          "matcher": "Edit|Write",
          "hooks": [{ "type": "command", "command": "cargo fmt" }]
        }
      ]
    }
-->

```bash
# Build
# TODO: e.g. cargo build, npm run build, python -m build

# Test
# TODO: e.g. cargo test, npm test, pytest

# Lint
# TODO: e.g. cargo clippy, npm run lint, ruff check .

# Format
# TODO: e.g. cargo fmt, npm run format, ruff format .
```

## Architecture

<!-- Describe project-specific architecture here. Focus on the "big picture" that requires reading multiple files to understand. -->

<!-- ============================================================
  TEMPLATE FRAMEWORK SECTION
  The sections below are provided by the template and define the
  agent workflow. Normally no changes are needed.
============================================================ -->

## Skills

| Skill | Description |
|-------|-------------|
| `/spec [feature-name]` | Interactively create a feature specification at `docs/specs/{feature}/spec.md` |
| `/task [feature-name]` | Parse the spec's implementation plan and generate individual task files under `docs/specs/{feature}/tasks/` |
| `/implement [task description]` | Execute an iterative cycle: auto-select agent → design → implement → Codex review → iterate → cleanup |

## Agents

All agents use `model: opus`.

| Agent | Role | Description |
|-------|------|-------------|
| `rust-engineer` | Implementation | Write Rust code following stack-specific best practices |
| `typescript-engineer` | Implementation | Write TypeScript code following stack-specific best practices |
| `python-engineer` | Implementation | Write Python code following stack-specific best practices |
| `codex-code-reviewer` | Review | Delegate review to OpenAI Codex CLI (`codex exec`). Supports Full Review, Validation Review, and Spec Review modes |
| `code-simplifier` (built-in) | Cleanup | Simplify and refine code as the final phase after implementation iterations pass review |

Implementation agents are selected automatically by `/implement` based on task context (file extensions, project config, codebase analysis).

## Conventions

- Use ASCII art for diagrams in CLI output. Use Mermaid for diagrams in Markdown files (specs, tasks, etc.).
- Specifications and tasks are stored under `docs/specs/{feature-name}/` and tracked via Update Logs with timestamps (`YYYY-MM-DD HH:MM:SS`).
- Feature names use kebab-case (e.g., `user-auth`, `data-pipeline`).
- Task files follow the naming pattern `task_{NNN}_{slug}.md`.
- Templates (spec-template.md, task-template.md) are written in Japanese.
- Plan mode outputs go to `~/.claude/plans/` by default — the `/implement` skill requires copying them back into `docs/specs/` for version control.
