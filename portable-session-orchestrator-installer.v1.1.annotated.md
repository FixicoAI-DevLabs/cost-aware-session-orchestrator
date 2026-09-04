# Cost-Aware Independent Session Orchestrator v1.1

Human-readable, annotated companion to `portable-session-orchestrator-installer.v1.1.json`.

Payload version: 1.1.2.

This document explains what the installer declares, why each mechanism exists, and how the current Claude Code adapter maps a provider-neutral orchestration protocol onto the host application. It is documentation, not an installer. The JSON file is the machine-readable package.

## Installation footprint

The installer makes one narrow settings change and creates five user-level files. Here, `~` means the recipient's home directory: for example, `C:\Users\Name` on Windows or `/home/name` on Linux.

| Action | Destination | Purpose |
|---|---|---|
| Modify by JSON merge | `~/.claude/settings.json` | Set `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` to the string `"1"` while preserving unrelated settings. |
| Create | `~/.claude/skills/coordinated-session-orchestration/SKILL.md` | Teach the lead session when and how to plan, dispatch, monitor, escalate, and finish independent-session work. |
| Create | `~/.claude/agents/economy-light-worker.md` | Define the economical, read-only worker role. |
| Create | `~/.claude/agents/economy-standard-worker.md` | Define the economical general worker role. |
| Create | `~/.claude/agents/economy-deep-worker.md` | Define the difficult-analysis worker role. |
| Create last | `~/.claude/skills/coordinated-session-orchestration/INSTALL-RECEIPT.json` | Record resolved paths, prior state, backups, versions, and hashes for verification and safe uninstallation. |

The package does **not** install a plugin, executable, daemon, model, scheduled job, or continuously running background service. It does not change the lead session's selected model or effort, and it does not automatically create a live team.

When the workflow is later used, Claude Code—not this installer—creates session-scoped team and task state beneath:

```text
~/.claude/teams/<team-name>/
~/.claude/tasks/<team-name>/
```

The source installer JSON remains wherever the recipient placed it. On current versions, the team configuration is removed when the session ends, while task-list state can persist locally under the host's retention policy.

## What behavior this adds

The package creates a reusable operating pattern with five defining properties:

1. One capable lead session owns decomposition, routing, monitoring, arbitration, and final synthesis.
2. Bounded assignments run in independent worker sessions with separate context windows.
3. Workers use economical resource profiles instead of automatically inheriting the lead's expensive configuration.
4. The lead reacts to worker completion or attention events and escalates only the branch that needs more capability.
5. The lead does not deliver the orchestrated result until every required assignment has reached a terminal state and been dispositioned.

The cost policy is not based on the idea that parallel sessions magically use fewer raw tokens. They often use more. Current official cost guidance warns that agent teams can use approximately seven times the tokens of standard sessions in plan-mode configurations. The intended savings come from allocating cheaper models to most worker work, minimizing repeated context, imposing supervisory turn and retry budgets, and reserving the capable lead for decisions that benefit from it.

## Version lineage

Version 1.1 preserves the original orchestration state machine and adds an explicit economic control plane.

The machine-readable lineage record identifies:

- predecessor file: `portable-session-orchestrator-installer.v1.json`
- predecessor version: `1.0.0`
- predecessor SHA-256: `13FE98547AC72F6E182C1EEB86594F9D3AD51DD9517279630B738A98CC93D84E`

The old file remains intact. This matters because a portable installer should have an auditable upgrade path instead of silently replacing its own history.

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
        feature enablement
        model mappings
        installed skill
        reusable worker-role definitions
        install / verify / uninstall procedures
```

The `portable_core` contains no provider-specific model names, feature flags, paths, or commands. A different application can implement the same protocol by supplying a new adapter without modifying the core.

## Activation contract

The receiving assistant recognizes exactly three commands when the user explicitly refers to the JSON file:

| Command | Meaning | Mutation allowed |
|---|---|---|
| `INSTALL` | Preflight, back up, merge settings, write declared files, and verify | Only declared installation changes |
| `VERIFY` | Check configuration, file contents, hashes, and compatibility | None |
| `UNINSTALL` | Remove or restore only content proven by the receipt | Only proven installed content |

Merely opening the JSON does nothing. JSON is data, not executable code. The receiving assistant must interpret it after the explicit command.

Before `INSTALL` mutates anything, it must show:

- resolved destination paths;
- a focused description of the settings merge;
- files that will be created or changed;
- backup behavior;
- compatibility warnings;
- any collision with existing user content.

The command authorizes only this installation. It does not authorize unrelated configuration changes, external communication, credential use, publication, deployment, or destructive work.

### Idempotency

If exactly the same version and content are already installed, `INSTALL` reports success without rewriting files.

If a destination exists with different content, the installer preserves it, shows a focused diff, and asks for a fresh decision. Existing user work is never assumed to be disposable.

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

### Why the adapter writes files under `~/.claude/agents`

Claude Code stores reusable worker-role definitions in `~/.claude/agents`. That directory name describes the host's configuration format, not the runtime topology selected by this package.

When the lead uses one of these definitions to create an agent-team teammate, the resulting worker is an independent session with its own context window. It is not an in-process helper call merely because its role came from an agent-definition file.

The distinction is:

```text
Definition file: reusable model, tools, instructions, requested effort, and requested turn budget
Runtime creation: independent agent-team teammate session
```

### Feature setting

The adapter merges one key into `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

This is a merge, not a replacement. Every unrelated setting is preserved. Invalid existing JSON causes a stop rather than an overwrite.

On current Claude Code versions, enabling this flag also changes ordinary delegation: a named delegation can launch as a teammate even when the user did not explicitly request a team. The installed policy supplies a consent gate, but it cannot narrow the host feature flag itself. Recipients who do not accept that broader host behavior should leave the flag disabled.

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

### Installed files

`INSTALL` declares these destinations:

```text
~/.claude/skills/coordinated-session-orchestration/SKILL.md
~/.claude/agents/economy-light-worker.md
~/.claude/agents/economy-standard-worker.md
~/.claude/agents/economy-deep-worker.md
~/.claude/skills/coordinated-session-orchestration/INSTALL-RECEIPT.json
```

The skill contains orchestration decisions and workflow. The three role definitions contain reusable resource envelopes and worker behavior.

The light role's documented tool allowlist contains only `Read`, `Grep`, and `Glob`. It therefore excludes command execution and file-editing tools while leaving host-required team-coordination tools available.

The standard and deep roles may use the host's inherited tool and permission set but are restricted by the assignment's explicit ownership and scope.

### Documented controls versus requested budgets

Current host documentation explicitly says that an agent-team teammate created from a reusable definition honors the definition's model, tool allowlist, and instruction body. It also says teammates inherit the lead's effort, and the teammate-definition mapping does not list `maxTurns`.

For that reason, v1.1 treats the controls differently:

| Control | Status in team mode |
|---|---|
| Worker model | Documented host control |
| Worker tool allowlist | Documented host control |
| Worker instruction body | Documented host control |
| Worker effort | Inherits the lead in agent-team mode; role value applies only in ordinary subagent mode |
| Worker `maxTurns` | Role value applies in ordinary subagent mode; lead-supervised budget in agent-team mode |

This distinction matters. The model asymmetry—the main automatic economic lever—is supported. Per-worker reasoning effort is not independently selected at teammate spawn: teammates inherit the lead's effort. A user can change the effort for later turns while viewing a teammate, but the orchestrator must not present role-frontmatter effort as an automatically applied teammate control. The lead must supervise the turn budget.

### Known host limitations

The adapter reports, rather than hides, these constraints:

- A session has exactly one session-scoped team; it cannot create additional named teams or share that team across sessions.
- Nested teams are unsupported.
- The lead role cannot be transferred.
- Teammates begin with the lead session's permission settings; individual modes can be changed after spawning.
- In-process teammates cannot be individually resumed after the session ends.
- Model availability and billing vary by account and provider configuration.
- Enabling agent teams can turn a named delegation into a teammate even when the user did not explicitly request a team.
- The session's team configuration is removed on exit, but its task-list directory can persist under the host's retention policy.

The portable core forbids workers from creating additional workers, which also keeps the workflow inside the host's no-nested-team limit.

## Installation procedure

The declared installation is deliberately defensive:

1. Resolve the user home through the host's documented mechanism.
2. Reject an empty, root, ambiguous, or non-user destination.
3. Confirm application version compatibility with Claude Code 2.1.178 or newer.
4. Parse the existing settings file; stop if it is invalid JSON.
5. If top-level `env` exists but is not a JSON object, stop rather than coercing or overwriting it.
6. Compute a merge that touches only the experimental feature flag.
7. Back up every existing destination that would change.
8. Create missing skill and role directories.
9. Write declared content using UTF-8 with final newlines.
10. Replace settings atomically through a temporary sibling file.
11. Write the receipt last.
12. Run static verification.

The receipt is written last so an interrupted installation cannot falsely claim that all preceding operations completed.

## Receipt and reversibility

The receipt records:

- installer name and version;
- UTC installation time;
- adapter and observed application version;
- every managed destination other than the receipt itself;
- whether each destination existed beforehand;
- backup locations;
- prior and installed SHA-256 hashes;
- prior and installed feature-flag values;
- static-verification result;
- live-test status.

The receipt records its own resolved path but not its own SHA-256. A file cannot contain its own final cryptographic hash without an unsatisfiable self-reference. Receipt integrity is checked by parsing it and validating the hashes it records for settings, skill, and worker-role content.

This makes uninstallation evidence-based.

`UNINSTALL` removes an installed file only when its current hash still matches the receipt. If the user modified the file after installation, it is preserved and reported.

The feature flag is removed only if the receipt proves the installer added it and its value is unchanged. If it existed before installation, its recorded prior value is restored.

Unrelated settings and files are always preserved.

After a completely successful uninstall, the receipt is removed. If an operation fails or a user-modified installed file must be preserved, the receipt remains so the unresolved uninstall can be inspected and resumed safely.

## Static verification

`VERIFY` checks:

- the manifest and host settings parse as JSON;
- the experimental feature flag has the declared value;
- the installed skill exactly matches its declared lines;
- all three role files exactly match their declarations;
- required frontmatter fields exist;
- the standard and deep roles contain effort values for ordinary subagent use, while clearly stating that teammates inherit the lead's effort;
- the light role does not claim an unsupported effort override;
- the receipt parses and its recorded hashes for settings, skill, and worker-role content match current content; the receipt is not required to hash itself.

Static verification proves installation integrity. It does not prove that a live team behaves correctly or costs less.

## Live acceptance test

The installer does not automatically create a team because doing so consumes account resources. A live exercise requires a separate user request.

The recommended minimal test is:

1. Keep the current session as lead.
2. Create two read-only light teammates.
3. Give them disjoint, tiny inspection assignments.
4. Confirm that each has an independent context and produces a terminal report.
5. Confirm the lead receives state changes without tight polling.
6. Confirm the lead waits for both reports and dispositions them before synthesis.
7. Confirm that teammates inherit the lead's effort and do not claim per-teammate effort differentiation.
8. Request worker shutdown, end the session when appropriate, and report which session and task state the host retains.

This test verifies topology and completion behavior at low cost. A later benchmark can compare quality, latency, raw tokens, and monetary cost against a single-session baseline.

## What the package does not promise

The package does not promise that:

- every parallel job will be cheaper;
- every worker model is available on every account;
- model names or prices will remain unchanged;
- per-teammate effort or `maxTurns` declarations control agent-team teammates;
- feature enablement proves behavioral correctness;
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

Adapter claims were checked against the official documentation on 2026-09-03:

- [Agent teams](https://code.claude.com/docs/en/agent-teams): independent teammate sessions, shared tasks, automatic messages and idle notifications, reusable-definition model and tool behavior, feature flag, minimum version, and current limitations.
- [Model configuration](https://code.claude.com/docs/en/model-config): model aliases, supported effort levels, and the cost implications of reasoning effort.
- [Cost management](https://code.claude.com/docs/en/costs): team token scaling, the approximate plan-mode multiplier, focused prompts, small teams, and economical teammate-model guidance.
- [Custom agent definitions](https://code.claude.com/docs/en/subagents): user-level definition paths and supported frontmatter fields.
- [Settings](https://code.claude.com/docs/en/settings): the default teammate-model setting and settings structure.
