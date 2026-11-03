# Live-Readiness Gates

The Watchman is a paper bot today, and live mode is refused in code. These gates define what must be true before that refusal could be removed. Every Critical threat in the [STRIDE analysis](stride-analysis.md) maps to at least one gate.

All 13 gates are open as of 3 November 2026.

| Gate | Requirement | Threats closed |
|---|---|---|
| G-01 | **Crafted-launch defence.** Wallets are clustered by funding source, creator history is checked, and position sizes stay small until results justify more. | T-01, T-02 |
| G-02 | **Key isolation.** A separate signer process holds the key, the bot process cannot read it, and the wallet is new, dedicated, and funded in stages. | T-18, T-19 |
| G-03 | **Verify before signing.** Every transaction is decoded and refused unless all programs are on the allowlist, the fee payer is the trading wallet, and no transfer goes to an unknown account. | T-25 |
| G-04 | **Simulate before sending.** Every buy and a matching sell are simulated, with slippage, priority fee and tip caps. | T-09, T-26 |
| G-05 | **Hard caps in code.** Position size, daily loss and drawdown limits cannot be loosened by config, and resuming after a halt needs a confirmation step. | T-10 |
| G-06 | **Checks fail closed.** The safety gate rejects a coin when an outside check is missing or malformed, responses are parsed with strict schemas, and token extensions use an allowlist. | T-03, T-04 |
| G-07 | **Complete, fresh data.** A coin with a gap in its observation window is rejected, stale prices trigger a defined outage policy, and entry data is confirmed through a second provider. | T-05, T-06, T-07 |
| G-08 | **Outside monitoring.** An external dead-man switch alarms when heartbeats stop, and decode errors raise an alert. | T-08, T-16 |
| G-09 | **Working kill switch.** The STOP path is anchored to the project root, and an emergency command can exit open positions. | T-11 |
| G-10 | **Chain is the source of truth.** The wallet is reconciled against the ledger at every start, a position closes only after on-chain confirmation, and held coins are never evicted. | T-12, T-13, T-14 |
| G-11 | **Protected pipeline.** CI runs dependency, secret and code scanning and blocks merges on failure. `main` is protected, Dependabot is on, and live mode runs from a reviewed tag. | T-21, T-22 |
| G-12 | **Assistant containment.** Secret files are blocked at tool-permission level, ledger text is treated as data, and commands need human approval. | T-24 |
| G-13 | **Private operations.** Live tuning and operational notes stay out of the public repo. | T-02, T-23 |

## Should fix, not blocking

These are worth fixing but would not stop a small, closely watched live test.

| ID | Item | Why it is not blocking |
|---|---|---|
| T-15 | Fake alerts through the webhook | Requires the URL to leak, and alerts do not trigger any action by themselves |
| T-17 | Terminal injection through coin names | Limited effect, and the fix reuses an existing function |
| T-20 | Unrestricted endpoint settings | Requires a config change, and G-11 protects that path |

## Order of work

1. **G-11 first.** A security pipeline is cheap, and it protects every later change.
2. **G-06 and G-12 next.** Both close real gaps with small changes, and neither needs live mode to exist.
3. **G-07 and G-08 after that.** These also improve the quality of paper results, which decide whether live mode happens at all.
4. **The remaining gates only if live mode is approved.** Paper results come first.
