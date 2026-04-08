---
spec: research-conventions
version: "2.1.0"
description: YAML frontmatter format, priority scale, and authoring rules for research notes
extracted_from: paulwu/agent365-management
requires: []
variables:
  - name: KNOWLEDGE_FOLDER
    description: "Folder containing knowledge notes"
    required: false
    default: "grounding"
  - name: CURATOR_AGENT_NAME
    description: "Name of the curation agent"
    required: false
    default: "Entra Curator"
---

# Research Conventions Spec

## Pattern

### Required YAML Frontmatter

Every note file in `{{KNOWLEDGE_FOLDER}}/` **must** begin with a YAML frontmatter block:

```yaml
---
Author: <name of the person or AI assistant that produced the content>
Priority: <integer, 1 = highest importance, higher = lower importance>
---
```

### Priority Scale

| Priority | Source Type |
|---|---|
| 0 | Human-authored corrections and overrides (factual fixes, deprecated replacements, pinned values) |
| 1 | Reserved — for notes verified against live primary source content in the current session |
| 2 | Cached primary source documentation (compiled directly but may be stale) |
| 3 | Other official documentation sources |
| 4 | AI assistant research and analysis (ChatGPT, Gemini, etc.) |
| 5+ | Community sources, informal notes, or unverified content |

### Rules

- **Every note must have frontmatter** — the `@{{CURATOR_AGENT_NAME}}` agent enforces this
- **Author field** — identifies who or what produced the content (human name or AI name)
- **Priority field** — determines which note wins when two sources conflict
- **Lower number = higher authority** — Priority 1 beats Priority 4
- **Live primary source always wins** regardless of any note's Priority value
- When citing a note, always include its `Author` from the frontmatter

### Notes Agent Template

The `@{{CURATOR_AGENT_NAME}}` agent is responsible for:

1. Creating new notes with proper frontmatter
2. Validating existing notes have the required fields
3. Assigning the correct Priority based on the source type
4. Preserving existing citation styles when editing notes
