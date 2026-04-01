# Copilot Instructions — arbitrated-grounding-specs

## Architecture

This is a **spec-driven development framework** — a collection of parameterized Copilot agent patterns designed to be imported into any repository. There is no application code, build system, or tests.

### Core concepts

```
specs/*.spec.md          Reusable, parameterized patterns (the "library")
.github/agents/*.md      Three meta-agents that operate on specs
docs/spec-driven-development/   Framework documentation
```

**Specs** are Markdown files with YAML frontmatter that define Copilot agent behaviors and project conventions. They use `{{VARIABLE}}` placeholders so a single spec can adapt to different projects.

**Meta-agents** form the operational layer:

| Agent | Role |
|---|---|
| `@spec-importer` | Applies specs to a target project — collects variable values interactively, generates project files |
| `@spec-drift` | Read-only comparison of a project's current state against its imported specs |
| `@spec-exporter` | Extracts new patterns from a project into parameterized spec files |

**Import flow:** spec repo → `@spec-importer` → target project gets `.github/copilot-instructions.md`, `.github/agents/*.agent.md`, `README.md` scaffold, `.spec-config.yaml`, and `docs/spec-driven-development.md`.

### Key file relationships

- `specs/manifest.yaml` — index of all available specs with versions
- `.spec-config.yaml` (in importing projects) — records which specs were imported and the variable values used; supports an `overrides` section for intentional divergences
- Each spec's `requires` field declares dependencies on other specs

## Conventions

### Spec file format

- **Naming:** `<spec-id>.spec.md` using kebab-case (e.g., `grounding-rules.spec.md`)
- **Frontmatter fields:** `spec`, `version`, `description`, `extracted_from`, `requires`, `variables`
- **Variables:** `{{UPPER_SNAKE_CASE}}` — defined in frontmatter with `name`, `description`, `required`, `default`, and `example`
- **Versioning:** Semantic versioning (patch = fixes, minor = backward-compatible additions, major = breaking changes)
- **Internal structure:** YAML frontmatter → heading → pattern description → rules → templates/examples

### Agent file format

- **Naming:** `PascalCase.agent.md` (e.g., `Spec-Importer.agent.md`)
- **Location:** `.github/agents/` in both this repo and importing projects
- Meta-agents (`Spec-Importer`, `Spec-Drift`) are auto-synced to importing projects on every import

### Variable deduplication

When multiple specs share a variable (e.g., `KNOWLEDGE_FOLDER` in both `grounding-rules` and `research-conventions`), the importer collects the value once and reuses it across all specs.
