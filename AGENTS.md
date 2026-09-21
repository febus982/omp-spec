# AGENTS.md

## Repository Role

This repository is the canonical source and OMP marketplace for the portable `feature-development` plugin. It contains workflow instructions and an OMP planner agent; it contains no application implementation.

## Structure

```text
.omp-plugin/marketplace.json
plugins/feature-development/
├── skills/feature-development/SKILL.md
├── agents/implementation-planner.md
└── README.md
```

The marketplace catalog name is `omp-spec`. The plugin name is `feature-development`.

## Portability Boundary

Keep plugin content project-agnostic.

The plugin MAY define:

- brainstorming, approval, planning, implementation, verification, and archival behavior;
- OpenSpec CLI and artifact contracts;
- OMP planner-agent behavior;
- generic integration points for external requirements and task trackers.

The plugin MUST NOT embed:

- project names or repository paths;
- organization-specific Notion, ClickUp, Jira, or other tracker identifiers;
- repository-specific build, test, branch, or deployment commands;
- product requirements or architecture decisions;
- credentials, MCP endpoints, or user-specific model selectors.

Those belong in the consuming project's `AGENTS.md` or equivalent context.

## Runtime Contract

The plugin ships two inseparable capabilities:

1. `skills/feature-development/SKILL.md` — owns the end-to-end state machine and approval gates.
2. `agents/implementation-planner.md` — turns approved OpenSpec artifacts into grounded file-level plans using OMP's `@plan` role.

Do not publish one without the other. If either name changes, migrate every reference in the skill, agent, catalog, and README in the same change. Use a clean cutover; do not retain aliases.

The workflow drives the OpenSpec CLI directly. It MUST NOT depend on generated `openspec-*` skills being discoverable across repositories. It SHOULD obtain artifact order, templates, and rules from `openspec status --json` and `openspec instructions --json` rather than duplicating a schema.

Preserve these safety properties unless a breaking release explicitly changes them:

- explicit design approval before durable change artifacts;
- explicit implementation-plan approval before product code;
- no product or architecture decisions invented by the agent;
- source fixes rather than symptom suppression;
- runtime verification in addition to structural validation;
- no archive while acceptance criteria fail;
- gradual brownfield spec backfilling rather than bulk speculative documentation.

## Documentation

The root `README.md` is the user-facing installation and operation guide. Update it in the same change whenever prerequisites, install commands, profile guidance, trigger syntax, approval gates, versioning, or upgrade behavior changes.

`plugins/feature-development/README.md` is the concise installed-plugin description. Keep it aligned with the root README without duplicating the complete maintenance guide.

Do not claim support for an OMP or OpenSpec version that has not been exercised.

## Versioning

`.omp-plugin/marketplace.json` contains both marketplace metadata and the installable plugin version.

Increment the plugin version for every published behavior or content change that installed users must receive:

- patch: compatible correction or clarification;
- minor: backward-compatible capability;
- major: changed approval gates, prerequisites, artifact semantics, or planner contract.

Update `metadata.version` when the catalog itself changes. Create Git tags matching releases after verification.

Do not add a license or change licensing terms without an explicit maintainer decision.

## Verification

There is no build or application test suite. Before committing a plugin change:

1. Parse `.omp-plugin/marketplace.json` as JSON.
2. Confirm the catalog source resolves inside the repository.
3. Confirm skill frontmatter has `name` and `description`, and the directory name matches `name`.
4. Confirm planner frontmatter has `name`, `description`, `model: "@plan"`, and `blocking: true`.
5. Search plugin content for accidental project-specific identifiers.
6. Add the checkout as a local marketplace and force-install it project-scoped in a throwaway or suitable test repository.
7. Restart OMP and confirm both the skill and planner are discovered.
8. Smoke-test natural-language and explicit skill activation without modifying product code.
9. For workflow behavior changes, exercise an initialized throwaway OpenSpec repository through the affected phase.

Commit and push releases from this repository's current `main` branch.
