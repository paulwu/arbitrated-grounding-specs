# Spec File Format Reference

## Overview

Each spec is a Markdown file with YAML frontmatter that describes a reusable pattern. Specs use `{{VARIABLE_NAME}}` mustache-style placeholders for project-specific values.

## File Structure

```markdown
---
spec: <spec-identifier>
version: "<semver>"
description: "<one-line description>"
extracted_from: "<owner/repo>"
requires: []                    # other specs this depends on
variables:
  - name: VARIABLE_NAME
    description: "<what this variable is>"
    required: true|false
    default: "<default value if optional>"
    example: "<example value>"
---

# <Spec Title>

## Pattern

<The reusable pattern, with {{VARIABLE_NAME}} placeholders>

## Template

<Optional: a file template that the importer can generate>
```

## YAML Frontmatter Properties

| Property | Required | Description |
|---|---|---|
| `spec` | ✅ | Unique identifier for the spec (kebab-case, e.g., `grounding-rules`) |
| `version` | ✅ | Semantic version (e.g., `1.0.0`) |
| `description` | ✅ | One-line description of the pattern |
| `extracted_from` | Recommended | The `owner/repo` this pattern was first extracted from |
| `requires` | No | List of other spec identifiers this spec depends on |
| `variables` | No | List of parameterized variables used in the pattern |

## Variable Syntax

Variables use mustache-style double braces: `{{VARIABLE_NAME}}`

```markdown
Treat live content from `{{PRIMARY_SOURCE_URL}}` as the highest-authority source.
Use `{{CACHED_BASELINE_FILE}}` as the cached baseline.
```

### Variable Definition

Each variable in the `variables` array has:

| Property | Required | Description |
|---|---|---|
| `name` | ✅ | Variable name (UPPER_SNAKE_CASE) |
| `description` | ✅ | What this variable represents |
| `required` | No | Whether the importer must collect this value (default: `true`) |
| `default` | No | Default value if the user doesn't provide one |
| `example` | Recommended | Example value shown to the user during import |

## Manifest File (`manifest.yaml`)

The spec repo root contains a `manifest.yaml` that indexes all available specs:

```yaml
name: arbitrated-grounding-specs
version: "2.0.0"
description: Reusable Copilot agent patterns for documentation knowledge bases

specs:
  - id: grounding-rules
    file: specs/grounding-rules.spec.md
    version: "2.0.0"
    description: Source priority hierarchy and contradiction detection

  - id: research-conventions
    file: specs/research-conventions.spec.md
    version: "2.0.0"
    description: YAML frontmatter format and priority scale for research notes

  - id: wizard-agent
    file: specs/wizard-agent.spec.md
    version: "2.0.0"
    description: Interactive wizard agent pattern with prerequisite checks

  - id: research-agent
    file: specs/research-agent.spec.md
    version: "2.0.0"
    description: Research agent pattern with grounding and contradiction detection

  - id: doc-architecture
    file: specs/doc-architecture.spec.md
    version: "2.0.0"
    description: Three-layer notes → docs → scripts architecture

  - id: readme-structure
    file: specs/readme-structure.spec.md
    version: "2.0.0"
    description: README structure with TOC, collapsible sections, and agent table

  - id: response-capture
    file: specs/response-capture.spec.md
    version: "2.0.0"
    description: Response capture conventions — folder layout, filename format, metadata headers

  - id: author-agent
    file: specs/author-agent.spec.md
    version: "2.0.0"
    description: Research-curator agent pattern — enforces frontmatter, priority headers, naming

  - id: advisor-agent
    file: specs/advisor-agent.spec.md
    version: "2.0.0"
    description: Advisor agent pattern — grounded Q&A with source-cited synthesis
```

## Project Config (`.spec-config.yaml`)

Each project that imports specs has a `.spec-config.yaml` in its root:

```yaml
spec_repo: paulwu/arbitrated-grounding-specs
spec_version: "2.0.0"
imported_at: "2026-03-28T06:00:00Z"

imports:
  - grounding-rules
  - research-conventions
  - research-agent
  - wizard-agent
  - doc-architecture
  - readme-structure

variables:
  # grounding-rules variables
  PRIMARY_SOURCE_URL: "https://learn.microsoft.com/en-us/entra/agent-id/"
  CACHED_BASELINE_FILE: "notes/Microsoft-Learn-Entra-AgentID.md"
  SECONDARY_NOTE_FILES: "ChatGPT.md, Gemini.md, Researcher.md, Microsoft-Learn.md"

  # research-agent variables
  RESEARCH_AGENT_NAME: "Entra Researcher"
  RESEARCH_AGENT_DESCRIPTION: "Research agent grounded on official Microsoft Learn Entra Agent ID documentation."
  RESPONSE_FOLDER: "answers"
  RESPONSE_TIMEZONE: "America/Los_Angeles"

  # doc-architecture variables
  KNOWLEDGE_FOLDER: "notes"
  SYNTHESIZED_DOCS_FOLDER: "docs"
  AUTOMATION_FOLDER: "scripts"

  # Project-specific
  PROJECT_NAME: "Agent 365 Management"
  PROJECT_DESCRIPTION: "Knowledge base for Microsoft Agent 365 management, Entra Agent ID, and the Agent Registry"
```

## Spec Composition

Specs can reference other specs using the `requires` field:

```yaml
---
spec: research-agent
version: "1.0.0"
requires:
  - grounding-rules
  - research-conventions
---
```

This means the research-agent spec assumes the grounding-rules and research-conventions patterns are also applied. The importer will warn if a required spec is not in the project's import list.

## Versioning Rules

- **Patch** (1.0.0 → 1.0.1): Fix typos, clarify wording, no behavioral change
- **Minor** (1.0.0 → 1.1.0): Add optional features, new variables with defaults, backward-compatible
- **Major** (1.0.0 → 2.0.0): Breaking changes — renamed variables, restructured pattern, removed features

Projects pin to a version. The `@spec-drift` agent compares the project's `spec_version` against the latest and shows what changed.
