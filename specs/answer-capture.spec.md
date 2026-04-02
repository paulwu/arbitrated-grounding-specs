---
spec: answer-capture
version: "2.1.0"
description: Response capture convention for saving agent responses to timestamped markdown files
extracted_from: paulwu/agent365-management
requires: []
variables:
  - name: RESPONSE_FOLDER
    description: "Folder where agent responses are saved"
    required: false
    default: "answers"
    example: "answers"
  - name: RESPONSE_TIMEZONE
    description: "Timezone for response file timestamps"
    required: false
    default: "America/Los_Angeles"
    example: "America/Los_Angeles"
  - name: RESPONSE_FILENAME_PREFIX
    description: "Prefix for response filenames"
    required: false
    default: "answer"
    example: "answer"
---

# Response Capture Spec

## Pattern

### When to Capture

After composing every response, save it to a markdown file in `{{RESPONSE_FOLDER}}/`.

### File Naming

Use the convention `{{RESPONSE_FILENAME_PREFIX}}-YY-MM-DD-HH-MM-SS.md` in **{{RESPONSE_TIMEZONE}}** timezone.

To get the timestamp, run:

```bash
TZ='{{RESPONSE_TIMEZONE}}' date '+%y-%m-%d-%H-%M-%S'
```

Example filename: `{{RESPONSE_FOLDER}}/{{RESPONSE_FILENAME_PREFIX}}-26-03-19-10-15-42.md`

### File Structure

The saved file must contain three sections:

```markdown
# Prompt

<the user's original question or prompt, quoted verbatim>

# Response

<the full response, including any contradiction warnings and tables>

# Sources

<list of every source consulted, in the format below>
```

### Sources Format

- **Knowledge notes**: `Author | {{KNOWLEDGE_FOLDER}}/<filename>` (e.g., `Microsoft Learn | grounding/Microsoft-Learn-Entra-AgentID.md`)
- **Web / live docs**: the full URL (e.g., `https://learn.microsoft.com/en-us/entra/agent-id/...`)

### Requirements for copilot-instructions.md

The project's agent instructions should include:

1. A reference to the response capture folder (`{{RESPONSE_FOLDER}}/`)
2. The file naming convention with timezone
3. The three-section file structure requirement (Prompt / Response / Sources)

### Confirmation Message

After saving a response, the agent should confirm inline:

```
✅ Response saved to `{{RESPONSE_FOLDER}}/{{RESPONSE_FILENAME_PREFIX}}-YY-MM-DD-HH-MM-SS.md`
```
