---
spec: research-agent
version: "2.0.0"
description: Research agent pattern with live doc fetching, note cross-referencing, contradiction detection, and response capture
extracted_from: paulwu/agent365-management
requires:
  - grounding-rules
  - research-conventions
  - response-capture
variables:
  - name: RESEARCH_AGENT_NAME
    description: "Name of the research agent"
    required: true
    example: "Entra Researcher"
  - name: RESEARCH_AGENT_DESCRIPTION
    description: "One-line description of the agent's expertise"
    required: true
    example: "Research agent grounded on official Microsoft Learn Entra Agent ID documentation."
  - name: PRIMARY_SOURCE_URL
    description: "The authoritative documentation URL"
    required: true
    example: "https://learn.microsoft.com/en-us/entra/agent-id/"
  - name: PRIMARY_SOURCE_NAME
    description: "Human-readable name for the primary source"
    required: true
    example: "Microsoft Learn"
  - name: CACHED_BASELINE_FILE
    description: "Path to the cached baseline note file"
    required: true
    example: "grounding/Microsoft-Learn-Entra-AgentID.md"
  - name: SECONDARY_NOTE_FILES
    description: "Comma-separated list of secondary note filenames"
    required: false
    default: ""
    example: "ChatGPT.md, Gemini.md, Researcher.md"
  - name: SITE_INDEX_URLS
    description: "Markdown table of documentation page URLs the agent can fetch"
    required: false
    default: ""
  - name: SCRIPTS_FOLDER
    description: "Folder containing automation scripts to reference"
    required: false
    default: "scripts"
---

# Research Agent Spec

## Pattern

### Agent Behavior

The research agent follows this workflow for every question:

1. **Fetch the relevant page(s)** from the primary source using `web_fetch` or `web_search`
2. **Cross-reference** with the cached baseline in `{{CACHED_BASELINE_FILE}}`
3. **Check other note files** in `{{KNOWLEDGE_FOLDER}}/` ({{SECONDARY_NOTE_FILES}}) for additional context
4. **Flag contradictions** between sources (see grounding-rules spec)
5. **Reference repository scripts** in `{{SCRIPTS_FOLDER}}/` when a workflow can be expedited with existing automation
6. **Save every response** to `{{RESPONSE_CAPTURE_FOLDER}}/`

### Script References

When the answer involves a workflow that a script in `{{SCRIPTS_FOLDER}}/` covers, always mention the script as a ready-to-use alternative alongside the raw API calls. Show the Quick Start commands from the script's README.

### Agent Profile Template

```yaml
---
name: {{RESEARCH_AGENT_NAME}}
description: {{RESEARCH_AGENT_DESCRIPTION}}
---
```

The agent prompt should include:
- Primary source declaration and fetching instructions
- Cross-reference instructions for cached baseline and secondary notes
- Contradiction detection rules (from grounding-rules spec)
- Script-to-topic mapping table
- Response capture instructions
- Site index of fetchable documentation URLs (if available)
