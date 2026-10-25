# E8-pq: Admission, revocation, repair and rekey in the pq suite

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 6.676 s; steps run: 25; steps not run: 0

Hypothesis: (H1) Every node of the deployment runs the pq suite, with keys of its size. (H2) Admission, delivery, revocation with repair and rekey behave exactly as in the other suite, and the oracle accepts the trace and rejects every broken control. (H3) The report records the size of every MLS object on the relay, for comparison.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.622 | {"branch": "5e5e22905f533fd016baae068e3427ff71e4dcb4ea08ca055faf2e15b46869d3", "epoch": 0} |
| 1 | A2 delegate bob | pass | 0.037 | {"revision": 1} |
| 2 | A3 delegate carol | pass | 0.037 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.816 | {"branch": "99a5ce38ce52a6274efdebf6b2f0b19a8347287775eebc9df65fef6a06fa7655", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol | pass | 1.072 | {"branch": "7d1fd22bef32869613011290ba3e5bd3cc3a66f3c98033fdd2a7d5d902c6eeff", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 the APF and every agent run the pq suite | pass | 0.001 | {"public_key_bytes": 1312, "services": {"alice": "pq", "apf": "pq", "bob": "pq", "carol": "pq"}, "suite": "pq"} |
| 6 | B1 alice sends m1 | pass | 0.164 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955865328} |
| 7 | B2 bob sends m2 | pass | 0.302 | {"epoch": 2, "op": "bob-000001", "recipients": ["alice", "carol"], "sent_ms": 1792955865630} |
| 8 | B3 drain | pass | 0.411 | {"dropped": 0, "duplicated": 0, "fetched": 4, "held": [], "posted": 4, "queued": {}, "tapped": 0} |
| 9 | B4 m1 and m2 reached every reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 4} |
| 10 | C1 alice revokes carol's read right | pass | 0.047 | {"read_fenced": true, "revision": 15} |
| 11 | C2 alice commits the repair; bob confirms; the fence reopens | pass | 1.042 | {"branch": "168f227a7187f9cee82a3b745192b9ba58c5fd576928ba98c48fa6f5218359e2", "epoch": 3, "evidence": "pair", "index": 2, "opened": false, "read_fenced_afte... |
| 12 | C3 alice sends m3 after the repair | pass | 0.173 | {"epoch": 3, "op": "alice-000002", "recipients": ["bob"], "sent_ms": 1792955867305} |
| 13 | C4 drain | pass | 0.311 | {"dropped": 0, "duplicated": 0, "fetched": 5, "held": [], "posted": 5, "queued": {}, "tapped": 0} |
| 14 | C5 bob received m3 | pass | 0.0 | {"agent": "bob", "delivered": ["alice-000002"]} |
| 15 | D1 alice revokes bob's write right | pass | 0.049 | {"read_fenced": false, "revision": 22} |
| 16 | D2 alice re-grants write at generation 1: a rekey offer | pass | 0.024 | {"pending": true} |
| 17 | D3 bob commits a new signature key and incarnation | pass | 0.676 | {"branch": "9001de9ba6f3585c18ae6112bb52d6829725276756266112d98554f782d61af7", "epoch": 4, "generation": 1, "public_key": "fab54aab3cfcae6c2870350df9394f5669... |
| 18 | D4 bob sends m4 with his new key | pass | 0.179 | {"epoch": 4, "op": "bob-000002", "recipients": ["alice"], "sent_ms": 1792955868543} |
| 19 | D5 drain | pass | 0.31 | {"dropped": 0, "duplicated": 0, "fetched": 6, "held": [], "posted": 6, "queued": {}, "tapped": 0} |
| 20 | D6 alice received m4 | pass | 0.0 | {"agent": "alice", "delivered": ["bob-000002"]} |
| 21 | F1 sizes of KeyPackages, commits, acks and Welcome messages on the relay | pass | 0.002 | {"ack": {"count": 4, "max": 2707, "min": 2707}, "commit": {"count": 4, "max": 22551, "min": 10054}, "key_package": {"count": 2, "max": 11261, "min": 11255}, ... |
| 22 | F2 the independent oracle accepts the exported APF trace | pass | 0.196 | {"events": 43, "kinds": {"admit": 2, "allocate": 4, "allow": 6, "confirm": 2, "consume": 6, "deny": 2, "emit": 4, "init": 1, "persist": 4, "recover": 5, "rek... |
| 23 | F3 the oracle rejects every broken control derived from the trace | pass | 0.204 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 24 | F4 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 6, fetched 6, dropped 0, duplicated 0, tapped 0
