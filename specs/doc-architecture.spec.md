---
spec: doc-architecture
version: "2.0.0"
description: Three-layer research → docs → scripts architecture for documentation knowledge bases
extracted_from: paulwu/agent365-management
requires: []
variables:
  - name: KNOWLEDGE_FOLDER
    description: "Folder for knowledge notes and cached documentation"
    required: false
    default: "research"
  - name: SYNTHESIZED_DOCS_FOLDER
    description: "Folder for synthesized topic guides"
    required: false
    default: "docs"
  - name: AUTOMATION_FOLDER
    description: "Folder for automation scripts"
    required: false
    default: "scripts"
  - name: PROJECT_NAME
    description: "Name of the project"
    required: true
    example: "Agent 365 Management"
  - name: PROJECT_DESCRIPTION
    description: "One-line description of the project"
    required: true
    example: "Knowledge base for Microsoft Agent 365 management"
---

# Documentation Architecture Spec

## Pattern

### Three-Layer Architecture

The repository has three working layers:

1. **`{{KNOWLEDGE_FOLDER}}/`** — Raw research and cached documentation (primary knowledge source)
2. **`{{SYNTHESIZED_DOCS_FOLDER}}/`** — Synthesized topic guides (generated from the research)
3. **`{{AUTOMATION_FOLDER}}/`** — Automation scripts and tooling that operationalize the documentation

### Information Flow

```
{{KNOWLEDGE_FOLDER}}/     →    {{SYNTHESIZED_DOCS_FOLDER}}/     →    {{AUTOMATION_FOLDER}}/
(raw research)                (synthesized guides)              (operational scripts)
```

- **New information** → add to `{{KNOWLEDGE_FOLDER}}/` first
- **End-user guides** → generate or update in `{{SYNTHESIZED_DOCS_FOLDER}}/`
- **Operational automation** → implement in `{{AUTOMATION_FOLDER}}/`

### Rules

- Keep factual answers grounded in the primary source first, then `{{KNOWLEDGE_FOLDER}}/`
- Do not answer from `{{SYNTHESIZED_DOCS_FOLDER}}/` alone — it's generated output, not the source of truth
- If you add or remove any file in `{{SYNTHESIZED_DOCS_FOLDER}}/`, update `README.md` in both the structure listing and the topic-guide table
- Add new research to `{{KNOWLEDGE_FOLDER}}/`; add or update end-user guides in `{{SYNTHESIZED_DOCS_FOLDER}}/`

### copilot-instructions.md Requirements

The `.github/copilot-instructions.md` file MUST contain a "Repository architecture" section with the following required elements:

1. **Three-layer declaration** — identifies `{{KNOWLEDGE_FOLDER}}/`, `{{SYNTHESIZED_DOCS_FOLDER}}/`, and `{{AUTOMATION_FOLDER}}/` as the three working layers
2. **Layer descriptions** — explains the purpose of each layer (knowledge source, synthesized guides, operational scripts/tooling)

Additional project-specific content in this section (cross-file workflows, build commands, codebase conventions, key file lists) is allowed and should not be flagged as drift.

### Documentation Conventions

- Use `###` step-oriented headings in `{{SYNTHESIZED_DOCS_FOLDER}}/`
- Use Markdown tables for role/license/permission mappings
- Include `References` sections with primary source links
- Use Mermaid diagrams for multi-step relationships
