---
spec: author-agent
version: "2.0.0"
description: Knowledge authoring agent pattern for creating and maintaining structured content in a knowledge base
extracted_from: paulwu/agent365-management
requires:
  - research-conventions
variables:
  - name: AUTHOR_AGENT_NAME
    description: "Name of the authoring agent"
    required: true
    example: "Research Curator"
  - name: AUTHOR_AGENT_DESCRIPTION
    description: "One-line description of the authoring agent"
    required: true
    example: "Agent for creating and maintaining research notes. Ensures all notes have proper Author and Priority headers."
  - name: KNOWLEDGE_FOLDER
    description: "Folder containing knowledge notes"
    required: false
    default: "research"
  - name: CONTENT_TEMPLATE_PATH
    description: "Path to a domain-specific content template file that defines mandatory sections for authored content. Leave empty if content is free-form."
    required: false
    default: ""
    example: ".advisor-rules/content-template.md"
---

# Author Agent Spec

## Pattern

### Agent Behavior

The author agent is responsible for:

1. **Creating new content** in `{{KNOWLEDGE_FOLDER}}/` with proper formatting
2. **Validating existing content** for structural completeness and required metadata
3. **Enforcing frontmatter** — every note must have the required YAML frontmatter fields (see research-conventions spec)
4. **Preserving existing styles** — when editing, maintain the original citation format and structure

### Two Modes

#### Author Mode

When creating new content:

1. Validate that all required metadata fields are present (Author, Priority at minimum)
2. If the user does not specify Author or Priority, ask before creating
3. Apply the domain-specific content template if `{{CONTENT_TEMPLATE_PATH}}` is set
4. Place the file in `{{KNOWLEDGE_FOLDER}}/`
5. Self-validate the output before confirming done

#### Validate Mode

When checking existing content:

1. Scan files in `{{KNOWLEDGE_FOLDER}}/` for structural completeness
2. Check required YAML frontmatter fields are present and valid
3. If a content template is defined, verify sections match the template
4. Produce a report with errors and warnings
5. Offer to auto-fix issues on user request

### Agent Profile Template

```yaml
---
name: {{AUTHOR_AGENT_NAME}}
description: {{AUTHOR_AGENT_DESCRIPTION}}
---
```

### Key Rules

- **Write to `{{KNOWLEDGE_FOLDER}}/` only** — the author agent creates and edits knowledge files
- **Never skip validation** — always check content against required format before confirming
- **Preserve existing content** — when editing, maintain citation styles and formatting conventions
- **Ask before overwriting** — if a file already exists, confirm with the user before replacing
- **Use `ask_user`** for collecting metadata values when not provided

### Requirements for copilot-instructions.md

The project's agent instructions should reference the author agent and explain:

1. The agent's name and invocation command (`@{{AUTHOR_AGENT_NAME}}`)
2. That it enforces frontmatter requirements from the research-conventions spec
3. That it writes to `{{KNOWLEDGE_FOLDER}}/`
