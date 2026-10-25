# E10: Delegation expiry recorded by the APF, then removed by an ordinary repair

- Result: **FAIL**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 50.433 s; steps run: 12; steps not run: 13

Hypothesis: (H1) Before any kernel step after a reader's delegation expires, nothing is recorded and the group is not fenced. (H2) The first release attempt after the expiry is refused as fenced, and the APF records the expiry as one ledger event although that release failed. (H3) An ordinary repair then removes the expired reader and reopens the group; later messages reach the remaining readers only. Before R-12 the group stalled: every release failed and no repair could be staged. (H4) The oracle accepts the trace, in which the expiry is the reader's denials.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.792 | {"branch": "3c72af86184ac0dbb766bca9413c119510c3cf1c16c35ad8508662eedc68d8e1", "epoch": 0} |
| 1 | A2 delegate bob: read, write, admit (long-lived) | pass | 0.034 | {"revision": 1} |
| 2 | A3 delegate carol: read, write, admit for 45 s | pass | 0.034 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.804 | {"branch": "51415be31799c74317545e9507d1b00e357287a45336659a2a00e7db4528ee9d", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol | pass | 1.1 | {"branch": "b1688c84871de5ce680fded0d05f810dae3077aed1be049647b80749406981f2", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 alice sends m1 while carol is valid | pass | 0.168 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955461707} |
| 6 | A7 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 7 | A8 carol received m1 | pass | 0.0 | {"agent": "carol", "delivered": ["alice-000001"]} |
| 8 | B1 wait past carol's expiry | pass | 47.0 | {"waited": 47} |
| 9 | B2 no kernel step yet: no expiry recorded | pass | 0.002 | {"count": 0, "fence_revision": null, "kind": "expire", "last": "consume", "read_fenced": false} |
| 10 | B3 not fenced, carol still in the roster | pass | 0.001 | {"epoch": 2, "policy_revision": 2, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 11 | C1 alice's release reaches the APF kernel: refused as fenced | fail | 0.087 | expected read_fenced, got unauthorized |

## Final state

- APF roster: ['alice', 'bob', 'carol']; epoch 2; policy revision 2; read fenced False
- Relay: posted 2, fetched 2, dropped 0, duplicated 0, tapped 0
