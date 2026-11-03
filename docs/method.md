# Method

## Scope

**In scope:** The Watchman v0.4.0 at commit [`c4e5cdb`](https://github.com/Iamendlesss/the-watchman/tree/c4e5cdba5e0223a116a5d65bdb90bfb636af467b), including its source, tracked config, dependencies, repository settings, design spec and project notes. The planned live executor is covered at design level.

**Out of scope:** The security of Solana, pump.fun, Helius, RugCheck and Discord themselves, which are treated as outside parties that can fail or misbehave. Whether the trading strategy is profitable is also out of scope.

## Why STRIDE

STRIDE walks through each component and asks six fixed questions, which suits a design review of one application. MITRE ATT&CK and the cyber kill chain describe how attackers behave across an intrusion, which is more useful for detection work than for reviewing an application's design.

## Steps

1. Read the source, spec and project notes to list components and data flows.
2. Draw trust boundaries wherever data or code changes owner.
3. Apply STRIDE to each component and list threats.
4. Check suspected flaws against the code and test the ones that can be tested offline.
5. Confirm which existing controls work.
6. Rate each threat for today and for a future live mode.
7. Turn the highest risks into [go-live gates](live-readiness.md).

## Rating scale

The rating is likelihood × impact. These are judgement calls meant to set priorities, not measurements. Live ratings assume the planned controls are not built yet, because none of them are.

| Likelihood | Meaning |
|---|---|
| 1 Low | Needs prior access or rare conditions |
| 2 Medium | Feasible for a motivated attacker, or happens by accident |
| 3 High | Cheap, common, or already observed |

| Impact | Meaning |
|---|---|
| 1 Low | Noise or inconvenience |
| 2 Medium | API key misuse, misleading data, or one position lost |
| 3 High | Several positions lost, or a wrong go-live decision |
| 4 Critical | Whole wallet lost, or control of the operator's machine |

| Score | Rating |
|---|---|
| 1 to 2 | Low |
| 3 to 4 | Medium |
| 6 | High |
| 8 to 12 | Critical |

## Evidence collected

All checks ran on 3 November 2026.

| Check | Result |
|---|---|
| Source review | All 25 source files and the helper script |
| Offline logic tests | The bot's own gate, parsing and tracker functions, run with crafted input and stubbed network calls |
| Secret scan | Pattern search of every commit, excluding the lockfile and IDL. Only documented public endpoints and test placeholders were found |
| Dependency audit | `npm audit` against the lockfile: 5 vulnerable packages, 3 high and 2 moderate |
| Lockfile review | 120 packages, all from `registry.npmjs.org`, 3 with install scripts |
| Dangerous code search | No `eval` or `child_process`. The only SQL interpolation adds a fixed clause, so no query contains outside data |
| Repository settings | Secret scanning and push protection on, Dependabot off, `main` not protected, no CI workflows |

## Limits of this review

- The bot was not run. The logic tests covered pure functions only, with no live network traffic and no database.
- The operator's `.env` file and local ledger were not read. Their state comes from the project notes.
- The early-trade recovery test used a stub that follows the documented RPC behaviour, not a live node.
- The secret scan was pattern based, so it could miss a secret in an unusual format.
- The project's 174 test cases were counted but not run.
- Audit results change over time, so they apply to the review date only.
- Ratings are one reviewer's judgement.
