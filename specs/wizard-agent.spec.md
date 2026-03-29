---
spec: wizard-agent
version: "1.0.0"
description: "Pattern for interactive agents that require user input and may invoke Entra ID sign-in, PowerShell scripts, or Azure CLI commands. Includes prerequisite checks (PowerShell availability, az account show tenant detection, Entra role and Graph permission verification), autopilot-mode warning, input collection via ask_user, and command generation (never direct execution — scripts use browser-based device code auth). Examples: BluePrint-Creator (guides users through creating an Entra Agent ID blueprint via Create-Blueprint.ps1)."
extracted_from: paulwu/agent365-management
requires: []
variables:
  - name: WIZARD_AGENT_NAME
    description: "Name of the wizard agent"
    required: true
    example: "BluePrint-Creator"
  - name: WIZARD_DESCRIPTION
    description: "One-line description of what the wizard does"
    required: true
    example: "Interactive wizard that guides you through creating an Entra Agent ID blueprint"
  - name: TARGET_SCRIPT
    description: "Path to the PowerShell script this wizard drives"
    required: true
    example: "scripts/Create-Blueprint.ps1"
  - name: INPUT_JSON_FILE
    description: "Name of the JSON input file the wizard generates"
    required: false
    default: ""
    example: "blueprint-input.json"
  - name: INPUT_JSON_EXAMPLE
    description: "Path to the example JSON template"
    required: false
    default: ""
    example: "scripts/blueprint-input.json.example"
  - name: REQUIRED_ENTRA_ROLE
    description: "Entra role needed to run the script"
    required: true
    example: "Agent ID Developer"
  - name: REQUIRED_GRAPH_PERMISSIONS
    description: "Comma-separated Graph API permissions needed"
    required: true
    example: "AgentIdentityBlueprint.Create, AgentIdentityBlueprint.ReadWrite.All"
---

# Wizard Agent Spec

## Pattern

### Frontmatter

```yaml
---
name: {{WIZARD_AGENT_NAME}}
description: {{WIZARD_DESCRIPTION}}
tools: ["execute", "read", "edit", "search"]
---
```

### Autopilot Warning (Required)

Every wizard agent must include this warning immediately after the description:

```markdown
> **⚠️ Autopilot mode:** This wizard requires interactive input at multiple steps.
> If you are in **autopilot mode** (Shift+Tab to check), switch to **interactive mode**
> first — otherwise the wizard cannot wait for your answers and will skip ahead or
> terminate early.
```

### Standard Workflow Steps

Wizard agents follow this step pattern (steps can be added/removed based on the specific wizard):

```
Step 1: Check PowerShell availability
Step 2: Auto-detect tenant via az account show (fall back to manual input)
Step 3: Verify required Entra role
Step 4: Verify app registration and Graph permissions
Step N: Collect input fields (wizard-specific)
Step N+1: Generate input file (if applicable)
Step N+2: Provide the run command (do NOT execute — requires browser auth)
Step N+3: Explain expected output and next steps
```

### Step 1: PowerShell Check

```bash
pwsh --version 2>/dev/null || echo "PWSH_NOT_FOUND"
```

If not found, detect platform and attempt auto-install. If install fails, generate `docs/PowerShell-install.md`.

### Step 2: Tenant Auto-Detection

Always try `az account show` first:

```bash
az account show --query "{tenantId:tenantId,name:name,user:user.name}" --output json 2>/dev/null || echo "AZ_NOT_LOGGED_IN"
```

- If logged in: show tenant, ask user to confirm
- If not logged in: ask user for tenant ID manually

### Step 3: Entra Role Verification

Explain the required role (`{{REQUIRED_ENTRA_ROLE}}`). If Azure CLI is logged in, verify via:

```bash
az rest --method GET --url "https://graph.microsoft.com/v1.0/me/memberOf/microsoft.graph.directoryRole?\$select=displayName" --output json
```

Warn but don't block if role not found (user may have equivalent permissions).

### Step 4: App Registration Check

Explain required Graph permissions: {{REQUIRED_GRAPH_PERMISSIONS}}

Guide user through creating an app registration if none exists.

### Command Generation (NOT Execution)

Because scripts require interactive browser authentication (device code flow via `Connect-MgGraph`), the wizard **must not attempt to execute the script**. Instead:

1. Collect all parameters
2. Generate any input files (e.g., `{{INPUT_JSON_FILE}}`)
3. Display the exact command the user should run in their terminal:

```bash
cd <repo-root>
pwsh -File ./{{TARGET_SCRIPT}} -TenantId "<tenant>" -ClientId "<client-id>"
```

### Key Rules

- **Never skip steps** — complete each before moving to the next
- **Do NOT execute scripts that need browser auth** — provide the command instead
- **Validate all inputs** — check GUID format, HTTPS URLs, non-empty strings
- **Use `ask_user`** for collecting field values
- **Do not store secrets in files** — client secrets go on the command line only
