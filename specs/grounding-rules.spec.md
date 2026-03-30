---
spec: grounding-rules
version: "2.0.0"
description: Source priority hierarchy, contradiction detection, and citation format for knowledge-base repositories
extracted_from: paulwu/agent365-management
requires: []
variables:
  - name: PRIMARY_SOURCE_URL
    description: "The authoritative documentation URL (highest-priority source)"
    required: true
    example: "https://learn.microsoft.com/en-us/entra/agent-id/"
  - name: PRIMARY_SOURCE_NAME
    description: "Human-readable name for the primary source"
    required: true
    example: "Microsoft Learn"
  - name: CACHED_BASELINE_FILE
    description: "Path to the cached baseline note file"
    required: true
    example: "research/Microsoft-Learn-Entra-AgentID.md"
  - name: SECONDARY_NOTE_FILES
    description: "Comma-separated list of secondary research note filenames"
    required: false
    default: ""
    example: "ChatGPT.md, Gemini.md, Researcher.md"
  - name: KNOWLEDGE_FOLDER
    description: "Folder containing knowledge notes (research, cached docs, etc.)"
    required: false
    default: "research"
  - name: DOCS_FOLDER
    description: "Folder containing generated documentation"
    required: false
    default: "docs"
---

# Grounding Rules Spec

## Pattern

### Source Hierarchy

All factual answers must follow this priority order:

1. **Live content** from `{{PRIMARY_SOURCE_URL}}` — always the highest-authority source
2. **Cached baseline** in `{{CACHED_BASELINE_FILE}}` — use when live fetches are unavailable
3. **Secondary research notes** in `{{KNOWLEDGE_FOLDER}}/` ({{SECONDARY_NOTE_FILES}}) — supporting context only
4. **Generated docs** in `{{DOCS_FOLDER}}/` — treat as output, NOT as a factual source of truth

### Contradiction Detection

When information from a note in `{{KNOWLEDGE_FOLDER}}/` conflicts with live or cached {{PRIMARY_SOURCE_NAME}} content:

1. **Flag the contradiction explicitly** with a ⚠️ warning
2. **List every conflicting source** — include the note's file path, Author (from frontmatter), and Priority alongside the {{PRIMARY_SOURCE_NAME}} page URL
3. **Prefer the {{PRIMARY_SOURCE_NAME}} version** as authoritative
4. Still show the disagreeing note's content so the user can decide whether to update it
5. Remind the user they can correct the note using `@Research-Curator`

### Contradiction Output Template

```
⚠️ **Contradiction detected:**

| Source | Says | Author | Priority |
|---|---|---|---|
| {{PRIMARY_SOURCE_NAME}} ([Page Title](url)) | <what the primary source says> | — | — |
| `{{KNOWLEDGE_FOLDER}}/<file>.md` | <what the note says> | <Author from frontmatter> | <Priority> |

**The {{PRIMARY_SOURCE_NAME}} version is authoritative.** If the note is outdated, you can update it with `@Research-Curator`.
```

### Priority-Based Conflict Resolution

When two notes disagree with each other (not with the primary source):

- **Lower Priority number = higher importance** — prefer the note with the lower number
- **Always present both sides** — list every conflicting note with file path, Author, and Priority
- When citing a note, include its `Author` (from the YAML frontmatter)

### copilot-instructions.md Requirements

The `.github/copilot-instructions.md` file MUST contain a "Canonical sources and grounding" section with the following required elements:

1. **Primary source declaration** — a statement establishing `{{PRIMARY_SOURCE_URL}}` as the highest-authority source
2. **Cached baseline reference** — a reference to `{{CACHED_BASELINE_FILE}}` as the fallback when live fetches are unavailable
3. **Secondary notes reference** — a statement that files in `{{KNOWLEDGE_FOLDER}}/` are secondary research only
4. **Generated docs caveat** — a statement that `{{DOCS_FOLDER}}/` is generated output, not the source of truth
5. **Contradiction handling** — instructions to flag contradictions with {{PRIMARY_SOURCE_NAME}} and prefer the primary source

Additional project-specific content in this section (introductory paragraphs, agent cross-references, exception clauses) is allowed and should not be flagged as drift.
