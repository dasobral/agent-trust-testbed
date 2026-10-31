# E9: Static apf_anchor, policy in the canonical ack, no GroupContextExtensions

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 6.057 s; steps run: 28; steps not run: 0

Hypothesis: (H1) The apf_anchor fixed at genesis is the same bytes at every member after every commit kind: admission, repair and rekey. (H2) Every canonical ack that a member merges under carries the APF's policy revision and digest at the canonical step; a repair ack's revision is at or above the fence revision it answers. (H3) A repair commit that also carries a GroupContextExtensions proposal, even one that repeats the current extensions, is refused and changes nothing; the plain repair then succeeds. (H4) The oracle accepts the trace.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.676 | {"branch": "4d9e5db86af6519d1e734590b7fd40396dc723658847f76fb8217af241b6b333", "epoch": 0} |
| 1 | A2 alice holds exactly one apf_anchor; record it as the genesis anchor | pass | 0.002 | {"agents": ["alice"], "anchor": "02201c56d24360b02b1864f3fe672b29048996de300a93c4893d88c20f9177038966036c6162", "policy_revision": 0} |
| 2 | A3 delegate bob: read, write, admit | pass | 0.039 | {"revision": 1} |
| 3 | A4 delegate carol: read, write, admit | pass | 0.039 | {"revision": 2} |
| 4 | A5 admit bob | pass | 0.791 | {"branch": "6dc289232886eccb84adb429cdad4388838cfa0b7dd5a153273702c277c556ac", "epoch": 1, "index": 0, "subject": "bob"} |
| 5 | A6 admit carol | pass | 1.056 | {"branch": "ae1dc35e538b3962068770528b4284c042f8ae9e498847e853e7bdab28a423de", "epoch": 2, "index": 1, "subject": "carol"} |
| 6 | A7 every member applies the admissions | pass | 0.004 | {"synced": ["alice", "bob", "carol"]} |
| 7 | A8 after two admissions: the genesis anchor everywhere, admit acks with the APF policy | pass | 0.004 | {"agents": ["alice", "bob", "carol"], "anchor": "02201c56d24360b02b1864f3fe672b29048996de300a93c4893d88c20f9177038966036c6162", "policy_revision": 2} |
| 8 | A9 alice sends m1 | pass | 0.178 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1793481490127} |
| 9 | A10 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 10 | B1 alice revokes carol's read right: the fence closes | pass | 0.045 | {"read_fenced": true, "revision": 10} |
| 11 | B2 the APF is fenced, carol still in the roster | pass | 0.001 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 12 | B3 a repair that also proposes GroupContextExtensions (the current ones) is refused | pass | 0.096 | {"error": "noncanonical_commit: proposals", "refused": "noncanonical_commit"} |
| 13 | B4 nothing changed: still fenced, carol still in the roster | pass | 0.0 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 14 | B5 the plain repair (Removes and a path) removes carol; its ack carries the fence policy | pass | 1.039 | {"ack_policy_revision": 3, "branch": "2d5b0309fbd5694abe8e402d57c0dc75f50e91acd609ba1f87a20346bb46607a", "epoch": 3, "evidence": "pair", "fence_revision_befo... |
| 15 | B6 after the repair: the genesis anchor at alice and bob, repair acks with the APF policy | pass | 0.004 | {"agents": ["alice", "bob"], "anchor": "02201c56d24360b02b1864f3fe672b29048996de300a93c4893d88c20f9177038966036c6162", "policy_revision": 3} |
| 16 | C1 alice revokes bob's write right | pass | 0.038 | {"read_fenced": false, "revision": 13} |
| 17 | C2 alice re-grants write at generation 1: a rekey offer | pass | 0.019 | {"pending": true} |
| 18 | C3 bob commits a new signature key (an empty commit with a path) | pass | 0.701 | {"branch": "ac2dee779474b3da917ea1682699ca95341f4edd11fc160a031a9d29419ef017", "epoch": 4, "generation": 1, "public_key": "03713dd5b3c05ef314d4176d884b516ca4... |
| 19 | C4 after the rekey: the genesis anchor, update acks with the APF policy | pass | 0.004 | {"agents": ["alice", "bob"], "anchor": "02201c56d24360b02b1864f3fe672b29048996de300a93c4893d88c20f9177038966036c6162", "policy_revision": 5} |
| 20 | C5 bob sends m2 with its new key | pass | 0.177 | {"epoch": 4, "op": "bob-000001", "recipients": ["alice"], "sent_ms": 1793481492661} |
| 21 | C6 drain | pass | 0.309 | {"dropped": 0, "duplicated": 0, "fetched": 3, "held": [], "posted": 3, "queued": {}, "tapped": 0} |
| 22 | C7 alice received m2 | pass | 0.0 | {"agent": "alice", "delivered": ["bob-000001"]} |
| 23 | D1 alice and bob are on the APF frontier | pass | 0.002 | {"agents": ["alice", "bob"], "epoch": 4} |
| 24 | D2 no delivery without APF consumption | pass | 0.004 | {"consumed": 3, "deliveries": 3, "lost_after_consume": 0} |
| 25 | D3 the oracle accepts the trace | pass | 0.204 | {"events": 30, "kinds": {"admit": 2, "allocate": 2, "allow": 6, "confirm": 2, "consume": 3, "deny": 2, "emit": 2, "init": 1, "persist": 2, "recover": 3, "rek... |
| 26 | D4 the oracle rejects the broken controls that apply | pass | 0.214 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 27 | D5 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 3, fetched 3, dropped 0, duplicated 0, tapped 0
