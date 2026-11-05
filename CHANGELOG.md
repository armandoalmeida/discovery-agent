# Changelog

All notable changes to the `discovery-agent` plugin.
Versions follow semver; a release happens when `version` in `.claude-plugin/plugin.json` changes.

## 1.0.0

- Packaged as the `discovery-agent` Claude Code plugin (`.claude-plugin/plugin.json`); `AGENTS.md` merged into `CLAUDE.md` as the single harness entrypoint.
- `discovery` agent: 4-phase flow (`grill-requirements` -> `draft-architecture` -> `draft-data-model` -> `write-adr`) turning a raw idea into `requirements.md`, `architecture.md`, `data-model.md`, and one ADR per decision.
- `refine` agent: audits an existing doc set (`audit-docs`), re-grills a picked finding, and applies edits via a bullet-diff preview and atomic write (`refine-docs`).
- Default stack the agents propose: Java + Spring Boot, PostgreSQL or MongoDB/DynamoDB, AWS.
- Diagram policy: Mermaid by default, `mingrammer/diagrams` for cloud/infra topology, one format per diagram type locked at first use.
