# Spec-Driven Development

> A framework for extracting, sharing, and synchronizing reusable Copilot agent patterns across repositories.

## What Is This?

This project uses a set of reusable patterns — grounding rules, wizard flows, notes conventions, documentation architecture — that are valuable across multiple repositories. **Spec-driven development** externalizes these patterns into parameterized specification files that can be versioned, shared, and imported into any project.

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    arbitrated-grounding-specs/                    │
│                    (dedicated spec repo)                     │
│                                                             │
│  specs/                                                     │
│  ├── grounding-rules.spec.md      Source hierarchy rules    │
│  ├── research-conventions.spec.md Research frontmatter/priority│
│  ├── wizard-agent.spec.md         Prerequisite/wizard flow  │
│  ├── research-agent.spec.md       Fetch/cross-ref/cite      │
│  ├── doc-architecture.spec.md     notes→docs→scripts layers │
│  ├── readme-structure.spec.md     TOC, collapsible, agents  │
│  ├── answer-capture.spec.md       Response capture layout   │
│  ├── author-agent.spec.md         Research-curator agent    │
│  └── advisor-agent.spec.md        Grounded Q&A advisor      │
│                                                             │
│  .github/agents/                                            │
│  ├── Spec-Exporter.agent.md       Extracts patterns → specs │
│  ├── Spec-Importer.agent.md       Applies specs → project   │
│  └── Spec-Drift.agent.md          Compares project vs specs │
│                                                             │
│  manifest.yaml                    Version, spec index       │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
     ┌─────────▼──────────┐    ┌──────────▼──────────┐
     │ agent365-management│    │azure-resilience-adv. │
     │ (imports specs)    │    │(imports specs)        │
     └────────────────────┘    └──────────────────────┘
```

### Three Meta-Agents

| Agent | Purpose |
|---|---|
| **`@spec-exporter`** | Reads a project's agent files, copilot-instructions, README, and notes → generates parameterized spec files to a local folder |
| **`@spec-importer`** | Reads spec files, collects project-specific variable values, and generates/updates project files (copilot-instructions.md, agent files, README structure, framework guide) |
| **`@spec-drift`** | Compares a project's current state against its imported specs and reports divergences with actionable diffs |

## Available Specs

| Spec | What It Captures |
|---|---|
| [grounding-rules.spec.md](../../specs/grounding-rules.spec.md) | Source priority hierarchy, contradiction detection, and citation format |
| [research-conventions.spec.md](../../specs/research-conventions.spec.md) | YAML frontmatter format and priority scale for knowledge notes |
| [wizard-agent.spec.md](../../specs/wizard-agent.spec.md) | Interactive wizard with prerequisite checks, script execution or command handoff |
| [research-agent.spec.md](../../specs/research-agent.spec.md) | Research agent with live doc fetching, cross-referencing, and contradiction detection |
| [doc-architecture.spec.md](../../specs/doc-architecture.spec.md) | Three-layer knowledge → docs → scripts documentation architecture |
| [readme-structure.spec.md](../../specs/readme-structure.spec.md) | README structure with TOC, collapsible folder tree, and agent table |
| [answer-capture.spec.md](../../specs/answer-capture.spec.md) | Response capture convention for saving agent responses to timestamped markdown files |
| [author-agent.spec.md](../../specs/author-agent.spec.md) | Knowledge authoring agent pattern for creating and maintaining structured content |
| [advisor-agent.spec.md](../../specs/advisor-agent.spec.md) | Domain advisor agent pattern for answering questions grounded on a structured knowledge base |

## Quick Start

### Importing specs into an existing project

```bash
# 1. Copy the meta-agents into your project
cp arbitrated-grounding-specs/.github/agents/Spec-Importer.agent.md \
   your-project/.github/agents/

# 2. In Copilot Chat (interactive mode):
@spec-importer Import grounding-rules and research-agent specs from ~/arbitrated-grounding-specs/specs/
```

### Exporting patterns from a project

```bash
# 1. Copy the exporter agent into your project
cp arbitrated-grounding-specs/.github/agents/Spec-Exporter.agent.md \
   your-project/.github/agents/

# 2. In Copilot Chat:
@spec-exporter Extract the grounding rules pattern from this project into specs/
```

### Updating specs after a new version

```
# 1. Check for updates
@spec-drift Compare this project against its imported specs

# 2. Re-import (handles all change types automatically)
@spec-importer Re-import specs

# 3. Verify the update is clean
@spec-drift Compare this project against its imported specs
```

### Checking for drift

```
@spec-drift Compare this project against the specs in ~/arbitrated-grounding-specs/specs/
```

## Spec Update Lifecycle

When specs are updated in the spec repo, consuming projects follow a four-step cycle to realize the changes:

```
┌──────────────────────────────────────────────────────────────────┐
│                     Spec Update Lifecycle                         │
│                                                                  │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌─────────┐ │
│  │ 1.DETECT │────▶│ 2.REVIEW │────▶│ 3.APPLY  │────▶│4.VERIFY │ │
│  │          │     │          │     │          │     │         │ │
│  │@spec-    │     │ Read     │     │@spec-    │     │@spec-   │ │
│  │ drift    │     │ changelog│     │ importer │     │ drift   │ │
│  └──────────┘     └──────────┘     └──────────┘     └─────────┘ │
│       │                                                   │      │
│       └──────────── Clean? Done! ◄────────────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

### What the importer handles during re-import

The `@spec-importer` detects ALL types of spec changes when it finds an existing `.spec-config.yaml`:

| Change Type | Detection | Resolution |
|---|---|---|
| New variable | Missing from config | Prompts for value |
| Changed variable metadata | Description/default differs | Informational notice |
| Removed variable | In config but not in spec | Offers to clean up |
| New dependency | Spec added a `requires` | Auto-adds or prompts |
| New/changed sections | Template content differs | Shows diff, asks to accept |
| New artifact | Referenced file doesn't exist | Offers to scaffold placeholder |
| Version bump | Config version < manifest | Updates config after re-import |

For the full details on each change type, see the [FAQ — Applying Spec Updates](./faq.md#applying-spec-updates-to-your-project).

## Further Reading

- [Spec File Format](./spec-format.md) — detailed format reference, variable syntax, examples
- [FAQ](./faq.md) — common questions about the spec-driven approach

## References

- This approach was developed in the [agent365-management](https://github.com/paulwu/agent365-management) project as a way to synchronize Copilot agent patterns across repositories.
