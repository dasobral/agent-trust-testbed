# E2: QRNG outage over the network: fail closed, change nothing, recover

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 6.59 s; steps run: 34; steps not run: 0

Hypothesis: (H1) While the QRNG service refuses requests, every step that needs fresh randomness on an agent (send, add, rekey) is refused with entropy_unavailable and nothing falls back to another source. (H2) A refused step changes neither the frontier, the roster nor the registered keys; a released but unprotected operation is abandoned at the APF (barrier B2). (H3) A node cannot start during the outage. (H4) When the source returns, every refused step succeeds without operator repair, and the oracle accepts the whole trace.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.685 | {"branch": "6c1bbac574193daf51283b8c450920d9b9f32c5c2030ac9565515c8d8edecdcb", "epoch": 0} |
| 1 | A2 delegate bob | pass | 0.039 | {"revision": 1} |
| 2 | A3 delegate carol | pass | 0.039 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.811 | {"branch": "77120a80408bb14b1d4b0f97286d56213019dfed90c063ea4a336dd3eedd379e", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 carol publishes a certified KeyPackage (not yet added) | pass | 0.05 | {"published": true, "rights": [{"generation": 0, "right": "read"}, {"generation": 0, "right": "write"}, {"generation": 0, "right": "admit"}], "subject": "car... |
| 5 | A6 alice sends m1 | pass | 0.185 | {"epoch": 1, "op": "alice-000001", "recipients": ["bob"], "sent_ms": 1793481430958} |
| 6 | A7 drain | pass | 0.409 | {"dropped": 0, "duplicated": 0, "fetched": 1, "held": [], "posted": 1, "queued": {}, "tapped": 0} |
| 7 | B1 the QRNG service starts refusing every request | pass | 0.0 | {"available": false} |
| 8 | B2 alice cannot send: the MLS message needs fresh randomness | pass | 0.136 | {"error": "entropy_unavailable", "refused": "entropy_unavailable"} |
| 9 | B3 alice cannot add carol: the commit needs fresh path secrets | pass | 0.002 | {"error": "entropy_unavailable", "refused": "entropy_unavailable"} |
| 10 | B4 alice revokes bob's write (the APF draws nothing) | pass | 0.039 | {"read_fenced": false, "revision": 11} |
| 11 | B5 alice offers write at generation 1 | pass | 0.02 | {"pending": true} |
| 12 | B6 bob cannot rekey: the new key needs fresh randomness | pass | 0.001 | {"error": "entropy_unavailable", "refused": "entropy_unavailable"} |
| 13 | B7 nothing changed: roster and epoch | pass | 0.0 | {"epoch": 1, "policy_revision": 3, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob"]} |
| 14 | B8 alice and bob are still on the APF frontier | pass | 0.001 | {"agents": ["alice", "bob"], "epoch": 1} |
| 15 | B9 bob stops | pass | 0.051 | {"stopped": "bob"} |
| 16 | B10 bob cannot start without its entropy source (fail closed) | pass | 0.101 | {"reason": "bob exited with status 1; see its log", "service": "bob", "started": false} |
| 17 | C1 the QRNG service recovers | pass | 0.0 | {"available": true} |
| 18 | C2 bob starts and resumes from its data directory | pass | 0.102 | {"service": "bob", "started": true} |
| 19 | C3 bob is the same member on the frontier | pass | 0.001 | {"epoch": 1, "future": 0, "membership": "member", "restarts": 1, "skipped_entries": 0, "waiting": []} |
| 20 | C4 alice sends m2 | pass | 0.17 | {"epoch": 1, "op": "alice-000003", "recipients": ["bob"], "sent_ms": 1793481431991} |
| 21 | C4b drain before the epoch changes (finding F-7: consume binds to the APF's current epoch) | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 22 | C5 alice adds carol with the same KeyPackage | pass | 1.047 | {"branch": "62ad609c00d147737cf39ca878b89b4c23cafb716dfad38bf44b0d1e952f0865", "epoch": 2, "index": 1, "subject": "carol"} |
| 23 | C6 bob completes the pending rekey | pass | 0.997 | {"branch": "27901408d9c5d98034043a059e8ea82f0ed5d3b6770c042b44cf69f149fe7548", "epoch": 3, "generation": 1, "public_key": "8cf4f68faf279f459d8b46255f87eae2ea... |
| 24 | C7 bob sends m3 with its new key | pass | 0.17 | {"epoch": 3, "op": "bob-000001", "recipients": ["alice", "carol"], "sent_ms": 1793481434616} |
| 25 | C8 carol sends m4 | pass | 0.3 | {"epoch": 3, "op": "carol-000001", "recipients": ["alice", "bob"], "sent_ms": 1793481434917} |
| 26 | C9 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 6, "held": [], "posted": 6, "queued": {}, "tapped": 0} |
| 27 | D1 final roster and epoch: add plus rekey | pass | 0.001 | {"epoch": 3, "policy_revision": 4, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 28 | D2 every agent is on the APF frontier | pass | 0.001 | {"agents": ["alice", "bob", "carol"], "epoch": 3} |
| 29 | D3 every message reached every certified reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 6} |
| 30 | D4 no delivery without APF consumption | pass | 0.006 | {"consumed": 6, "deliveries": 6, "lost_after_consume": 0} |
| 31 | D5 the oracle accepts the trace, the abandoned release included | pass | 0.205 | {"events": 40, "kinds": {"abandon": 1, "admit": 2, "allocate": 5, "allow": 6, "consume": 6, "deny": 1, "emit": 4, "init": 1, "persist": 4, "recover": 4, "rek... |
| 32 | D6 the oracle rejects the broken controls that apply to this trace | pass | 0.196 | {"rejected": {"rekey that reuses another member's key": "stale_key", "release cover that omits a reader": "unsound_cover", "replayed consumption": "replay", ... |
| 33 | D7 alice's counter shows the refused draws (bob's counter restarted with bob; B6 asserts its refusal); nothing else failed | pass | 0.002 | {"nodes": {"alice": {"bytes": 440, "failures": 2, "key_exchange": "none (plain HTTP)", "post_quantum": false, "requests": 12, "source": "qrng"}, "apf": {"byt... |

## Final state

- APF roster: ['alice', 'bob', 'carol']; epoch 3; policy revision 4; read fenced False
- Relay: posted 6, fetched 6, dropped 0, duplicated 0, tapped 0
