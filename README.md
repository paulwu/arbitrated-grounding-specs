# Curated Advisor Specs

A [spec-driven development](docs/spec-driven-development/README.md) framework for building **knowledge repositories with trusted, grounded AI agents** — where every answer traces back to sources you control.

Standard chatbots hallucinate freely. These specs define a different model: agents that follow strict source hierarchies, detect contradictions against your curated knowledge, and cite their sources — giving you far greater control over what the AI says and why. Each spec captures a battle-tested pattern (grounding rules, wizard flows, research pipelines, documentation architecture) as a parameterized template you can import into any repository.

**The feedback loop:** Use these specs to scaffold a new knowledge repo. As you refine patterns in your project, extract the improvements back into specs here — so every repo in the ecosystem benefits.

## Documentation

👉 **[Full documentation →](docs/spec-driven-development/README.md)**

- [Spec File Format Reference](docs/spec-driven-development/spec-format.md) — YAML frontmatter, `{{VARIABLE}}` syntax, manifest and project config schemas
- [FAQ](docs/spec-driven-development/faq.md) — How export/import works, applying to other projects, drift detection, overrides

## Available Specs

| Spec | What It Captures |
|---|---|
| [`grounding-rules`](specs/grounding-rules.spec.md) | Source priority hierarchy, contradiction detection, citation format |
| [`notes-conventions`](specs/notes-conventions.spec.md) | YAML frontmatter (`Author`, `Priority`), priority scale (1-5+) |
| [`wizard-agent`](specs/wizard-agent.spec.md) | Prerequisite checks, `az account show` detection, autopilot warning, command generation |
| [`research-agent`](specs/research-agent.spec.md) | Fetch live docs, cross-reference cached notes, flag contradictions, save response |
| [`doc-architecture`](specs/doc-architecture.spec.md) | `notes/` → `docs/` → `scripts/` three-layer architecture |
| [`readme-structure`](specs/readme-structure.spec.md) | README with TOC, collapsible folder tree, agent table, prerequisite warnings |

## Quick Start

### 1. Copy the meta-agents into your project

```bash
mkdir -p your-project/.github/agents
cp .github/agents/Spec-Importer.agent.md your-project/.github/agents/
cp .github/agents/Spec-Drift.agent.md    your-project/.github/agents/
```

> **Note:** You only need to copy the agents manually the first time. After that, `@spec-importer` will automatically sync the latest agent files from the spec repo on every import.

### 2. Import specs (interactive mode — press Shift+Tab first)

```
@spec-importer Import grounding-rules and research-agent specs from ~/curated-advisor-specs/specs/
```

The importer collects your project-specific values (URLs, folder names, agent names) and generates the corresponding files — including a lightweight `docs/spec-driven-development.md` guide that links back to the full documentation here.

### 3. Check for drift later

```
@spec-drift Compare this project against its imported specs
```

## Projects Using These Specs

| Project | Specs Imported |
|---|---|
| [agent365-management](https://github.com/paulwu/agent365-management) | All 6 specs (reference implementation) |

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│  curated-advisor-specs (this repo)                           │
│  Parameterized specs + meta-agents                           │
│                                                              │
│         ┌──────────┐      ┌──────────┐                       │
│         │ Import   │      │ Extract  │                       │
│         │ specs ↓  │      │ back ↑   │                       │
│         └─────┬────┘      └────┬─────┘                       │
└───────────────┼────────────────┼─────────────────────────────┘
                │                │
       ┌────────▼────────┐  ┌───┴────────────────┐
       │ Your Knowledge  │  │ Another Knowledge  │
       │ Repo            │  │ Repo               │
       │ (grounded       │  │ (refines patterns, │
       │  agents)        │  │  feeds them back)  │
       └─────────────────┘  └────────────────────┘
```

Specs flow **into** projects via `@spec-importer`. Pattern improvements flow **back** via `@spec-exporter`. `@spec-drift` keeps everything in sync.

## How It Was Built

These specs were extracted from the [agent365-management](https://github.com/paulwu/agent365-management) project — a knowledge base for Microsoft Agent 365 / Entra Agent ID. The patterns (source hierarchies, contradiction detection, wizard flows) evolved through iterative development and were generalized into parameterized specs so any repository can adopt and improve them.

## License

This project is licensed under the [GNU General Public License v3.0](LICENSE).
