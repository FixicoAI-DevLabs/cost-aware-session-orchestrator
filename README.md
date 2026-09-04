# Cost-Aware Independent Session Orchestrator

Version 1.1.2 · Claude Code adapter · CC0-1.0

This package installs a reusable Claude Code workflow in which one capable lead session plans and supervises several economical, independent teammate sessions. The lead monitors their progress, handles exceptions, escalates only difficult branches, and waits for every required assignment before delivering the combined result.

## Package contents

| File | Audience | Purpose |
|---|---|---|
| `portable-session-orchestrator-installer.v1.1.json` | Receiving assistant | Machine-readable `INSTALL`, `VERIFY`, and `UNINSTALL` contract |
| `portable-session-orchestrator-installer.v1.1.annotated.md` | Humans | Detailed explanation of the mechanic, resource policy, safeguards, and host adapter |
| `README.md` | Humans | This quick-start and installation overview |

## What INSTALL changes

Here, `~` means the recipient's home directory—for example, `C:\Users\Name` on Windows or `/home/name` on Linux.

| Action | Destination | Purpose |
|---|---|---|
| Merge one setting | `~/.claude/settings.json` | Enable experimental agent teams while preserving unrelated settings |
| Create | `~/.claude/skills/coordinated-session-orchestration/SKILL.md` | Lead-session orchestration policy |
| Create | `~/.claude/agents/economy-light-worker.md` | Economical read-only teammate role |
| Create | `~/.claude/agents/economy-standard-worker.md` | Economical general teammate role |
| Create | `~/.claude/agents/economy-deep-worker.md` | High-effort, bounded teammate role |
| Create last | `~/.claude/skills/coordinated-session-orchestration/INSTALL-RECEIPT.json` | Hashes, backups, prior settings, and uninstall evidence |

The settings merge adds:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

The installer does **not** install a plugin, executable, daemon, model, scheduled job, or continuously running service. It does not change the lead's current model or effort, and it does not create or run a team during installation.

## Requirements

- Claude Code 2.1.178 or newer.
- An account and provider configuration that can use the selected model aliases.
- Permission to modify the recipient's user-level `~/.claude` configuration.
- A receiving assistant capable of reading this JSON manifest and performing its declared file operations.

Agent teams are experimental and disabled by default. The installer enables the documented feature flag but does not claim that static installation proves live behavior.

On current Claude Code versions, enabling that flag also changes ordinary delegation: a named delegation can launch as a teammate even when the user did not explicitly ask for a team. The installed skill supplies a consent gate, but the underlying feature flag is broader than this package. Disable or uninstall the flag if that host behavior is not acceptable.

## Install

Keep all three package files together, open Claude Code where it can read them, and enter:

```text
@portable-session-orchestrator-installer.v1.1.json INSTALL
```

Before mutation, the receiving assistant must show the resolved destinations, intended settings merge, backups, compatibility warnings, and collisions. If an existing destination differs, it must preserve the file and request a fresh decision rather than overwrite it.

If the existing settings file is invalid JSON, or if its top-level `env` member exists but is not a JSON object, installation stops without coercing or overwriting that content.

The installer writes timestamped backups for changed existing files, writes the receipt last, and then performs static verification.

## Verify

Run a read-only integrity check with:

```text
@portable-session-orchestrator-installer.v1.1.json VERIFY
```

Verification checks settings, declared file contents, frontmatter, receipt data, and SHA-256 hashes. It does not launch teammates or consume resources for a live team test.

## Uninstall

Run evidence-based removal with:

```text
@portable-session-orchestrator-installer.v1.1.json UNINSTALL
```

Uninstallation removes a file only when its current hash matches the installation receipt. User-modified files are preserved. The settings flag is removed or restored only when the receipt proves what the installer changed. The receipt is removed after a fully successful uninstall, but retained when unresolved or modified content remains.

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

Claude Code stores reusable teammate-role definitions beneath `~/.claude/agents`, which can make the naming confusing. When those definitions are used to create agent-team teammates, the workers are full independent Claude Code sessions with separate context windows, automatic messages, and shared task state.

The host currently documents that teammate definitions control the worker model, tool allowlist, and added instructions. It also says teammates inherit the lead's effort and does not list `maxTurns` as a teammate-definition control. The role files retain `effort` and `maxTurns` for ordinary subagent use, but in agent-team mode this package treats turn counts only as lead-supervised budgets and does not claim per-teammate effort differentiation.

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

Read `portable-session-orchestrator-installer.v1.1.annotated.md` for the complete state model, eligibility gate, assignment contract, monitoring loop, completion barrier, safety rules, adapter limitations, and portability guidance.

Official references:

- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Model configuration](https://code.claude.com/docs/en/model-config)
- [Cost management](https://code.claude.com/docs/en/costs)
- [Custom agent definitions](https://code.claude.com/docs/en/subagents)
- [Settings](https://code.claude.com/docs/en/settings)

## Integrity

SHA-256 checksums for the two versioned payload files:

```text
D091998DD80EA7AD04B241ED77A5E129C9D87187984DEED69058D39683A01769  portable-session-orchestrator-installer.v1.1.json
7F498AF01148A79BFDA5D78C6CAB8C623FA57FC5C277D75457E9739C723C7044  portable-session-orchestrator-installer.v1.1.annotated.md
```
