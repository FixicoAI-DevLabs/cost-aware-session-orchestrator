# Cost-Aware Independent Session Orchestrator v1.2

Human-readable, annotated companion to `portable-session-orchestrator-installer.v1.2.json`.

Payload version: 1.2.0.

This document explains what the installer declares, why each mechanism exists, and how the current Claude Code adapter maps a provider-neutral orchestration protocol onto the host application. It is documentation, not an installer. The JSON file is the machine-readable package.

## Installation footprint

The installer creates one manual-only skill and one receipt in that skill's directory. Here, `~` means the recipient's home directory: for example, `C:\Users\Name` on Windows or `/home/name` on Linux.

| Action | Destination | Purpose |
|---|---|---|
| Create | `~/.claude/skills/cost-aware-independent-session-orchestrator/SKILL.md` | Add the manually invoked orchestration policy. |
| Create last | `~/.claude/skills/cost-aware-independent-session-orchestrator/INSTALL-RECEIPT.json` | Record the exact managed file, prior state, backup, version, and hashes for verification and safe uninstallation. |

That is the complete persistent footprint. The installer does **not** read or change `settings.json`; install global agent definitions; read or change memories, hooks, workflows, teams, tasks, or unrelated skill bodies; or install a plugin, executable, daemon, model, scheduled job, or background service. It does not change the lead session's selected model or effort, and it does not create a live team.

When the workflow is later used, Claude Code—not this installer—creates session-scoped team and task state beneath:

```text
~/.claude/teams/<team-name>/
~/.claude/tasks/<team-name>/
```

The source installer JSON remains wherever the recipient placed it. Those session-state paths are created and managed by Claude Code only after a separately authorized live orchestration run. On current versions, the team configuration is removed when the session ends, while task-list state can persist locally under the host's retention policy.

## What behavior this adds

The package creates a reusable operating pattern with five defining properties:

1. One capable lead session owns decomposition, routing, monitoring, arbitration, and final synthesis.
2. Bounded assignments run in independent worker sessions with separate context windows.
3. Workers use economical resource profiles instead of automatically inheriting the lead's expensive configuration.
4. The lead reacts to worker completion or attention events and escalates only the branch that needs more capability.
5. The lead does not deliver the orchestrated result until every required assignment has reached a terminal state and been dispositioned.

The cost policy is not based on the idea that parallel sessions magically use fewer raw tokens. They often use more. Current official cost guidance warns that agent teams can use approximately seven times the tokens of standard sessions in plan-mode configurations. The intended savings come from allocating cheaper models to most worker work, minimizing repeated context, imposing supervisory turn and retry budgets, and reserving the capable lead for decisions that benefit from it.

## Version lineage

Version 1.2 preserves the orchestration state machine and economic control plane while changing installation to a safe default. It removes persistent settings mutation and reusable global worker definitions, requires manual skill invocation, completes collision detection before any mutation, narrows verification, and makes uninstall strictly receipt-governed.

The machine-readable lineage record identifies:

- predecessor file: `portable-session-orchestrator-installer.v1.1.json`
- predecessor version: `1.1.2`
- predecessor SHA-256: `D091998DD80EA7AD04B241ED77A5E129C9D87187984DEED69058D39683A01769`

The predecessor remains recoverable through repository history. Version 1.2 does not automatically remove, migrate, or adopt a version 1.1 installation because doing so would require inspecting and mutating broader user configuration. A recipient who has version 1.1 installed receives a migration warning and retains control of the cleanup decision.

## The two layers

The JSON deliberately separates the mechanic from the host implementation:

```text
portable_core
    eligibility
    resource policy
    assignment contract
    execution and monitoring
    escalation
    completion barrier
    safety

adapters
    claude_code
        session-scoped activation
        model mappings
        manual-only installed skill
        protected surfaces and collision checks
        install / verify / uninstall procedures
```

The `portable_core` contains no provider-specific model names, feature flags, paths, or commands. A different application can implement the same protocol by supplying a new adapter without modifying the core.

## Activation contract

The receiving assistant recognizes exactly three commands when the user explicitly refers to the JSON file:

| Command | Meaning | Mutation allowed |
|---|---|---|
| `INSTALL` | Complete preflight and collision checks, then write the manual skill and receipt | Only the declared skill and receipt |
| `VERIFY` | Check the declared skill, receipt, hashes, and compatibility | None |
| `UNINSTALL` | Remove or restore only content proven by the receipt | Only proven installed content |

Merely opening the JSON does nothing. JSON is data, not executable code. The receiving assistant must interpret it after the explicit command.

Before `INSTALL` mutates anything, it must show:

- resolved destination paths;
- files that will be created or changed;
- backup behavior;
- compatibility warnings;
- any collision with existing user content.

The command authorizes only this installation. It does not authorize unrelated configuration changes, external communication, credential use, publication, deployment, or destructive work.

### Idempotency

If exactly the same version and content are already installed, `INSTALL` reports success without rewriting files.

If a destination exists with different content, or another skill declares the same frontmatter name, the installer stops before creating directories, backups, temporary files, or a receipt. It preserves the collision, shows only the conflicting path and a focused diff of the exact target when applicable, and asks for a fresh decision. Existing user work is never assumed to be disposable.

## Eligibility gate

Independent sessions are useful only when the job partitions cleanly. The installer therefore makes orchestration conditional rather than automatic.

All of these should be true:

- There are at least two bounded assignments.
- Assignments have little or no write overlap.
- Independent context windows improve elapsed time, coverage, verification, or variance reduction.
- The host exposes independent sessions, a task or team registry, and completion or attention signals.
- Each worker can receive a precise deliverable, evidence requirement, scope boundary, and stop condition.
- The expected benefit exceeds coordination and compute cost.

The lead rejects orchestration when any of these apply:

- One coherent pass is enough.
- Workers would contend over the same mutable state.
- Required permissions or external effects have not been authorized.
- The host cannot supply genuine independent sessions.
- Worker terminal states cannot be observed and reconciled.
- The only rationale is to make the work look more thorough.

This gate directly addresses the common failure mode where parallelism becomes an expensive ritual.

## State model

Every assignment moves through a visible lifecycle:

```text
proposed -> confirmed -> dispatched -> running
                                  |        |
                                  |        +-> needs_attention
                                  |        |
                                  +------> waiting

running / waiting / needs_attention
        -> completed | failed | cancelled
        -> integrated
```

`needs_attention` is intentionally non-terminal. A lead cannot treat a worker asking for permission, input, or conflict resolution as finished.

`integrated` is a lead-side bookkeeping state. It means the worker's terminal report has been evaluated and dispositioned, not merely received.

## Resource policy

### Objective

The goal is to minimize cost-equivalent compute and duplicated context while satisfying explicit quality, coverage, latency, and evidence requirements.

This is more precise than “use fewer tokens.” Two inexpensive workers may consume more tokens but cost less than one long run on a premium model. Conversely, careless parallelism can be both more expensive and worse because workers repeat the same context and investigation.

The lead must not claim savings unless the host exposes adequate telemetry or the estimate uses a declared rate source and transparent assumptions.

### Lead profile

The lead uses the user's chosen capable model and reasoning effort. The installer does not change the current session.

The lead is responsible for:

- decomposition and task routing;
- assignment-ledger maintenance;
- monitoring and blocker resolution;
- challenging unsupported conclusions;
- reconciling conflicting reports;
- final synthesis.

The lead should avoid:

- reproducing each worker's full investigation;
- doing bulk extraction that a light worker can do reliably;
- assigning every worker the lead's model merely for convenience.

The lead is expensive where judgment is valuable, economical where repetition is not.

### Worker profiles

The portable core defines capabilities rather than model brands.

| Profile | Appropriate work | Default turn cap | Default write posture |
|---|---|---:|---|
| Light | Discovery, inventory, extraction, status, narrow comparison | 8-turn supervisory budget | Read-only |
| Standard | Bounded implementation, focused review, tests, source-backed analysis, reproduction | 12-turn supervisory budget | Explicitly owned scope only |
| Deep | Difficult bounded reasoning, adversarial review, ambiguous diagnosis, high-risk verification | 16-turn supervisory budget | Explicitly owned scope only |

The routing rule is simple: choose the cheapest profile likely to satisfy the assignment's acceptance criteria.

The deep profile is not the normal worker setting. It is a targeted escalation or a deliberate choice for a genuinely difficult branch.

Maximum-effort reasoning is also not a default. An unconstrained reasoning budget can erase the cost advantage that the worker topology is supposed to create.

### Context economy

Each worker receives a context capsule rather than the lead's whole conversation by default. A capsule contains only what the assignment needs:

- relevant facts and decisions;
- exact paths or interfaces;
- constraints and boundaries;
- evidence already collected;
- acceptance criteria;
- required output schema.

When multiple workers need the same evidence, the lead should create one compact evidence packet instead of independently rediscovering or repeatedly transmitting it.

### Concurrency waves

The default wave contains three workers. The lead uses no more than four concurrently unless the user requested a larger count or accepted a displayed cost-and-benefit justification.

Larger jobs are handled in waves:

```text
Wave 1: discovery and independent checks
          |
          v
Lead integrates dependency-producing evidence
          |
          v
Wave 2: implementation and focused review
          |
          v
Wave 3: disputed or high-risk branches only
```

Waves limit token bursts, reduce duplicated work, and let later assignments use validated evidence from earlier ones.

The configured maximum never overrides the host application's own team or concurrency limit.

### Retry and escalation budget

The orchestration policy gives each assignment at most:

- one focused follow-up to the same worker;
- one replacement worker or profile promotion;
- zero silent scope expansions.

The escalation ladder is branch-local:

1. Correct a narrow misunderstanding or missing evidence request.
2. Promote or replace only the failed or disputed assignment.
3. Let the capable lead adjudicate if ambiguity remains.

One difficult worker does not cause the entire team to move to a more expensive model.

When the agreed budget is exhausted, the lead stops dispatching, preserves completed evidence, states what remains, and asks for a decision if the requested result cannot be completed within budget.

## Assignment contract

Every worker assignment contains the following fields:

| Field | Why it exists |
|---|---|
| `assignment_id` | Stable reference for messages, status, and integration |
| `objective` | The result the worker owns |
| `why_independent` | Evidence that a separate session is justified |
| `worker_profile` | Resource routing decision |
| `scope_in` / `scope_out` | Prevents accidental expansion |
| `owned_files_or_state` | Enforces one active writer per mutable target |
| `context_capsule` | Minimizes context cost and leakage |
| `allowed_tools_or_actions` | Grants least authority |
| `forbidden_actions` | Makes critical boundaries explicit |
| `deliverable_schema` | Makes outputs composable |
| `evidence_required` | Prevents unsupported conclusions |
| `acceptance_criteria` | Lets the lead validate success |
| `max_turns` | Sets the requested or lead-supervised worker budget |
| `timeout_or_wait_policy` | Defines monitoring behavior |
| `dependency_ids` | Prevents premature dispatch |
| `stop_conditions` | Prevents runaway work |

The ownership invariant is important: a mutable file, branch, record, or external-state target has at most one active writer. Review workers remain read-only unless ownership is explicitly reassigned.

## Lead workflow

The installed skill directs the lead to follow this sequence:

1. Test whether independent sessions are justified.
2. Show the proposed team, worker profiles, waves, boundaries, and material cost implications.
3. Obtain confirmation unless the user's current message already requests orchestration.
4. Create a ledger of required assignments and terminal conditions.
5. Select the least expensive adequate profile for each assignment.
6. Create one team and dispatch only dependency-ready work.
7. Continue useful lead-only work without duplicating worker investigations.
8. Monitor events or bounded waits with progressive backoff.
9. Correct, replace, or escalate only branches needing attention.
10. Validate and disposition every terminal report.
11. Apply the completion barrier.
12. Synthesize the final result after required workers are terminal; request shutdown and let the host perform its session-end cleanup.

This is the mechanic that makes the original task remain active while other task sessions run. The lead is not passively waiting: it owns a ledger, receives state changes, handles exceptions, and withholds completion until the barrier passes.

## Monitoring protocol

The preferred mechanism is event-driven:

- a worker sends the lead a direct message;
- the shared task state changes;
- the host reports completion or a need for attention.

If events are unavailable, the fallback is a bounded wait with progressive backoff. Rapid repeated polling is prohibited because it consumes attention and context without producing new evidence.

The lead recognizes these attention conditions:

- permission required;
- missing input;
- scope conflict;
- budget risk;
- contradictory evidence;
- worker failure.

Each condition leads to a specific intervention rather than a generic “keep trying” prompt.

## Worker terminal report

Workers return a structured report with:

```text
assignment_id
status
summary
evidence
artifacts_changed
validation
uncertainties
recommended_followup
resource_observations
```

Allowed terminal statuses are `completed`, `failed`, `cancelled`, and `needs_attention`. For final synthesis, `needs_attention` is still open and must be resolved.

Concise terminal reports prevent the lead from having to absorb an entire worker transcript. Evidence and material caveats survive; exploratory chatter does not have to.

## Completion barrier

The lead may deliver the orchestrated result only after all of these checks pass:

- No required assignment is still proposed, dispatched, running, waiting, or in need of attention.
- Every required assignment is completed, failed, or cancelled.
- Every terminal report is accepted, rejected with reason, superseded, or explicitly deferred.
- Accepted claims are traceable to evidence or labeled as inference.
- Worker disagreements are reconciled or disclosed.
- Authorized changes were validated in proportion to risk.
- Resource use is reported when available, and missing telemetry is not invented.
- Temporary coordination state is cleaned up or intentionally retained with an explanation.

If a required assignment fails, the lead can still provide useful work, but it must be labeled as partial and identify the omission, impact, available evidence, and next action.

## Safety model

The orchestrator does not grant workers new authority. Each worker receives the smallest useful combination of context, tools, permissions, and writable scope.

Workers cannot independently:

- publish or deploy;
- send external messages;
- make purchases;
- use credentials outside the authorized scope;
- alter global configuration;
- perform irreversible actions;
- create more workers or nested teams;
- mutate unowned state.

An explicit user authorization can permit a particular external effect, but that authority must be carried into the exact assignment. Team creation is not blanket permission.

## Resource telemetry

When the host exposes the information, the lead records:

- number of workers and concurrency waves;
- requested and observed model for each worker;
- requested and observed effort;
- turn count;
- input, cache, output, and total tokens;
- estimated or reported monetary cost;
- retries, replacements, and escalations;
- elapsed time;
- accepted, rejected, and unresolved deliverables.

A valid economic comparison needs a declared baseline, such as “one capable session performing the whole job.” Worker count, total raw tokens, or fewer premium-model turns alone cannot prove savings.

If the host does not expose a datum, the report says `unavailable`. It does not manufacture precision.

## Claude Code adapter

The initial adapter targets Claude Code 2.1.178 or newer and its experimental agent-team capability.

### Host terminology mapping

| Portable term | Host term |
|---|---|
| Lead session | Team lead |
| Worker session | Teammate |
| Assignment ledger | Shared task list plus lead disposition ledger |
| Attention event | Direct teammate message or task-state change |
| Completion barrier | Verification of all required shared tasks before cleanup and synthesis |

### Why the adapter does not install worker definitions

Claude Code discovers reusable user-level agent definitions beneath `~/.claude/agents`. A definition placed there can participate in later automatic delegation, including sessions unrelated to this package. Version 1.2 therefore leaves that global registry untouched.

Light, standard, and deep are policy labels inside the manually invoked skill. When the workflow is authorized, the lead names the selected model and assignment constraints directly in each teammate creation request. The runtime worker is still an independent agent-team session with its own context window; it simply has no persistent global definition.

### Manual-only skill

The installed skill frontmatter includes:

```yaml
name: cost-aware-independent-session-orchestrator
description: Manually coordinate a capable lead session and economical independent teammate sessions after the user explicitly invokes this skill. Never load or run automatically.
disable-model-invocation: true
```

The `disable-model-invocation` field prevents Claude from invoking the skill automatically and removes its description from normal model context. The user starts it explicitly with `/cost-aware-independent-session-orchestrator`.

### Session-scoped feature activation

The installer leaves agent teams disabled in persistent configuration. For a dedicated orchestration session, launch Claude Code with the experimental flag set only for that process or temporary shell:

PowerShell:

```powershell
$env:CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS='1'; claude
```

POSIX shell:

```sh
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

Close the dedicated shell afterward, or remove the temporary variable from that shell. Do not place the flag in `settings.json` unless the user separately requests persistent enablement and understands that it affects ordinary named delegation too.

### Model mapping

The portable profiles map to editable host defaults:

| Runtime role | Model | Effort in agent-team mode | Lead-supervised turn budget |
|---|---|---|---:|
| Lead | Current session | Current session | Current session |
| Light worker | `haiku` | Inherits lead | 8 |
| Standard worker | `sonnet` | Inherits lead | 12 |
| Deep worker | `sonnet` | Inherits lead | 16 |

The mapping is intentionally asymmetric. A high-capability lead can supervise several economical workers without forcing each worker to use the lead's configuration.

These aliases are adapter defaults, not universal guarantees. Availability and billing depend on the user's account. If a configured alias is unavailable, the skill discloses the host's substitution and asks for an economical alternative. It must not silently accept upgrades to the lead model.

### Installed files and protected surfaces

`INSTALL` declares these destinations:

```text
~/.claude/skills/cost-aware-independent-session-orchestrator/SKILL.md
~/.claude/skills/cost-aware-independent-session-orchestrator/INSTALL-RECEIPT.json
```

The installer must not read or modify `~/.claude/settings.json`, `~/.claude/agents/**`, memories, hooks, workflows, team state, task state, or unrelated skills. Verification and uninstallation have the same boundary.

Preflight checks the exact destination paths and performs one bounded name-collision scan beneath `~/.claude/skills`. That scan may inspect only each `SKILL.md` frontmatter `name` field; it must not ingest unrelated skill bodies or report unrelated configuration. Every collision check finishes before any directory, backup, temporary file, or receipt is created.

### Documented runtime controls versus policy budgets

Current host documentation says teammates inherit the lead's effort. It does not expose `maxTurns` as a teammate-specific control. Version 1.2 therefore distinguishes host controls from lead-supervised policy:

| Control | Status in team mode |
|---|---|
| Worker model | Named explicitly in the teammate creation request |
| Worker assignment instructions | Named explicitly in the teammate creation request |
| Worker effort | Inherits the lead in agent-team mode |
| Worker turn budget | Lead-supervised policy; not represented as a host-enforced `maxTurns` claim |

This distinction matters. The model asymmetry—the main automatic economic lever—is supported. Per-worker reasoning effort is not independently selected at teammate spawn: teammates inherit the lead's effort. A user can change the effort for later turns while viewing a teammate, but the orchestrator must not present role-frontmatter effort as an automatically applied teammate control. The lead must supervise the turn budget.

### Known host limitations

The adapter reports, rather than hides, these constraints:

- A session has exactly one session-scoped team; it cannot create additional named teams or share that team across sessions.
- Nested teams are unsupported.
- The lead role cannot be transferred.
- Teammates begin with the lead session's permission settings; individual modes can be changed after spawning.
- In-process teammates cannot be individually resumed after the session ends.
- Model availability and billing vary by account and provider configuration.
- While the session variable is enabled, named delegation can become a teammate even when the user did not explicitly request a team.
- The session's team configuration is removed on exit, but its task-list directory can persist under the host's retention policy.

The portable core forbids workers from creating additional workers, which also keeps the workflow inside the host's no-nested-team limit.

## Installation procedure

The declared installation is deliberately narrow and defensive:

1. Resolve the user home through the host's documented mechanism.
2. Reject an empty, root, ambiguous, or non-user destination.
3. Confirm application version compatibility with Claude Code 2.1.178 or newer.
4. Compute the exact skill bytes declared by the manifest.
5. Resolve the exact skill and receipt destinations.
6. Check the exact directory, skill, and receipt paths and scan only skill-name frontmatter for a duplicate name.
7. Finish all collision checks before any mutation. A difference stops installation and requires a fresh decision.
8. If replacement of the exact skill file is separately authorized, make a timestamped byte-for-byte backup and record its hash.
9. Create only the declared skill directory if needed.
10. Write the skill atomically through a temporary sibling file and verify its hash.
11. Write the receipt last using the same atomic pattern.
12. Run static verification without reading protected surfaces.

The receipt is written last so an interrupted installation cannot falsely claim that all preceding operations completed.

## Receipt and reversibility

The receipt records:

- installer name and version;
- UTC installation time;
- adapter and observed application version;
- the one managed skill destination;
- whether that exact skill existed beforehand;
- its backup location when replacement was separately authorized;
- prior and installed SHA-256 hashes;
- the manual-only installation mode and pre-mutation collision result;
- static-verification result;
- live-test status.

The receipt records its own resolved path but not its own SHA-256. A file cannot contain its own final cryptographic hash without an unsatisfiable self-reference. Receipt integrity is checked by parsing it and validating the hash it records for the managed skill.

This makes uninstallation evidence-based.

`UNINSTALL` removes or restores the skill only when its current hash still matches the installed hash in the receipt. If the user modified it after installation, it is preserved and reported. A pre-existing skill is restored only after the backup matches the recorded prior hash; a missing or mismatched backup causes a safe stop. The installer-created backup is removed only after the restored file is verified byte-for-byte.

Settings, agent definitions, memories, hooks, workflows, team state, task state, unrelated skills, and unrelated files are never uninstall targets.

After a completely successful uninstall, the receipt is removed. If an operation fails or a user-modified installed file must be preserved, the receipt remains so the unresolved uninstall can be inspected and resumed safely.

## Static verification

`VERIFY` checks:

- the manifest parses as JSON;
- the manifest declares no settings, agent, memory, hook, workflow, team, or task destination;
- the installed skill exactly matches its declared lines;
- the skill frontmatter has the exact name, a manual-only description, and `disable-model-invocation: true`;
- the receipt parses, names only the declared skill, and its installed hash matches current skill content;
- no protected surface is read as part of verification.

Static verification proves installation integrity. It does not prove that a live team behaves correctly or costs less.

## Live acceptance test

The installer does not automatically create a team because doing so consumes account resources. A live exercise requires a separate user request.

The recommended minimal test is:

1. Launch a dedicated Claude Code session with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` set only for that process or temporary shell.
2. Invoke `/cost-aware-independent-session-orchestrator` manually.
3. Keep the current session as lead and create two read-only light teammates only after explicit execution consent.
4. Give them disjoint, tiny inspection assignments.
5. Confirm that each has an independent context and produces a terminal report.
6. Confirm the lead receives state changes without tight polling.
7. Confirm the lead waits for both reports and dispositions them before synthesis.
8. Confirm that teammates inherit the lead's effort and do not claim per-teammate effort differentiation.
9. Request worker shutdown, end the dedicated session, and report which session and task state the host retains.

This test verifies topology and completion behavior at low cost. A later benchmark can compare quality, latency, raw tokens, and monetary cost against a single-session baseline.

## What the package does not promise

The package does not promise that:

- every parallel job will be cheaper;
- every worker model is available on every account;
- model names or prices will remain unchanged;
- per-teammate effort or `maxTurns` declarations control agent-team teammates;
- installation persistently enables agent teams;
- session-scoped feature activation proves behavioral correctness;
- more workers imply more coverage;
- independent sessions remove the need for lead validation;
- a receiving assistant will execute arbitrary JSON without inspecting it.

It installs a disciplined protocol and a current host adapter. Actual effectiveness must be observed on the recipient's tasks and account.

## Porting to another application

A new adapter must supply equivalents for:

1. an independent worker-session creation primitive;
2. per-worker model or resource selection;
3. a shared or lead-maintained assignment registry;
4. worker-to-lead messages or state-change events;
5. bounded waiting;
6. terminal status detection;
7. safe cleanup;
8. reversible configuration and installation receipts.

If a host lacks per-worker model selection, the orchestration topology can still work, but the economic claim weakens. If it lacks independent contexts or observable terminal states, it does not implement this mechanic.

The portable core should remain unchanged. Host names, paths, flags, model aliases, and configuration formats belong only in the new adapter.

## Source basis

Adapter claims were checked against the official documentation on 2026-09-04:

- [Agent teams](https://code.claude.com/docs/en/agent-teams): independent teammate sessions, shared tasks, automatic messages and idle notifications, feature flag, inherited effort, model selection, token guidance, and current limitations.
- [Skills](https://code.claude.com/docs/en/skills): user-level skill paths, explicit invocation, and `disable-model-invocation` behavior.
- [Model configuration](https://code.claude.com/docs/en/model-config): model aliases, supported effort levels, and the cost implications of reasoning effort.
- [Cost management](https://code.claude.com/docs/en/costs): team token scaling, the approximate plan-mode multiplier, focused prompts, small teams, and economical teammate-model guidance.
- [Custom agent definitions](https://code.claude.com/docs/en/subagents): user-level definition discovery and automatic delegation behavior, which is why this package does not install definitions there.
