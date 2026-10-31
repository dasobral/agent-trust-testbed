# E4: Hostile commit distribution: forged, replayed, spliced, hidden and reordered commits

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 8.607 s; steps run: 48; steps not run: 0

Hypothesis: (H1) A member never merges a commit without a valid APF ack whose parent is its own frontier: replayed, garbage and spliced handshake entries are skipped and change nothing. (H2) A member from which the relay hides a commit stays at its old epoch: it cannot release, and envelopes of the newer epoch wait instead of being decrypted or lost; once the commit appears it catches up and delivers them. (H3) Entries served out of order are applied in parent order. (H4) The trace stays acceptable to the oracle.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.663 | {"branch": "ac624468c066a35d08d87a0bb755d946bfa058d37ea12860053bddc655cea961", "epoch": 0} |
| 1 | A2 delegate bob | pass | 0.037 | {"revision": 1} |
| 2 | A3 delegate carol | pass | 0.038 | {"revision": 2} |
| 3 | A4 admit bob (log index 0) | pass | 0.773 | {"branch": "b34f09288566b84f053cfd1d9aedbe03035dc3b7eef7ebb1dc0dfacc010f21b0", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol (log index 1) | pass | 1.025 | {"branch": "7f3740876e177b9643a527535c6c1111e20c6db716c9325cde367ef9231e0296", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | B1 the relay replays bob's admission commit (index 2) | pass | 0.0 | {"action": "replay", "index": 2} |
| 6 | B2 the relay injects an entry with a garbage ack (index 3) | pass | 0.0 | {"action": "inject", "index": 3} |
| 7 | B3 the relay splices carol's commit with bob's ack (index 4) | pass | 0.0 | {"action": "splice", "index": 4} |
| 8 | B4 every member processes the forged entries | pass | 0.035 | {"synced": ["alice", "bob", "carol"]} |
| 9 | B5 every member is still on the APF frontier | pass | 0.002 | {"agents": ["alice", "bob", "carol"], "epoch": 2} |
| 10 | B6 bob skipped the three forged entries | pass | 0.0 | {"epoch": 2, "future": 0, "membership": "member", "restarts": 0, "skipped_entries": 3, "waiting": []} |
| 11 | B7 alice sends m1: the group still works | pass | 0.166 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1793481454121} |
| 12 | B8 drain | pass | 0.51 | {"dropped": 0, "duplicated": 0, "fetched": 2, "held": [], "posted": 2, "queued": {}, "tapped": 0} |
| 13 | C1 the relay hides log index 5 from bob | pass | 0.0 | {"hidden_from": null, "indices": [5], "reader": "bob"} |
| 14 | C2 alice revokes carol's write | pass | 0.038 | {"read_fenced": false, "revision": 10} |
| 15 | C3 alice offers carol write at generation 1 | pass | 0.02 | {"pending": true} |
| 16 | C4 carol commits its rekey (log index 5, hidden from bob) | pass | 0.677 | {"branch": "eac82254052dcca2e824cf4510a663015fc5badfb0bc8e55ea2e2d8fb0704c5a", "epoch": 3, "generation": 1, "public_key": "e66a9a5df5a572f937931382566d8ff7ff... |
| 17 | C5 the relay splices the old admission commit with the new rekey ack (index 6) | pass | 0.0 | {"action": "splice", "index": 6} |
| 18 | C5b every member processes the log | pass | 0.033 | {"synced": ["alice", "bob", "carol"]} |
| 19 | C6 bob is stuck at the old epoch and refused the splice | pass | 0.001 | {"epoch": 2, "future": 0, "membership": "member", "restarts": 0, "skipped_entries": 4, "waiting": []} |
| 20 | C7 bob cannot release from a stale frontier | pass | 0.047 | {"error": "not_ready: the local frontier differs from the APF frontier", "refused": "not_ready"} |
| 21 | C8 alice sends m2 at the new epoch | pass | 0.167 | {"epoch": 3, "op": "alice-000002", "recipients": ["bob", "carol"], "sent_ms": 1793481455614} |
| 22 | C9 carol's copy drains; bob's copy waits for its commit | pass | 0.412 | {"dropped": 0, "duplicated": 0, "fetched": 4, "held": [], "posted": 4, "queued": {}, "tapped": 0} |
| 23 | C10 bob has not delivered m2 yet | pass | 0.0 | {"agent": "bob", "not_delivered": ["alice-000002"]} |
| 24 | C11 the relay shows index 5 to bob again | pass | 0.0 | {"hidden_from": null, "reader": "bob"} |
| 25 | C12 bob catches up and delivers the waiting envelope | pass | 0.509 | {"dropped": 0, "duplicated": 0, "fetched": 4, "held": [], "posted": 4, "queued": {}, "tapped": 0} |
| 26 | C13 bob is back on the APF frontier | pass | 0.001 | {"epoch": 3, "future": 0, "membership": "member", "restarts": 0, "skipped_entries": 4, "waiting": []} |
| 27 | C14 bob received m2 once: delay, not loss | pass | 0.0 | {"agent": "bob", "delivered": ["alice-000002"]} |
| 28 | D1 the relay hides everything from index 7 on from carol | pass | 0.0 | {"hidden_from": 7, "indices": [], "reader": "carol"} |
| 29 | D2 alice revokes bob's write | pass | 0.037 | {"read_fenced": false, "revision": 18} |
| 30 | D3 offer write at generation 1 | pass | 0.019 | {"pending": true} |
| 31 | D4 bob rekeys write (log index 7) | pass | 0.676 | {"branch": "db7a1ffe79ef72cd0fd35a16df2185ea8219c33a21a4b583431f31a6ea3d8225", "epoch": 4, "generation": 1, "public_key": "ad7dff34ab7f47073b7b363a472cbcac0f... |
| 32 | D5 alice revokes bob's admit | pass | 0.039 | {"read_fenced": false, "revision": 20} |
| 33 | D6 offer admit at generation 1 | pass | 0.019 | {"pending": true} |
| 34 | D7 bob rekeys admit (log index 8) | pass | 0.676 | {"branch": "08aa2a6b379710921d500964e1e802a56e82a776c0113df0a3ec9677862c88a9", "epoch": 5, "generation": 1, "public_key": "336c1c9d34a7298dd718784f7a9f2e65d0... |
| 35 | D8 the relay will serve carol's entries in reverse order | pass | 0.0 | {"reader": "carol", "reversed": true} |
| 36 | D9 the relay shows carol both commits at once | pass | 0.0 | {"hidden_from": null, "reader": "carol"} |
| 37 | D10 carol processes the reversed batch | pass | 0.589 | {"synced": ["alice", "bob", "carol"]} |
| 38 | D11 every member is on the APF frontier | pass | 0.001 | {"agents": ["alice", "bob", "carol"], "epoch": 5} |
| 39 | D12 carol sends m3 | pass | 0.167 | {"epoch": 5, "op": "carol-000001", "recipients": ["alice", "bob"], "sent_ms": 1793481458761} |
| 40 | D13 bob sends m4 with its newest key | pass | 0.302 | {"epoch": 5, "op": "bob-000002", "recipients": ["alice", "carol"], "sent_ms": 1793481459063} |
| 41 | D14 drain | pass | 0.51 | {"dropped": 0, "duplicated": 0, "fetched": 8, "held": [], "posted": 8, "queued": {}, "tapped": 0} |
| 42 | E1 roster and epoch: two admissions and three rekeys | pass | 0.001 | {"epoch": 5, "policy_revision": 8, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 43 | E2 every message reached every certified reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 8} |
| 44 | E3 no delivery without APF consumption | pass | 0.008 | {"consumed": 8, "deliveries": 8, "lost_after_consume": 0} |
| 45 | E4 the oracle accepts the trace: no forged entry reached the APF | pass | 0.205 | {"events": 47, "kinds": {"admit": 2, "allocate": 5, "allow": 6, "consume": 8, "deny": 3, "emit": 4, "init": 1, "persist": 4, "recover": 7, "rekey": 3, "relea... |
| 46 | E5 the oracle rejects the broken controls that apply | pass | 0.2 | {"rejected": {"rekey that reuses another member's key": "stale_key", "release cover that omits a reader": "unsound_cover", "replayed consumption": "replay", ... |
| 47 | E6 the only errors any agent logged are the forged entries 3 and 6 | pass | 0.001 | {"forged_indices_logged": {"alice": [3], "bob": [3, 6], "carol": [3]}} |

## Final state

- APF roster: ['alice', 'bob', 'carol']; epoch 5; policy revision 8; read fenced False
- Relay: posted 8, fetched 8, dropped 0, duplicated 0, tapped 0
