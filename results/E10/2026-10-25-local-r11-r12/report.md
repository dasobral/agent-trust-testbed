# E10: Delegation expiry recorded by the APF, then removed by an ordinary repair

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 52.162 s; steps run: 25; steps not run: 0

Hypothesis: (H1) Before any kernel step after a reader's delegation expires, nothing is recorded and the group is not fenced. (H2) The first release attempt after the expiry is refused as fenced, and the APF records the expiry as one ledger event although that release failed. (H3) An ordinary repair then removes the expired reader and reopens the group; later messages reach the remaining readers only. Before R-12 the group stalled: every release failed and no repair could be staged. (H4) The oracle accepts the trace, in which the expiry is the reader's denials.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.651 | {"branch": "5354181425534d96aa44d7570ff47c54e86c0fd4c772dfdc738dbdbd866b1e59", "epoch": 0} |
| 1 | A2 delegate bob: read, write, admit (long-lived) | pass | 0.037 | {"revision": 1} |
| 2 | A3 delegate carol: read, write, admit for 45 s | pass | 0.037 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.792 | {"branch": "237b658e0a48b2db4019bc281da8abfb07c220f268498b87db4ed0425f5ba416", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol | pass | 1.072 | {"branch": "a36a59a7f0be908dd7601bab7e94b282b2d71021c59ddb923814972674eea184", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 alice sends m1 while carol is valid | pass | 0.166 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955890029} |
| 6 | A7 drain | pass | 0.411 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 7 | A8 carol received m1 | pass | 0.0 | {"agent": "carol", "delivered": ["alice-000001"]} |
| 8 | B1 wait past carol's expiry | pass | 47.0 | {"waited": 47} |
| 9 | B2 no kernel step yet: no expiry recorded | pass | 0.002 | {"count": 0, "fence_revision": null, "kind": "expire", "last": "consume", "read_fenced": false} |
| 10 | B3 not fenced, carol still in the roster | pass | 0.001 | {"epoch": 2, "policy_revision": 2, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 11 | C1 alice's release reaches the APF kernel: refused as fenced | pass | 0.095 | {"error": "read_fenced", "refused": "read_fenced"} |
| 12 | C2 the APF recorded one expire event, kept although the release failed | pass | 0.002 | {"count": 1, "fence_revision": 3, "kind": "expire", "last": "expire", "read_fenced": true} |
| 13 | C3 fenced, carol still in the roster | pass | 0.0 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 14 | D1 alice's ordinary repair removes carol and reopens the group | pass | 0.977 | {"ack_policy_revision": 3, "branch": "9ee6cf98982b4a8d058973099302f6bb9ae924bb895ab5c244692b844606caf6", "epoch": 3, "evidence": "pair", "fence_revision_befo... |
| 15 | D2 open again, carol removed | pass | 0.0 | {"epoch": 3, "policy_revision": 3, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob"]} |
| 16 | D3 alice sends m2 | pass | 0.172 | {"epoch": 3, "op": "alice-000003", "recipients": ["bob"], "sent_ms": 1792955938690} |
| 17 | D4 drain | pass | 0.31 | {"dropped": 0, "duplicated": 0, "fetched": 3, "held": [], "posted": 3, "queued": {}, "tapped": 0} |
| 18 | D5 bob received m2 | pass | 0.0 | {"agent": "bob", "delivered": ["alice-000003"]} |
| 19 | D6 carol did not receive m2 | pass | 0.0 | {"agent": "carol", "not_delivered": ["alice-000003"]} |
| 20 | E1 alice and bob are on the APF frontier | pass | 0.001 | {"agents": ["alice", "bob"], "epoch": 3} |
| 21 | E2 no delivery without APF consumption | pass | 0.004 | {"consumed": 3, "deliveries": 3, "lost_after_consume": 0} |
| 22 | E3 the oracle accepts the trace | pass | 0.23 | {"events": 31, "kinds": {"admit": 2, "allocate": 3, "allow": 6, "confirm": 2, "consume": 3, "deny": 3, "emit": 2, "init": 1, "persist": 2, "recover": 3, "rel... |
| 23 | E4 the oracle rejects the broken controls built on carol's denial | pass | 0.198 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "release after the read de... |
| 24 | E5 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 3; policy revision 3; read fenced False
- Relay: posted 3, fetched 3, dropped 0, duplicated 0, tapped 0
