# E8-pq: Admission, revocation, repair and rekey in the pq suite

- Result: **PASS**
- Backend: `local`; topology `three-agents`; entropy `qrng`
- Duration: 6.978 s; steps run: 25; steps not run: 0

Hypothesis: (H1) Every node of the deployment runs the pq suite, with keys of its size. (H2) Admission, delivery, revocation with repair and rekey behave exactly as in the other suite, and the oracle accepts the trace and rejects every broken control. (H3) The report records the size of every MLS object on the relay, for comparison.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.655 | {"branch": "6b0e82fa7d63bb3738bae058193087bdae03d675ea40c92f428ef237be58e8db", "epoch": 0} |
| 1 | A2 delegate bob | pass | 0.042 | {"revision": 1} |
| 2 | A3 delegate carol | pass | 0.037 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.844 | {"branch": "ef33c78b4761ddee1bb46a43b9b2fa9d44afa79b4a223fa0036b3860287fb26b", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol | pass | 1.074 | {"branch": "dac9b85b879b63135e7b5b3336cb9353b0b5b1d35fe59d29d689167d0db7cff7", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 the APF and every agent run the pq suite | pass | 0.001 | {"public_key_bytes": 1312, "services": {"alice": "pq", "apf": "pq", "bob": "pq", "carol": "pq"}, "suite": "pq"} |
| 6 | B1 alice sends m1 | pass | 0.166 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1793481472824} |
| 7 | B2 bob sends m2 | pass | 0.301 | {"epoch": 2, "op": "bob-000001", "recipients": ["alice", "carol"], "sent_ms": 1793481473125} |
| 8 | B3 drain | pass | 0.409 | {"dropped": 0, "duplicated": 0, "fetched": 4, "held": [], "posted": 4, "queued": {}, "tapped": 0} |
| 9 | B4 m1 and m2 reached every reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 4} |
| 10 | C1 alice revokes carol's read right | pass | 0.046 | {"read_fenced": true, "revision": 15} |
| 11 | C2 alice commits the repair; bob confirms; the fence reopens | pass | 1.076 | {"branch": "1f9dfd824c9ef0f12dcd6bdb0a74a4528d057891a22859c95be28268fcba7f51", "epoch": 3, "evidence": "pair", "index": 2, "opened": false, "read_fenced_afte... |
| 12 | C3 alice sends m3 after the repair | pass | 0.165 | {"epoch": 3, "op": "alice-000002", "recipients": ["bob"], "sent_ms": 1793481474822} |
| 13 | C4 drain | pass | 0.409 | {"dropped": 0, "duplicated": 0, "fetched": 5, "held": [], "posted": 5, "queued": {}, "tapped": 0} |
| 14 | C5 bob received m3 | pass | 0.0 | {"agent": "bob", "delivered": ["alice-000002"]} |
| 15 | D1 alice revokes bob's write right | pass | 0.036 | {"read_fenced": false, "revision": 22} |
| 16 | D2 alice re-grants write at generation 1: a rekey offer | pass | 0.019 | {"pending": true} |
| 17 | D3 bob commits a new signature key and incarnation | pass | 0.7 | {"branch": "52c4767d89a58a2517240d6b74a669777b67c6cc2fe960e566d91c0309724965", "epoch": 4, "generation": 1, "public_key": "38ac10cd5b278fe8c50a60902a49fdaab9... |
| 18 | D4 bob sends m4 with his new key | pass | 0.171 | {"epoch": 4, "op": "bob-000002", "recipients": ["alice"], "sent_ms": 1793481476159} |
| 19 | D5 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 6, "held": [], "posted": 6, "queued": {}, "tapped": 0} |
| 20 | D6 alice received m4 | pass | 0.0 | {"agent": "alice", "delivered": ["bob-000002"]} |
| 21 | F1 sizes of KeyPackages, commits, acks and Welcome messages on the relay | pass | 0.002 | {"ack": {"count": 4, "max": 2707, "min": 2707}, "commit": {"count": 4, "max": 22551, "min": 10054}, "key_package": {"count": 2, "max": 11261, "min": 11255}, ... |
| 22 | F2 the independent oracle accepts the exported APF trace | pass | 0.205 | {"events": 43, "kinds": {"admit": 2, "allocate": 4, "allow": 6, "confirm": 2, "consume": 6, "deny": 2, "emit": 4, "init": 1, "persist": 4, "recover": 5, "rek... |
| 23 | F3 the oracle rejects every broken control derived from the trace | pass | 0.206 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 24 | F4 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 6, fetched 6, dropped 0, duplicated 0, tapped 0
