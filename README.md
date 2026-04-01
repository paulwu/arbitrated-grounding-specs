# Curated Advisor Specs

A [spec-driven development](docs/spec-driven-development/README.md) framework for building **knowledge repositories with trusted, grounded AI agents** — where every answer traces back to sources you control.

Standard chatbots hallucinate freely. These specs define a different model: agents that follow strict source hierarchies, detect contradictions against your curated knowledge, and cite their sources — giving you far greater control over what the AI says and why. Each spec captures a battle-tested pattern (grounding rules, wizard flows, research pipelines, documentation architecture) as a parameterized template you can import into any repository.

**The feedback loop:** Use these specs to scaffold a new knowledge repo. As you refine patterns in your project, extract the improvements back into specs here — so every repo in the ecosystem benefits.

## The Arbitrated Grounding Pattern

Arbitrated Grounding is an architectural pattern for AI agents designed to resolve conflicts between curated insights and official documentation. It moves beyond simple retrieval by introducing a reconciliation loop that ranks information based on source authority and contextual relevance.

The pattern ensures that AI-generated content is not only "grounded" in data but is also audited against a human-governed "Sovereign Truth."

### Core Components

**1. The Curator** — *The Insightful Synthesizer*

- **Output:** Grounding
- The Curator performs the "effortful" task of sifting through raw data to select, organize, and contextualize high-signal information. It focuses on creating a cohesive narrative or "fact-set" that serves as the primary foundation for the system.

**2. The Researcher** — *The Final Arbitrator*

- **Output:** Answers
- The Researcher acts as the auditor. It cross-references the Grounding (from the Curator) against Official Documentation. Using a weighted ranking score (typically 0.7 for Official Docs / 0.3 for Curator Context), it resolves discrepancies to produce the final, verified truth.

**3. The Authority Record** — *The Sovereign Directive*

- **Output:** Override
- A human-governed registry that sits at the top of the hierarchy. If a query matches a rule in this record, the Override bypasses all AI logic. This ensures deterministic control over business rules, specific terminology, or known hallucination triggers.

### The Logic Flow

1. **Synthesis** — The Curator produces a refined Grounding set from raw inputs.
2. **Arbitration** — The Researcher compares Grounding vs. Official Docs. It applies ranking logic to resolve conflicts and outputs Answers.
3. **Enforcement** — The system checks the Authority Record. If a match exists, the Override replaces the Answers for that specific data point.

### Why Use This Pattern?

- **Conflict Resolution** — Provides a mathematical way to handle "hallucinations" or outdated info by ranking sources.
- **Separation of Concerns** — Separates the creation of context (Curator) from the verification of facts (Researcher).
- **Deterministic Safety** — The Authority Record provides a "kill switch" to enforce human-verified truths when the AI is uncertain or incorrect.

## Documentation

👉 **[Full documentation →](docs/spec-driven-development/README.md)**

- [Spec File Format Reference](docs/spec-driven-development/spec-format.md) — YAML frontmatter, `{{VARIABLE}}` syntax, manifest and project config schemas
- [FAQ](docs/spec-driven-development/faq.md) — How export/import works, applying to other projects, drift detection, overrides

## Available Specs

| Spec | What It Captures |
|---|---|
| [`grounding-rules`](specs/grounding-rules.spec.md) | Source priority hierarchy, contradiction detection, citation format |
| [`research-conventions`](specs/research-conventions.spec.md) | YAML frontmatter (`Author`, `Priority`), priority scale (1-5+) |
| [`wizard-agent`](specs/wizard-agent.spec.md) | Prerequisite checks, `az account show` detection, autopilot warning, command generation |
| [`research-agent`](specs/research-agent.spec.md) | Fetch live docs, cross-reference cached notes, flag contradictions, save response |
| [`doc-architecture`](specs/doc-architecture.spec.md) | `notes/` → `docs/` → `scripts/` three-layer architecture |
| [`readme-structure`](specs/readme-structure.spec.md) | README with TOC, collapsible folder tree, agent table, prerequisite warnings |
| [`response-capture`](specs/response-capture.spec.md) | Response capture conventions — folder layout, filename format, metadata headers |
| [`author-agent`](specs/author-agent.spec.md) | Research-curator agent pattern — enforces frontmatter, priority headers, naming |
| [`advisor-agent`](specs/advisor-agent.spec.md) | Advisor agent pattern — grounded Q&A with source-cited synthesis |

## Quick Start

### 1. Copy the meta-agents into your project

**Option A — From a local clone of this repo:**

```bash
mkdir -p your-project/.github/agents
cp .github/agents/Spec-Importer.agent.md your-project/.github/agents/
cp .github/agents/Spec-Drift.agent.md    your-project/.github/agents/
```

**Option B — Download from GitHub (no clone needed):**

Run this from your project's root directory:

```bash
mkdir -p .github/agents
curl -sL https://raw.githubusercontent.com/paulwu/curated-advisor-specs/main/.github/agents/Spec-Importer.agent.md -o .github/agents/Spec-Importer.agent.md
curl -sL https://raw.githubusercontent.com/paulwu/curated-advisor-specs/main/.github/agents/Spec-Drift.agent.md    -o .github/agents/Spec-Drift.agent.md
```

> **Note:** You only need to copy the agents manually the first time. After that, `@spec-importer` will automatically sync the latest agent files from the spec repo on every import.

### 2. Import specs (interactive mode — press Shift+Tab first)

The importer auto-downloads specs from GitHub — no local clone needed:

```
@spec-importer Import grounding-rules and research-agent
```

If the agent can't reach GitHub, point it to a local clone instead:

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
| [agent365-management](https://github.com/paulwu/agent365-management) | All 9 specs (reference implementation) |

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
