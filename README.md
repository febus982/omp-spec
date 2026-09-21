# OMP Spec

An Oh My Pi marketplace containing a portable, spec-driven feature-development workflow.

The `feature-development` plugin combines:

- structured brainstorming and explicit design approval;
- OpenSpec change proposals and delta specs;
- an OMP planner agent routed through `@plan`;
- OMP-native implementation and task delegation;
- runtime and specification verification;
- OpenSpec archival and gradual brownfield spec backfilling.

Project-specific planning systems, task trackers, repository maps, and release rules stay in each consuming project's `AGENTS.md` or equivalent context.

## Prerequisites

### Oh My Pi

Use an OMP version with marketplace plugins, Agent Skills, and custom task agents. The plugin installs both:

```text
skills/feature-development/SKILL.md
agents/implementation-planner.md
```

The planner uses OMP's `@plan` model role. Configure that role in `~/.omp/agent/config.yml` if it is not already mapped:

```yaml
modelRoles:
  plan: <provider>/<model>:high
```

Replace the selector with a model available in your OMP installation.

### OpenSpec CLI

The workflow requires the OpenSpec CLI. It has been tested with OpenSpec 1.13.1.

Install the current CLI and confirm it is available:

```bash
npm install -g @fission-ai/openspec@latest
openspec --version
```

OpenSpec currently requires Node.js 20.19.0 or newer when installed through npm.

### OpenSpec project initialization

Initialize every implementation repository that should own code-level specifications:

```bash
cd /path/to/project
openspec init --tools oh-my-pi
```

Commit the generated `openspec/` and `.omp/` files with the project.

The feature-development skill drives the OpenSpec CLI directly, so it does not require a particular OpenSpec workflow profile. The default `core` profile is sufficient.

For convenient manual access, the recommended profile is the core workflows plus `verify`:

```text
explore, propose, apply, update, sync, archive, verify
```

Configure it interactively:

```bash
openspec config profile
```

Select the six core workflows and `Verify change`, then apply the profile in each initialized repository:

```bash
openspec update
```

The optional `new`, `continue`, `ff`, `bulk-archive`, and `onboard` workflows are not required by this plugin.

## Install from the OMP marketplace

Add this GitHub repository as a marketplace:

```text
/marketplace add febus982/omp-spec
```

Install the plugin for the current user:

```text
/marketplace install feature-development@omp-spec
```

Or install it only for the current project:

```text
/marketplace install --scope project feature-development@omp-spec
```

CLI equivalents:

```bash
omp plugin marketplace add febus982/omp-spec
omp plugin install feature-development@omp-spec
```

For a project-scoped installation:

```bash
omp plugin install --scope project feature-development@omp-spec
```

Restart OMP after installation so both the skill and planner agent are discovered.

## Configure the consuming project

The plugin is intentionally project-agnostic. Put project policy in `AGENTS.md` or equivalent context, including:

- authoritative requirements and design records;
- task tracker and status rules;
- repository map and affected-repository tags;
- branch, commit, review, and deployment rules;
- whether cross-repository work uses separate OpenSpec changes or an OpenSpec Store.

OpenSpec remains the code-level behavioral contract. It should refine rather than silently replace an authoritative product requirement or external task.

## Trigger the workflow

Natural-language activation:

```text
Let's brainstorm organization support.
```

With an existing task:

```text
Let's brainstorm the feature described in <task URL>. Read the live task and linked requirements first.
```

Deterministic activation:

```text
/skill:feature-development Let's brainstorm organization support.
```

The workflow has two explicit approval gates:

1. design approval;
2. implementation-plan approval.

Between those gates it advances automatically without requiring a sequence of `/opsx-*` commands.

## Workflow

```text
intent
  → brainstorm
  → approved design
  → project planning records
  → OpenSpec proposal/specs/design/tasks
  → OMP implementation plan
  → approved plan
  → OMP implementation
  → OpenSpec and runtime verification
  → archive
```

The workflow uses OpenSpec's live schema graph through `openspec status` and `openspec instructions`. It therefore supports custom OpenSpec schemas without hardcoding the default artifact order.

For existing codebases, only behavior touched by a change is specified. Archiving merges that delta into `openspec/specs/`; unrelated legacy behavior remains undocumented until future work touches it.

## Update

Refresh the marketplace catalog and upgrade the installed plugin:

```text
/marketplace update omp-spec
/marketplace upgrade feature-development@omp-spec
```

CLI equivalents:

```bash
omp plugin marketplace update omp-spec
omp plugin upgrade feature-development@omp-spec
```

Restart OMP after upgrading.

## Local development

Add a local checkout as a marketplace:

```text
/marketplace add /absolute/path/to/omp-spec
/marketplace install --force --scope project feature-development@omp-spec
```

Restart OMP, then verify:

- `feature-development` is a discovered skill;
- `implementation-planner` is an available task agent;
- the planner resolves the configured `@plan` role;
- an initialized throwaway repository can create and strictly validate an OpenSpec change;
- no product code is written before both approval gates.

## Versioning

The marketplace and plugin versions are declared in `.omp-plugin/marketplace.json`. Increment the plugin version whenever users need `omp plugin upgrade` to install changed skill or agent content.

Use semantic versioning:

- patch: wording or compatibility fixes without changing approval or artifact contracts;
- minor: backward-compatible workflow behavior or new optional capability;
- major: changed approval gates, prerequisites, artifact semantics, or planner contract.

## License

MIT. You may use, copy, modify, merge, publish, distribute, sublicense, and sell the software, provided the copyright and license notice remain with copies or substantial portions.

The software is provided “as is,” without warranty. The authors and copyright holders are not liable for claims or damages arising from its use. See [`LICENSE`](LICENSE) for the complete terms.
