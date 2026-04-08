# Spec-Driven Development FAQ

## General

### What is a spec?

A spec is a parameterized Markdown file that captures a reusable pattern — like grounding rules, wizard agent flows, or documentation conventions. It uses `{{VARIABLE_NAME}}` placeholders so the same pattern can be applied to different projects with different values.

### Where do the specs live?

Specs live in their own repository (`arbitrated-grounding-specs`). Each project that uses specs imports only the ones it needs. The spec repo also contains the three meta-agents (`@spec-exporter`, `@spec-importer`, `@spec-drift`).

### Will the spec repo be documented in this project?

Yes. The `docs/spec-driven-development/` folder in this project documents the spec format, available specs, and how they relate to this project's patterns. This serves as both the documentation and the reference implementation since this project (`agent365-management`) is where the patterns were originally developed.

## Export and Sync

### Does `@spec-exporter` push to the spec repo?

No. The `@spec-exporter` writes spec files to a **local folder** (default: `specs/` in the current project). The user is responsible for reviewing the generated specs and committing/pushing them to the master spec repo. This is a deliberate safety boundary — the agent never pushes to a remote repository.

**Typical workflow:**
```
@spec-exporter → writes to local specs/ folder
     ↓
User reviews the generated specs
     ↓
User copies specs to the spec repo, commits, and pushes
```

### How do I update a spec after improving a pattern?

1. Make the improvement in your project (e.g., update a wizard agent's flow)
2. Run `@spec-exporter` to re-extract the updated pattern
3. Review the diff against the existing spec
4. Commit the updated spec to the spec repo with a version bump
5. Other projects can then import the new version via `@spec-importer`

### What if the spec repo and my project get out of sync?

Run `@spec-drift` — it compares your project's current files against the specs in your `.spec-config.yaml` and reports:
- Which patterns have diverged
- What's different (with diffs)
- Whether the divergence is intentional (project-specific override) or accidental (drift)

## Applying Spec Updates to Your Project

### How do I know when specs have been updated?

Run `@spec-drift` — it compares your project's `spec_version` against the latest manifest and reports any version gap. You can also check the spec repo's commit history or releases.

### How do I apply a spec update?

Re-run `@spec-importer`. When it detects an existing `.spec-config.yaml`, it enters **re-import mode** and walks you through every change:

```
@spec-importer Re-import specs
```

The importer handles all types of spec changes automatically — see below.

### What types of spec changes can happen, and how are they handled?

| Change Type | How You'll Know | What the Importer Does |
|---|---|---|
| **New variable** | Importer lists new variables with descriptions | Prompts you for a value (shows example and default) |
| **Changed variable default/description** | Importer shows the before/after | Informational — your existing value is preserved |
| **Removed variable** | Importer flags it as deprecated | Asks whether to remove from `.spec-config.yaml` |
| **New dependency** (`requires`) | Importer checks if the required spec is imported | Auto-adds if missing, or asks you to confirm |
| **New section in generated content** | Importer regenerates files and shows diffs | You review and accept the updated file |
| **Changed section content** | Importer shows diff between old and new | You review and accept or keep your version |
| **New artifact/file** | Importer detects referenced files that don't exist | Offers to create a placeholder |
| **Removed section** | `@spec-drift` flags orphan content in your files | You decide whether to remove the section |
| **Version bump** | Importer shows semver impact (major/minor/patch) | Updates `spec_version` in `.spec-config.yaml` |

### What do the version numbers mean?

Specs use [semantic versioning](https://semver.org/):

- **Patch** (2.0.0 → 2.0.1): Bug fixes, typo corrections. Safe to re-import without review.
- **Minor** (2.0.0 → 2.1.0): Additive changes — new variables, new sections, new specs. Non-breaking. Existing behavior is unchanged; you just get new capabilities.
- **Major** (2.0.0 → 3.0.0): Breaking changes — removed variables, restructured sections, changed semantics. Review the changelog before re-importing.

### What's the full update workflow?

```
1. Detect   →  @spec-drift          (shows version gap + file diffs)
2. Review   →  Read the changelog    (understand what changed)
3. Apply    →  @spec-importer        (re-import handles all change types)
4. Verify   →  @spec-drift           (confirm everything is clean)
```

### What if I want to skip some changes?

During re-import, the importer asks for confirmation at each step. You can:
- Skip new variables (they'll use their default value)
- Decline to scaffold new artifacts
- Keep your existing file versions instead of accepting regenerated ones
- Add items to the `overrides` section of `.spec-config.yaml` to permanently suppress drift warnings for intentional divergences

### What if a spec update conflicts with my project-specific changes?

The importer shows a diff and asks whether to overwrite or keep your version. If you keep yours, `@spec-drift` will flag it as drift on future checks. To silence the warning, add an override to `.spec-config.yaml`:

```yaml
overrides:
  grounding-rules:
    - section: "Source Hierarchy"
      reason: "Project uses a 3-tier hierarchy without corrections layer"
```

## Implementing in Other Projects

### How do I apply specs to a new project?

Three steps:

1. **Copy the meta-agents** into your project:
   ```bash
   cp arbitrated-grounding-specs/.github/agents/Spec-Importer.agent.md \
      your-project/.github/agents/
   ```

2. **Create a `.spec-config.yaml`** in your project root listing which specs to import and your project-specific variable values:
   ```yaml
   spec_repo: paulwu/arbitrated-grounding-specs
   spec_version: "1.0.0"
   imports:
     - grounding-rules
     - research-agent
   variables:
     PRIMARY_SOURCE_URL: "https://learn.microsoft.com/en-us/azure/well-architected/"
     CACHED_BASELINE_FILE: "notes/WAF-docs.md"
   ```

3. **Run `@spec-importer`** in Copilot Chat (interactive mode):
   ```
   @spec-importer Import specs from ~/arbitrated-grounding-specs/specs/ using .spec-config.yaml
   ```
   The importer generates `.github/copilot-instructions.md`, agent files, README sections, and a lightweight `docs/spec-driven-development.md` guide with a link back to the full documentation.

### What if I only want some specs?

Specs are composable — you can import any subset. Just list the ones you want in your `.spec-config.yaml` under `imports`. The importer will warn if a spec has a `requires` dependency you haven't imported, but it won't block you.

### Can I override parts of a spec for my project?

Yes. After importing, you can edit the generated files. The `@spec-drift` agent will flag these as intentional divergences. To suppress the warning, add an override section to your `.spec-config.yaml`:

```yaml
overrides:
  grounding-rules:
    - section: "Contradiction Detection"
      reason: "Project uses a simplified format without priority numbers"
```

### What about brand-new projects?

The fastest path:

```bash
# Use the spec repo as a GitHub template
gh repo create my-new-project --template paulwu/arbitrated-grounding-specs
cd my-new-project

# Edit .spec-config.yaml with your project-specific values
# Then run the importer
@spec-importer Scaffold this project from specs
```

### Do I need all three meta-agents in every project?

No. Most projects only need `@spec-importer` and `@spec-drift`. The importer automatically syncs both of these from the spec repo on every import, so you only need to copy them manually once. The `@spec-exporter` is only copied if it already exists in your project — it's meant for projects where new patterns are being developed.

| Agent | Who needs it | Auto-synced? |
|---|---|---|
| `@spec-importer` | Every project that uses specs | ✅ Yes |
| `@spec-drift` | Projects that want ongoing sync checking | ✅ Yes |
| `@spec-exporter` | Only the project(s) where patterns are developed | Only if already present |

## Technical Details

### What files does the importer generate?

Depending on which specs are imported:

| Spec | Files Generated/Updated |
|---|---|
| `grounding-rules` | `.github/copilot-instructions.md` (canonical sources section) |
| `research-conventions` | `.github/agents/Entra-Curator.agent.md` |
| `research-agent` | `.github/agents/<ResearchAgent>.agent.md` |
| `wizard-agent` | `.github/agents/<WizardAgent>.agent.md` (template) |
| `doc-architecture` | `.github/copilot-instructions.md` (architecture section), `notes/`, `docs/` folders |
| `readme-structure` | `README.md` (TOC, agent table, collapsible structure) |
| *(always)* | `docs/spec-driven-development.md` (lightweight framework guide with link to full docs) |
| *(always)* | `.github/agents/Spec-Importer.agent.md`, `Spec-Drift.agent.md` (synced from spec repo) |

### What's the variable syntax?

Mustache-style double braces: `{{VARIABLE_NAME}}`. Variables are defined in the spec's YAML frontmatter and filled from the project's `.spec-config.yaml`.

### How is versioning handled?

Specs use semantic versioning (semver). Projects pin to a version in `.spec-config.yaml`. When a new spec version is available, `@spec-drift` shows the diff between your current version and the latest.
