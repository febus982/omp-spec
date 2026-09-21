---
name: feature-development
description: Orchestrate feature work from natural-language brainstorming through approved design, OpenSpec artifacts, OMP planning and implementation, verification, and archival without manual phase commands.
compatibility: Requires Oh My Pi with task agents and the OpenSpec CLI.
metadata:
  version: "1.0.0"
---

# Feature Development

Own the transitions for substantial feature work. The user should not need to invoke a sequence of `/opsx:*` commands or choose the next workflow skill.

Use this skill when the user asks to brainstorm, design, build, or implement a feature or behavior change. Do not use it for a narrow bug fix with known intended behavior; use systematic debugging when available.

## Outcome

Move one approved change through this state machine:

```text
intent
  → brainstorm
  → approved design
  → planning-system links
  → OpenSpec proposal/specs/design/tasks
  → OMP implementation plan
  → approved plan
  → OMP implementation
  → OpenSpec and runtime verification
  → archive
```

Automatic progression means no manual slash-command handoffs. It does not remove the two human approval gates: approved design and approved implementation plan.

## Prerequisites

Before creating OpenSpec artifacts:

1. Resolve every affected Git repository.
2. Read that repository's `AGENTS.md` or equivalent loaded context.
3. Confirm `openspec` is available.
4. Confirm each affected repository contains `openspec/config.yaml`.
5. Confirm the OMP agent `implementation-planner` is discoverable.

OpenSpec installation is a runtime prerequisite, not an authoring prerequisite for this skill. If it is missing, preserve the approved design, report the exact missing prerequisite, and stop before fabricating OpenSpec output.

For an uninitialized repository, the standard OMP target is:

```bash
openspec init --tools oh-my-pi
```

OpenSpec initialization may generate harness-specific workflows under `.omp/`, but this skill drives the CLI directly and does not require those generated skills to be discoverable.

## Sources of truth

Repository context defines authoritative product documentation, planning systems, task trackers, and release policy. Follow it before this generic workflow.

- OpenSpec is the repository-local, versioned contract for one code change. It must refine rather than silently replace any authoritative external requirement.
- `tasks.md` is the execution checklist for an OpenSpec change. It does not replace an authoritative external task when the project uses one.
- Put stable external planning links in the proposal when they exist.
- Load any project-mandated planning skill before reading or mutating external planning records.
- For a cross-repository feature, default to one OpenSpec change per affected repository with the same change slug and shared planning links.
- Do not adopt OpenSpec Stores or another cross-repository specification architecture unless repository context or an explicit user decision selects it.

## Resume rule

Inspect existing artifacts and resume at the earliest incomplete phase. Never restart brainstorming or overwrite an approved artifact merely because a new session began.

Treat these as phase evidence:

- approved design: explicit approval in conversation plus the durable design/requirement record required by project context;
- OpenSpec planning complete: proposal, delta specs, optional design, and tasks exist and validate;
- implementation plan complete: `openspec/changes/<change>/plan.md` exists and matches current artifacts;
- implementation complete: all applicable `tasks.md` checkboxes are checked and changed behavior has been exercised;
- verification complete: OpenSpec verification passes and the changed runtime surface was exercised;
- archive complete: the change is absent from active changes and present in the archive.

If an upstream artifact changes, invalidate downstream plan and verification evidence. Regenerate only what became stale.

## Phase 1: Brainstorm

Use the disciplined parts of Superpowers brainstorming, but this skill owns the handoff; do not invoke `writing-plans`.

1. Classify the work:
   - **spike**: feasibility investigation, no production code;
   - **bounded**: small change to an existing, readable flow;
   - **architectural**: new subsystem, cross-repository behavior, or public/interface change.
2. Inspect relevant project context before asking questions.
3. State the classification so the user can correct it.
4. Establish intended outcome, users, constraints, non-goals, and observable success.
5. Ask only missing questions, one focused question at a time.
6. For architectural work, offer 2–3 approaches with concrete tradeoffs and a recommendation.
7. Present the design at the appropriate depth: short in chat for bounded work; sectioned and durable for architectural work.
8. Stop for explicit design approval. Approval covers only the presented design.

A spike ends with findings and a recommendation. It does not enter OpenSpec or implementation automatically.

If hidden complexity turns bounded work into architectural work, stop, reclassify, and complete the architectural design gate.

## Phase 2: Durable planning and OpenSpec

After design approval, continue without asking the user to invoke another command.

1. Apply repository planning rules. Create or update authoritative external records through any project-mandated planning skill before deriving OpenSpec artifacts from them.
2. Choose one kebab-case change slug. Reuse it across affected repositories.
3. For each repository, run `openspec context --json` with that repository as the working directory. Stop if it does not resolve that repository's OpenSpec root.
4. Create or resume the change with the OpenSpec CLI. For a new change, run `openspec new change "<slug>"`.
5. Build artifacts from OpenSpec's live schema rather than hardcoding a generated workflow:
   - run `openspec status --change "<slug>" --json`;
   - compute the required artifact closure from `applyRequires` and each artifact's `requires` edges;
   - for every ready artifact, run `openspec instructions <artifact-id> --change "<slug>" --json`;
   - read completed dependency files, apply the returned `context`, `rules`, `template`, and `instruction`, then write to `resolvedOutputPath`;
   - repeat status and instruction lookup until every required artifact is done or schema-skipped.
6. Ground artifacts in the approved design and authoritative planning links. The artifact set normally contains:
   - `proposal.md`: motivation, scope, capabilities, impact, and external planning links;
   - `specs/<capability>/spec.md`: normative requirements and observable scenarios;
   - `design.md` only when the schema requires it or non-trivial technical decisions need durable rationale;
   - `tasks.md`: coarse, ordered, checkable implementation work.
7. Run `openspec validate "<slug>" --type change --strict --json` from the repository root. Fix artifact defects before planning implementation.

Do not introduce new product or architecture decisions while extracting artifacts. Return to the design gate if a material unresolved decision appears.

## Phase 3: OMP implementation plan

Dispatch the blocking `implementation-planner` OMP agent through `task`. Provide:

- affected repositories and their local instructions;
- exact OpenSpec change directories;
- authoritative planning links;
- approved scope and exclusions;
- the required output path `openspec/changes/<change>/plan.md` in each repository.

The planner must inspect real code, reuse repository conventions, identify exact files and symbols, map call sites, define dependency order, separate genuinely parallel slices, and specify runtime verification. It must not implement.

Review the generated plan against proposal, specs, design, and tasks. Correct contradictions before presenting it. Then stop for explicit plan approval. The user may approve, request changes, or cancel implementation.

Do not turn design approval into plan approval. Do not write product code before the plan gate.

## Phase 4: Implement with OMP

After plan approval, continue without an OpenSpec apply command. OMP owns execution.

1. Convert the approved plan into the session todo list, preserving every plan item.
2. Read each target repository's local instructions again before its first edit.
3. Resolve prerequisites in the parent session.
4. Dispatch task agents only for genuinely independent slices with explicit file ownership and shared contracts. Keep dependent or same-file work serialized.
5. Use repository-native isolation when appropriate. Each Git repository remains an independent root.
6. Implement the approved scope only. Do not add speculative validation, retries, telemetry, abstractions, or unrelated refactors.
7. Update `tasks.md` checkboxes only after the corresponding observable behavior is complete.
8. If a bug, test failure, or unexpected behavior appears, load `skill://systematic-debugging` when available; otherwise apply the same evidence-first root-cause process. Never guess through repeated fixes.
9. Run one integrated validation pass after parallel work converges.

OpenSpec artifacts are not proof. Exercise the changed runtime surface as required by repository context.

## Phase 5: Verify and converge

Run `openspec validate "<slug>" --type change --strict --json` from every affected repository, then perform the implementation-to-artifact checks below. The generated `openspec-verify-change` skill may be used as additional guidance when it is discoverable, but this workflow must not depend on cross-repository skill discovery.

Verification must cover:

- OpenSpec structural validity;
- every normative scenario in the delta specs;
- every applicable `tasks.md` item;
- design/spec consistency;
- affected call sites and repository boundaries;
- the actual runtime surface or a focused smoke scenario;
- repository-specific tests or checks that exercise the change.

On failure:

1. classify the failure as implementation, artifact, environment, or external dependency;
2. fix the source, not the symptom;
3. invalidate stale evidence;
4. rerun the narrow failing proof;
5. rerun integrated verification.

Stop and report after three failed hypotheses that point to an architectural problem. Do not silently weaken a spec or test to obtain a pass.

## Phase 6: Sync trackers and archive

Proceed only after verification passes.

1. Update authoritative external task status through any project-mandated planning integration.
2. Run `openspec archive "<slug>" --yes --json` from each affected repository after confirming its delta specs should update that repository's living code-level specs. Use `--skip-specs` only when the approved change has no behavioral specs.
3. Confirm delta specs were merged into living code-level specs and the change moved to the archive.
4. Follow repository rules for independent commits and branch handling.
5. Report exact artifacts, repositories, verification evidence, tracker updates, and archive result.

Do not archive blocked, partially implemented, or failing work. Warnings that affect acceptance criteria block archive; informational warnings remain recorded.

## Interaction rules

- Continue automatically between phases until an approval gate or real external blocker.
- Never stop merely to announce a phase transition.
- Never ask the user to invoke a slash command that the agent can replace by loading the corresponding skill.
- Ask one focused question at a time during brainstorming.
- Use existing repository and planning data before asking for information.
- Preserve the user's decisions verbatim; label assumptions and resolve them before they become requirements.
- Keep OpenSpec tasks coarse and the OMP plan file-level. Do not duplicate the same granularity in both.
- Do not claim success from generated files, passing unit tests alone, or unchecked task boxes.
