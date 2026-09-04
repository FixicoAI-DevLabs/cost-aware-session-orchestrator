# Cost-Aware Independent Session Orchestrator

Version 1.2.0 · safe-default Claude Code adapter · CC0-1.0

This package installs a reusable Claude Code workflow in which one capable lead session plans and supervises several economical, independent teammate sessions. The lead monitors their progress, handles exceptions, escalates only difficult branches, and waits for every required assignment before delivering the combined result.

## Package contents

| File | Audience | Purpose |
|---|---|---|
| `portable-session-orchestrator-installer.v1.2.json` | Receiving assistant | Machine-readable `INSTALL`, `VERIFY`, and `UNINSTALL` contract |
| `portable-session-orchestrator-installer.v1.2.annotated.md` | Humans | Detailed explanation of the mechanic, resource policy, safeguards, and host adapter |
| `README.md` | Humans | This quick-start and installation overview |

## What INSTALL changes

Here, `~` means the recipient's home directory—for example, `C:\Users\Name` on Windows or `/home/name` on Linux.

| Action | Destination | Purpose |
|---|---|---|
| Create | `~/.claude/skills/cost-aware-independent-session-orchestrator/SKILL.md` | Manual-only orchestration skill |
| Create last | `~/.claude/skills/cost-aware-independent-session-orchestrator/INSTALL-RECEIPT.json` | Exact ownership, backup, collision, and hash evidence |

That is the complete persistent footprint. The installer does **not** read or modify `settings.json`; install worker definitions; touch memories, hooks, workflows, teams, tasks, or other skills; or create a plugin, executable, daemon, model, scheduled job, or background service.

The installed skill includes `disable-model-invocation: true`, so Claude cannot load it automatically. The user must invoke `/cost-aware-independent-session-orchestrator` explicitly.

## Requirements

- Claude Code 2.1.178 or newer.
- An account and provider configuration that can use the selected model aliases.
- Permission to create the one declared user-level skill directory.
- A receiving assistant capable of reading this JSON manifest and performing its declared file operations.

Agent teams are experimental and disabled by default. This installer deliberately leaves them disabled. For a live run, launch a dedicated interactive session with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` set only for that process or temporary shell. While enabled, named delegation can launch teammates even without an explicit team request, so use that session only for deliberate orchestrated work.

## Install

Keep all three package files together, open Claude Code where it can read them, and enter:

```text
@portable-session-orchestrator-installer.v1.2.json INSTALL
```

Before mutation, the receiving assistant must resolve every destination and finish the exact-path and same-name collision checks. If anything differs, it stops before creating a directory, backup, temporary file, or receipt; preserves the existing content; shows only the relevant conflict; and requests a fresh decision.

The installer does not read `settings.json` or unrelated skill bodies. It writes the skill atomically, writes the receipt last, and then performs static verification.

## Verify

Run a read-only integrity check with:

```text
@portable-session-orchestrator-installer.v1.2.json VERIFY
```

Verification checks only the manifest, declared skill, manual-invocation frontmatter, receipt, and SHA-256 hash. It does not read settings, agents, memories, workflows, hooks, teams, tasks, or unrelated skill bodies. It does not launch teammates.

## Uninstall

Run evidence-based removal with:

```text
@portable-session-orchestrator-installer.v1.2.json UNINSTALL
```

Uninstallation operates only on the declared skill file, receipt, and skill directory. It removes or restores the skill only when the current and backup hashes prove the operation is safe. Modified or ambiguous files are preserved, and the receipt remains until every owned operation succeeds. Settings and all other user content are outside uninstall scope.

## Runtime model policy

| Runtime role | Model | Effort in agent-team mode | Lead-supervised turn budget |
|---|---|---|---:|
| Lead | Current session | Current session | Current session |
| Light teammate | `haiku` | Inherits lead | 8 |
| Standard teammate | `sonnet` | Inherits lead | 12 |
| Deep teammate | `sonnet` | Inherits lead | 16 |

The lead routes each assignment to the least expensive role likely to meet its acceptance criteria. Normal work is dispatched in waves of two to four teammates, with three as the default. More than four concurrent teammates requires an explicit requested count or an accepted cost-and-benefit justification.

One failed branch can receive one focused correction and one replacement or profile promotion. Unrelated workers are not upgraded merely because one assignment is difficult.

## Independent sessions, not ordinary subagent calls

The skill instructs the lead to create named agent-team teammates with an explicit economical model in each spawn request. Those workers are full independent Claude Code sessions with separate context windows, automatic messages, and shared task state.

No files are installed beneath `~/.claude/agents`. Light, standard, and deep are routing labels embedded in the manual skill, not globally discoverable agent definitions. Claude Code says teammates inherit the lead's effort and does not expose `maxTurns` as a teammate control, so turn limits are lead-supervised budgets rather than host-enforced limits.

The main automatic economic lever is therefore heterogeneous model routing, not heterogeneous reasoning effort. If lower effort per independent worker is essential, use separately launched sessions or manually change the viewed teammate's effort where the host supports it.

## Cost warning

Independent sessions are not inherently token-saving. Official guidance warns that agent teams can consume approximately seven times the tokens of standard sessions in plan-mode configurations.

The economic strategy is therefore to:

- keep worker context packets small;
- use economical worker models;
- keep teams and assignments bounded;
- dispatch in dependency-aware waves;
- restrict retries and replacements;
- escalate only the branch that needs more capability;
- report cost or token telemetry only when the host exposes reliable data.

Do not claim savings merely because fewer worker turns used the lead's model.

## What happens during a live team run

Claude Code creates and manages runtime state beneath:

```text
~/.claude/teams/<team-name>/
~/.claude/tasks/<team-name>/
```

Those directories are not installed by this package. On Claude Code 2.1.178 and later, the session creates them automatically. The team configuration is removed when the session ends, while task-list state can persist locally under the host's retention policy.

The lead does not deliver the orchestrated answer until every required assignment is terminal and dispositioned. Failed or cancelled required work produces an explicitly labeled partial result rather than a false completion claim.

## Further detail

Read `portable-session-orchestrator-installer.v1.2.annotated.md` for the complete state model, eligibility gate, assignment contract, monitoring loop, completion barrier, safety rules, adapter limitations, and portability guidance.

Official references:

- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Skills](https://code.claude.com/docs/en/skills)
- [Model configuration](https://code.claude.com/docs/en/model-config)
- [Cost management](https://code.claude.com/docs/en/costs)
- [Custom agent definitions](https://code.claude.com/docs/en/subagents)

## Integrity

SHA-256 checksums for the two versioned payload files:

```text
B655D66AE2CC26A3ED99EC5410EE6863009A8B72D9F0E9C05893D0792D5CE54E  portable-session-orchestrator-installer.v1.2.json
B395F7250C098C96424843DD61C61EA32EAB7DD944DB33247FACAC1A20269D88  portable-session-orchestrator-installer.v1.2.annotated.md
```
