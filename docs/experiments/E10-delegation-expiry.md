# E10: Delegation expiry as a ledger event (agent-trust R-12)

Spec: [`experiments/e10-delegation-expiry.toml`](../../experiments/e10-delegation-expiry.toml) ·
Topology: [`topologies/three-agents.toml`](../../topologies/three-agents.toml) ·
Evidence: [`results/E10/`](../../results/E10/)

## Question

The formal companion of agent-trust predicted a progress gap (alignment note A-5).
Delegations carry `expires_at`, and the kernel evaluated it at each request. So an
expired reader made every release fail, but expiry never closed the fence. No repair
could then remove the reader, and the group stalled until an admin revoked it. R-12
(D-16) records the expiry as a ledger event at the first mutating kernel step after
`expires_at`, with `revoke`'s cascade and fence rule. The event is kept even when that
step fails. Does the stall go away on the network, with the APF's real clock?

## Setup

alice (root); bob delegated `read, write, admit` for the default 30 days; carol the
same rights for 45 seconds (`ttl_secs = 45`). Both are admitted, and m1 reaches carol.
The run then waits 47 seconds.

## Hypotheses and results

| | Hypothesis | Steps | Observed |
|---|---|---|---|
| H1 | Before any kernel step after the expiry, nothing is recorded and the group is not fenced | B2, B3 | No `expire` event; open; carol in the roster |
| H2 | The next release attempt is refused as fenced, and the expiry is recorded although that release failed | C1–C3 | `read_fenced`; one `expire` event, the last in the history; fenced |
| H3 | An ordinary repair removes carol and reopens the group; later messages reach bob only | D1–D6 | Removed `[carol]`; open; m2 delivered to bob, not to carol |
| H4 | The oracle accepts the trace, in which the expiry appears as carol's denials | E3, E4 | Accepted; the broken controls built on carol's denial are rejected |

**Control.** The same experiment with agent-trust `feat/r11-r12` (R-11 without R-12)
fails at C1: `expected read_fenced, got unauthorized`. That is the stall the theory
predicted. The release is refused because carol's chain has expired, but nothing
closes the fence, so no repair can be planned. Evidence:
`results/E10/2026-10-25-local-control-pre-r12/`.

## Limits

The run waits on wall-clock time, so it takes about a minute, and on a slow backend the
admissions must finish within carol's 45 seconds. The APF's clock is trusted. The
experiment does not test clock skew between services.
