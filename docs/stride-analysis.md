# STRIDE Analysis

All threats to [The Watchman](https://github.com/Iamendlesss/the-watchman) v0.4.0 at commit `284b4f3`, by component.

**STRIDE:** Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege.

**Ratings:** likelihood × impact, before any planned control is built. See the [method](method.md) for the scale.

**Evidence:** *Tested* means reproduced offline with crafted input or confirmed by a tool. *Code* means confirmed by reading the source. *Notes* means recorded in the project's private operating notes. *Design* means reasoned from the planned architecture.

## 1. Launch stream and tracker

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-01 | **Crafted launch.** One operator with many wallets looks like a healthy crowd. The gate counts wallets, not people, and treats a single wallet as the creator. | S | Medium (3×1) | Critical (3×3) | Tested | Cluster wallets by funding source, keep a creator history blocklist, keep positions small |
| T-02 | **Predictable filter.** The thresholds and judging time are public in the repo, so a launcher can aim for them. | I | Low (1×1) | High (2×3) | Code | Keep live tuning private, add small random timing jitter |
| T-05 | **Decisions on partial data.** Stream gaps are not tracked per coin and early-trade recovery is best effort, so a coin can be judged without knowing part of its history is missing. | T, R | Medium (3×1) | High (3×2) | Tested | Track completeness per coin, reject a coin when part of its window is missing |
| T-06 | **Stale prices during outages.** With every connection down, stop rules keep running on the last known price. Project notes record a 28-minute outage. | D | Medium (3×1) | Critical (3×3) | Code, Notes | Stale-price guard, second independent provider, defined outage policy |
| T-07 | **One data source trusted.** Stream data is treated as chain truth with no second check, and a custom stream address is not forced to use TLS. | S, T | Low (1×1) | Medium (1×3) | Code | Require `wss` and a host allowlist, confirm entry data through a second provider |
| T-08 | **Program upgrade drift.** pump.fun can change its program. A layout change could be misread, and decode errors only appear in a status line. | T | Low (2×1) | Medium (2×2) | Code | Alert on decode-error spikes, check the IDL hash in CI |

## 2. Safety gate

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-03 | **Third-party check fails open.** If RugCheck is down, rate limited, or changes its response format, a coin can pass with no danger recorded. | T, D | Medium (3×1) | High (3×2) | Tested | Validate the response shape, make the check required in live mode, alert on skip rates |
| T-04 | **Mint check gaps.** Token extensions are checked against a deny-list, and missing authority fields read as safe. | T | Low (1×1) | Medium (2×2) | Tested | Allowlist known-safe extensions, parse the response with a strict schema |
| T-09 | **No sell simulation.** The design calls for proving a coin can be sold before buying it. That check is not built yet. | S | Low (2×1) | Medium (2×2) | Code | Simulate a sell before every live entry |

## 3. Risk manager and kill switch

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-10 | **Limits are settings, not hard caps.** Config can raise position size close to the whole wallet or switch off safety stages, and one command resets the drawdown floor. | E, T | Low (1×1) | Critical (2×4) | Code | Hard caps in code for live mode, log a config hash at start, alert when a safety stage is off, confirm on resume |
| T-11 | **Kill switch limits.** The STOP file only blocks new entries, and its path depends on the folder the bot starts from. | T, D | Low (1×1) | High (2×3) | Code | Anchor the path to the project root, add an emergency exit for open positions |

## 4. Ledger

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-12 | **No tamper evidence.** The ledger is an ordinary local database. Paper results decide whether the bot goes live, so edited or lost records could drive the wrong call. | T, R | Medium (1×3) | Medium (1×3) | Code | Hash-chain the decisions, back up separately, treat the chain as the truth in live mode |
| T-13 | **Recorded sales that never happened.** Restart logic closes leftover positions at an estimated price. That is acceptable on paper and wrong if carried into live mode. | R, T | Low (2×1) | High (2×3) | Code | Reconcile wallet against ledger at start, record a close only after on-chain confirmation |
| T-14 | **Launch flood.** Under extreme launch volume the tracker drops its oldest coin first, even one that is held, and the recovery path closes every open position. | D | Low (1×1) | Medium (1×3) | Tested, Notes | Never evict held coins, close only the affected position, release the subscription on eviction |

## 5. Alerts and monitoring

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-15 | **Fake alerts.** Anyone holding the webhook URL can post as the bot, and alerts are not authenticated. | S | Low (1×1) | Low (1×2) | Code | Treat alerts as advisory, confirm locally before acting, rotate the URL if exposed |
| T-16 | **Silent failure.** Monitoring is messages from the bot itself. If the Mac sleeps or the bot dies, nothing raises an alarm. | D | Low (2×1) | High (2×3) | Code, Notes | External dead-man switch that alarms when heartbeats stop |
| T-17 | **Terminal injection.** Coin names are cleaned before Discord, but some console lines print them raw. | T | Low (1×1) | Low (1×2) | Code | Reuse the existing cleaning function for console output |

## 6. Secrets and local config

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-18 | **Plain-text secrets file.** API keys sit in `.env`. Permissions are checked by only one command, and backup or sync tools can copy the file. | I | Medium (2×2) | Critical (2×4) | Code | Keychain storage, permission check at every start, exclude from backups and sync |
| T-19 | **Trading key shared with every package.** An environment variable is visible to all code in the process, and paper mode does not refuse to start when the key is set. | I, E | Low (1×1) | Critical (2×4) | Code | Separate signer process, refuse to start in paper mode if the key is set |
| T-20 | **Unrestricted endpoint settings.** The Helius key goes to whichever host the config names, and it travels in the URL. | I | Low (1×2) | Medium (1×3) | Code | Host allowlist and `https`/`wss` check, the same pattern the Discord sender already uses |

## 7. Supply chain and source

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-21 | **Malicious or vulnerable packages.** There are 120 packages, 3 of them with install scripts, and development tools run in the same folder as `.env`. The audit on the review date reported 5 vulnerable packages: 3 high and 2 moderate. One is a runtime package whose affected feature the bot does not use on network input. The other 4 are development tools. | T, E | Medium (1×4) | Critical (2×4) | Tested | Built on 3 November 2026: a [security pipeline](https://github.com/Iamendlesss/the-watchman/blob/main/docs/security-pipeline.md) with dependency, secret and code scanning, `npm ci` only, and Dependabot. 4 of the 5 flagged packages were upgraded, and the runtime one has a dated exception until it can be migrated |
| T-22 | **Source integrity.** At review, `main` had no branch protection or required checks, there was no CI, and Dependabot was off. A stolen GitHub token could change what runs after the next pull. | T | Low (1×2) | Medium (1×4) | Tested | Built on 3 November 2026: `main` requires the Security gate for everyone, including the owner, force-pushes are blocked, and Dependabot is on. Still planned: signed commits, and running live mode from a reviewed tag |
| T-23 | **Operational details in public.** Project notes stated which secrets are set, where the bot runs, and its known issues. | I | Low (2×1) | Medium (2×2) | Code | Fixed on 3 November 2026 for the notes, which were removed from the repo and its history. Live tuning is covered by T-02 |

## 8. AI coding assistant

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-24 | **Prompt injection.** Coin names are attacker-chosen text that reach the ledger and logs an assistant reads. Public issues and pull requests are a second route. The assistant can run commands on the machine holding `.env`. | E, I | Medium (1×4) | Critical (2×4) | Code, Design | Block secret files at tool-permission level rather than by instruction, treat ledger text as data, approve commands by hand |

## 9. Live executor (not built)

| ID | Threat | STRIDE | Now | If live | Evidence | Planned control |
|---|---|---|---|---|---|---|
| T-25 | **Hostile swap instructions.** A remote API supplies transaction data, so a compromised API could include a drain. | T, E | Not built | Critical (2×4) | Design | Decode every transaction before signing: program allowlist, fee payer check, no transfers to unknown accounts |
| T-26 | **Sandwich attacks.** Forced exits accept any slippage by design, and entries allow a 10% shortfall, which gives other traders room to profit from the bot's orders. | T | Not built | High (3×2) | Code, Design | Per-transaction slippage and tip caps, private transaction routing |
