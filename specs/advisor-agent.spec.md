---
spec: advisor-agent
version: "2.1.0"
description: Domain advisor agent pattern for answering questions grounded on a structured knowledge base
extracted_from: paulwu/azure-rbac-advisor
requires:
  - grounding-rules
  - response-capture
variables:
  - name: ADVISOR_AGENT_NAME
    description: "Name of the advisor agent"
    required: true
    example: "Azure RBAC Advisor"
  - name: ADVISOR_AGENT_DESCRIPTION
    description: "One-line description of the advisor agent"
    required: true
    example: "Answers Azure RBAC least-privilege questions grounded on the resources/ reference library."
  - name: KNOWLEDGE_BASE_FOLDER
    description: "Folder the advisor reads from as its primary knowledge source"
    required: true
    example: "resources"
  - name: ADVISOR_SCOPE
    description: "Domain the advisor covers — used in off-topic redirect messages"
    required: true
    example: "Azure RBAC least-privilege permissions"
  - name: ANSWER_FOLDER
    description: "Folder where advisor answers are saved"
    required: false
    default: "answer"
  - name: LOG_FOLDER
    description: "Folder where advisor prompt logs are saved (typically gitignored)"
    required: false
    default: "log"
  - name: CORRECTIONS_FILE
    description: "Inherited from grounding-rules. Path to a corrections/overrides file. See grounding-rules spec for hierarchy details."
    required: false
    default: ""
    example: ".override-rules/corrections.md"
---

# Advisor Agent Spec

## Pattern

### Agent Behavior

The advisor agent follows this workflow for every question:

1. **Search the knowledge base** — use glob/grep to find relevant files in `{{KNOWLEDGE_BASE_FOLDER}}/`
2. **Read the matching files** — load content from the knowledge base
3. **Check corrections** — if `{{CORRECTIONS_FILE}}` is defined, apply any overrides before answering
4. **Answer from grounded content** — compose the answer using only verified knowledge base content
5. **Cite sources** — always reference which files the answer draws from
6. **Log the prompt** — save to `{{LOG_FOLDER}}/`
7. **Save the answer** — save to `{{ANSWER_FOLDER}}/`

### Off-Topic Redirect

When the user asks something outside the advisor's domain (`{{ADVISOR_SCOPE}}`), redirect with a scoping flow:

1. Acknowledge the question is outside scope
2. Ask clarifying questions to narrow to an in-scope topic (one question at a time)
3. Only generate an answer once the question is in scope

### Source Citation

Every answer must cite its source files:

```markdown
> 📄 Source: `{{KNOWLEDGE_BASE_FOLDER}}/<path>/<file>.md`
```

If the answer draws from multiple files, cite all of them.

### Logging

Every prompt must be logged automatically:

- **Log folder:** `{{LOG_FOLDER}}/` (typically excluded from git)
- **Log filename:** timestamp-based, project-specific format
- **Log content:** the original prompt, timestamp, and any metadata

### Answer Saving

Every answer must be saved automatically:

- **Answer folder:** `{{ANSWER_FOLDER}}/` (typically tracked in git for quality comparison)
- **Answer filename:** timestamp-based, matching the log filename convention
- **Answer content:** the full answer with source citations

### Corrections and Overrides

This spec uses the corrections layer defined in the **grounding-rules** spec. If `{{CORRECTIONS_FILE}}` is defined (inherited from grounding-rules), the advisor must:

1. Read the corrections file before generating any answer
2. Apply factual overrides, deprecated command replacements, and pinned values
3. Corrections take precedence over knowledge base content and agent knowledge (per the grounding-rules source hierarchy)
4. Protected/locked content in the knowledge base takes precedence over corrections

See `grounding-rules.spec.md` for the full source hierarchy and contradiction detection rules.

### Agent Profile Template

```yaml
---
name: {{ADVISOR_AGENT_NAME}}
description: {{ADVISOR_AGENT_DESCRIPTION}}
tools: ["read", "search", "grep", "glob", "write", "edit", "bash"]
---
```

### Key Rules

- **Read knowledge base; never modify it** — the advisor reads `{{KNOWLEDGE_BASE_FOLDER}}/` but does not write to it
- **Write output only to `{{LOG_FOLDER}}/` and `{{ANSWER_FOLDER}}/`** — never to the knowledge base
- **Never invent facts** — if the knowledge base doesn't contain the answer, say so clearly
- **Always cite sources** — every claim must reference the specific file it comes from
- **Log every interaction** — prompts are logged before generating answers

### Requirements for copilot-instructions.md

The project's agent instructions should reference the advisor agent and explain:

1. The agent's name and purpose
2. That it reads from `{{KNOWLEDGE_BASE_FOLDER}}/` as a reference library
3. That it saves logs and answers to `{{LOG_FOLDER}}/` and `{{ANSWER_FOLDER}}/`
4. That it must not modify knowledge base files
