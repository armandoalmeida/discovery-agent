# discovery-agent

A Claude Code plugin for **project discovery**: turn a raw idea into structured requirements, architecture, data model, and Architecture Decision Records - through a disciplined, question-first conversation.

This repository ships:

- `.claude-plugin/plugin.json` - the plugin manifest
- `agents/discovery.agent.md` - the Discovery agent profile
- `agents/refine.agent.md` - the Refine agent profile
- `skills/` - the phase skills (`grill-requirements`, `draft-architecture`, `draft-data-model`, `write-adr`, `audit-docs`, `refine-docs`)

The agents do **not** write source code. Their output is Markdown documents and diagrams.

## What it does

For each idea you bring, the Discovery agent walks 4 phases:

| Phase | Skill | Output |
| --- | --- | --- |
| 1. Grill requirements | `grill-requirements` | `requirements.md` |
| 2. Draft architecture | `draft-architecture` | `architecture.md` (+ topology diagram) |
| 3. Draft data model | `draft-data-model` | `data-model.md` |
| 4. Record decisions | `write-adr` | `decisions/NNNN-*.md` |

The Refine agent then keeps that doc set healthy: audits for gaps and contradictions, re-grills a specific finding, and applies approved edits via a bullet-diff preview.

Documents land in your current working directory under `<cwd>/<project-slug>/`, not inside this repo.

## Default technical stack

The agent frames every proposal around this stack first, notes alternatives only when warranted:

- **Language / framework**: Java + Spring Boot
- **Relational DB**: PostgreSQL
- **NoSQL DB**: MongoDB or DynamoDB (chosen by access pattern)
- **Cloud**: AWS

## Diagrams

- **Default**: Mermaid (component, sequence, ER, flowchart). No toolchain required.
- **Cloud / infra topology**: [mingrammer/diagrams](https://github.com/mingrammer/diagrams). Requires:
  - `brew install graphviz`
  - `pip3 install diagrams`

  The agent guides you to install these but never installs them itself.
- One format per diagram type per project - locked at first use.

## Installation

As a Claude Code plugin:

```
/plugin marketplace add <this-repo-url-or-path>
/plugin install discovery-agent
```

For local development, point Claude Code at this repo as a local plugin/marketplace source instead of publishing it.

## Using the agent

Once installed:

1. Start Claude Code.
2. Invoke the agent by name (`@discovery` / `@refine`, or via the harness's agent picker) or describe an idea and let the agent's description trigger it.
3. Answer questions. The agent will grill you until requirements are clear before proposing architecture.

The agent asks the working directory, language, and project slug at the start of each session.

## Repository layout

```
discovery-agent/
  CLAUDE.md                     # harness entrypoint (source of truth)
  README.md                     # this file
  .claude-plugin/
    plugin.json                 # plugin manifest
  agents/
    discovery.agent.md          # the orchestrator agent
    refine.agent.md             # the doc-refinement agent
  skills/
    grill-requirements/SKILL.md
    draft-architecture/SKILL.md
    draft-data-model/SKILL.md
    write-adr/SKILL.md
    audit-docs/SKILL.md
    refine-docs/SKILL.md
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) and the procedure body.
2. If the skill belongs in a phase, reference it from `agents/discovery.agent.md` or `agents/refine.agent.md`.

## Portability

Agent and skill files are plain Markdown with YAML frontmatter - no Claude-specific tool assumptions beyond `CLAUDE.md` itself. The `.claude-plugin/plugin.json` manifest is additive.
