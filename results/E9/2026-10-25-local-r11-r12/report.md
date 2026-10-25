# E9: Static apf_anchor, policy in the canonical ack, no GroupContextExtensions

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 6.034 s; steps run: 28; steps not run: 0

Hypothesis: (H1) The apf_anchor fixed at genesis is the same bytes at every member after every commit kind: admission, repair and rekey. (H2) Every canonical ack that a member merges under carries the APF's policy revision and digest at the canonical step; a repair ack's revision is at or above the fence revision it answers. (H3) A repair commit that also carries a GroupContextExtensions proposal, even one that repeats the current extensions, is refused and changes nothing; the plain repair then succeeds. (H4) The oracle accepts the trace.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.64 | {"branch": "b3f3466b730aaf4522b13b83cd737b4e0aa4b8864f1e5063d3ff4965a50270c4", "epoch": 0} |
| 1 | A2 alice holds exactly one apf_anchor; record it as the genesis anchor | pass | 0.002 | {"agents": ["alice"], "anchor": "022021795161cc98715f46b83a5208da1b67054a23efc922838d9d98096842fec44e036c6162", "policy_revision": 0} |
| 2 | A3 delegate bob: read, write, admit | pass | 0.037 | {"revision": 1} |
| 3 | A4 delegate carol: read, write, admit | pass | 0.037 | {"revision": 2} |
| 4 | A5 admit bob | pass | 0.811 | {"branch": "2d9caaf073eb51933e3bd46818a2be63628bfcfbd782684525311004eefec743", "epoch": 1, "index": 0, "subject": "bob"} |
| 5 | A6 admit carol | pass | 1.051 | {"branch": "761b4cbb5411f999ade1964ee4d7157b671dcf4365181178fd6330ffb1b77df1", "epoch": 2, "index": 1, "subject": "carol"} |
| 6 | A7 every member applies the admissions | pass | 0.004 | {"synced": ["alice", "bob", "carol"]} |
| 7 | A8 after two admissions: the genesis anchor everywhere, admit acks with the APF policy | pass | 0.004 | {"agents": ["alice", "bob", "carol"], "anchor": "022021795161cc98715f46b83a5208da1b67054a23efc922838d9d98096842fec44e036c6162", "policy_revision": 2} |
| 8 | A9 alice sends m1 | pass | 0.166 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955882305} |
| 9 | A10 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 10 | B1 alice revokes carol's read right: the fence closes | pass | 0.046 | {"read_fenced": true, "revision": 10} |
| 11 | B2 the APF is fenced, carol still in the roster | pass | 0.001 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 12 | B3 a repair that also proposes GroupContextExtensions (the current ones) is refused | pass | 0.094 | {"error": "noncanonical_commit: proposals", "refused": "noncanonical_commit"} |
| 13 | B4 nothing changed: still fenced, carol still in the roster | pass | 0.001 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 14 | B5 the plain repair (Removes and a path) removes carol; its ack carries the fence policy | pass | 1.004 | {"ack_policy_revision": 3, "branch": "0eb58848a1e6bc523c319409288c1981f8caf7567570067c72edc2f128348716", "epoch": 3, "evidence": "pair", "fence_revision_befo... |
| 15 | B6 after the repair: the genesis anchor at alice and bob, repair acks with the APF policy | pass | 0.003 | {"agents": ["alice", "bob"], "anchor": "022021795161cc98715f46b83a5208da1b67054a23efc922838d9d98096842fec44e036c6162", "policy_revision": 3} |
| 16 | C1 alice revokes bob's write right | pass | 0.038 | {"read_fenced": false, "revision": 13} |
| 17 | C2 alice re-grants write at generation 1: a rekey offer | pass | 0.02 | {"pending": true} |
| 18 | C3 bob commits a new signature key (an empty commit with a path) | pass | 0.683 | {"branch": "052595ed7539b0aae0fbddd5b69e74b9aa75b22fceb6507f097983e9e9f33096", "epoch": 4, "generation": 1, "public_key": "c1817a950608b3fe6c7e2b7680ed05d688... |
| 19 | C4 after the rekey: the genesis anchor, update acks with the APF policy | pass | 0.004 | {"agents": ["alice", "bob"], "anchor": "022021795161cc98715f46b83a5208da1b67054a23efc922838d9d98096842fec44e036c6162", "policy_revision": 5} |
| 20 | C5 bob sends m2 with its new key | pass | 0.171 | {"epoch": 4, "op": "bob-000001", "recipients": ["alice"], "sent_ms": 1792955884779} |
| 21 | C6 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 3, "held": [], "posted": 3, "queued": {}, "tapped": 0} |
| 22 | C7 alice received m2 | pass | 0.0 | {"agent": "alice", "delivered": ["bob-000001"]} |
| 23 | D1 alice and bob are on the APF frontier | pass | 0.001 | {"agents": ["alice", "bob"], "epoch": 4} |
| 24 | D2 no delivery without APF consumption | pass | 0.004 | {"consumed": 3, "deliveries": 3, "lost_after_consume": 0} |
| 25 | D3 the oracle accepts the trace | pass | 0.195 | {"events": 30, "kinds": {"admit": 2, "allocate": 2, "allow": 6, "confirm": 2, "consume": 3, "deny": 2, "emit": 2, "init": 1, "persist": 2, "recover": 3, "rek... |
| 26 | D4 the oracle rejects the broken controls that apply | pass | 0.196 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 27 | D5 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 3, fetched 3, dropped 0, duplicated 0, tapped 0
