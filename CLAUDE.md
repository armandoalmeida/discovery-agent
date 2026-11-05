# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repo is the **discovery-agent** plugin. It ships two agents:

- `agents/discovery.agent.md` - turns a raw idea into a fresh doc set.
- `agents/refine.agent.md` - keeps an existing doc set healthy.

Read the relevant agent profile on every new session.

## What this repo is

A Claude Code plugin for **Discovery** and **Refine**, two project-ideation AI agents. There is no build system, test runner, or compiler.

Key artefacts:

- `.claude-plugin/plugin.json` - the plugin manifest (name, description, version).
- `agents/discovery.agent.md` - the Discovery agent profile (orchestrator, Phases 1-4).
- `agents/refine.agent.md` - the Refine agent profile (audits + updates an existing doc set).
- `skills/grill-requirements/SKILL.md` - Phase 1: turn a raw idea into `requirements.md`.
- `skills/draft-architecture/SKILL.md` - Phase 2: produce `architecture.md` + topology diagram.
- `skills/draft-data-model/SKILL.md` - Phase 3: produce `data-model.md` with access-pattern-first design.
- `skills/write-adr/SKILL.md` - Phase 4: write one ADR per non-obvious decision.
- `skills/audit-docs/SKILL.md` - Refine phase A: scan an existing doc set for gaps, contradictions, stale open questions. Read-only.
- `skills/refine-docs/SKILL.md` - Refine phase C: bullet-diff preview then atomic multi-file write.

## Installation

As a Claude Code plugin:

```
/plugin marketplace add <this-repo-url-or-path>
/plugin install discovery-agent
```

Or for local development, point Claude Code at the repo directly (`--plugin-dir <path-to-this-repo>` / add it as a local marketplace entry).

## Architecture

**`agents/discovery.agent.md`** and **`agents/refine.agent.md`** are the single sources of truth for their respective agent behavior. YAML frontmatter (`name`, `description`, `color`) followed by the full specification.

**Skills** live under `skills/<name>/SKILL.md`. Each has YAML frontmatter (`name`, `description`) followed by the procedure. A skill's `description` must be specific enough that another agent (or the harness) can decide when to trigger it.

**The agents write documents into the user's current working directory**, not into this repo. This repo only ships the plugin manifest, agent profiles, and skills.

## What the discovery agent produces

For each project, the agent walks 4 phases, each mapped to one skill:

| Phase | Skill | Output file |
| --- | --- | --- |
| 1 | `grill-requirements` | `<cwd>/<slug>/requirements.md` |
| 2 | `draft-architecture` | `<cwd>/<slug>/architecture.md` (+ `diagrams/*.py` / `*.png`) |
| 3 | `draft-data-model` | `<cwd>/<slug>/data-model.md` |
| 4 | `write-adr` | `<cwd>/<slug>/decisions/NNNN-*.md`, one per decision |

## What the refine agent produces

The refine agent runs standalone against an existing doc set. It does not create the doc set from scratch - if `requirements.md` is missing, it refuses and redirects to `discovery`. Its output is edits to existing files, gated by a bullet-diff preview and an explicit user `apply`.

| Phase | Skill | Effect |
| --- | --- | --- |
| A | `audit-docs` | Scan the doc set, rank findings blocking > important > nice-to-have. Read-only. |
| B | `grill-requirements` / `draft-architecture` / `draft-data-model` (grill-only mode) | Re-run the matching skill's `Question dimensions` for the picked finding. No file writes. |
| B (ADR) | `write-adr` | Full end-to-end ADR creation when a finding requires a new decision record. |
| C | `refine-docs` | Bullet-diff preview across every affected file, then sequential write on `apply`. Deletes resolved open questions and appends a `## Change log` entry to every touched file. |

## Default technical stack the agent proposes

- Java + Spring Boot
- PostgreSQL (relational) or MongoDB / DynamoDB (NoSQL, chosen by access pattern)
- AWS

## Diagram policy

- Default format: **Mermaid** (component, sequence, ER, flowchart).
- Cloud / infra topology: **mingrammer/diagrams** (Python + Graphviz). Requires `python3 -m pip install diagrams` and `brew install graphviz`. Agent guides the user to install; it does not install itself.
- One format per diagram type per project - locked at first use.

## Writing conventions

- Never use the em dash. Use plain `-`.
- Markdown: one full sentence per line, no mid-sentence wrapping. Keeps diffs readable.
- Skill and agent files require YAML frontmatter with at minimum `name` and `description`.

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with YAML frontmatter and a specific `description`.
2. Reference the new skill from the relevant agent profile (`agents/discovery.agent.md` or `agents/refine.agent.md`) if it belongs in a phase.

## Adding a new agent

1. Create `agents/<name>.agent.md` with YAML frontmatter (`name`, `description`, `color`) and the full agent specification.
