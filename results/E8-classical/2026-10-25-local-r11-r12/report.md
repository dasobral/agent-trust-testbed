# E8-classical: Admission, revocation, repair and rekey in the classical suite

- Result: **PASS**
- Backend: `local`; topology `three-agents-classical`; entropy `qrng`
- Duration: 6.927 s; steps run: 25; steps not run: 0

Hypothesis: (H1) Every node of the deployment runs the classical suite, with keys of its size. (H2) Admission, delivery, revocation with repair and rekey behave exactly as in the other suite, and the oracle accepts the trace and rejects every broken control. (H3) The report records the size of every MLS object on the relay, for comparison.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.619 | {"branch": "bcf8d95459d4e14fcc5a855c81ef01b4b2ca91babce61e42ed591f29c3e7745d", "epoch": 0} |
| 1 | A2 delegate bob | pass | 0.036 | {"revision": 1} |
| 2 | A3 delegate carol | pass | 0.036 | {"revision": 2} |
| 3 | A4 admit bob | pass | 0.789 | {"branch": "a4a6aec6690fc3a3d89b20646d4dcbf98437a2a2c1c0ed325249e7f2f630f1dd", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 admit carol | pass | 1.12 | {"branch": "b79f7fd88f711647ec72e202bd44a4c4c50ef568b640af0f9ed29b8310a1f58c", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 the APF and every agent run the classical suite | pass | 0.001 | {"public_key_bytes": 32, "services": {"alice": "classical", "apf": "classical", "bob": "classical", "carol": "classical"}, "suite": "classical"} |
| 6 | B1 alice sends m1 | pass | 0.165 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955873708} |
| 7 | B2 bob sends m2 | pass | 0.303 | {"epoch": 2, "op": "bob-000001", "recipients": ["alice", "carol"], "sent_ms": 1792955874010} |
| 8 | B3 drain | pass | 0.511 | {"dropped": 0, "duplicated": 0, "fetched": 4, "held": [], "posted": 4, "queued": {}, "tapped": 0} |
| 9 | B4 m1 and m2 reached every reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 4} |
| 10 | C1 alice revokes carol's read right | pass | 0.047 | {"read_fenced": true, "revision": 15} |
| 11 | C2 alice commits the repair; bob confirms; the fence reopens | pass | 1.002 | {"branch": "4b8f7946ca755b146c5a9423de9186a9b0d5ae4f1562ae2e80554d2965022f55", "epoch": 3, "evidence": "pair", "index": 2, "opened": false, "read_fenced_afte... |
| 12 | C3 alice sends m3 after the repair | pass | 0.166 | {"epoch": 3, "op": "alice-000002", "recipients": ["bob"], "sent_ms": 1792955875738} |
| 13 | C4 drain | pass | 0.411 | {"dropped": 0, "duplicated": 0, "fetched": 5, "held": [], "posted": 5, "queued": {}, "tapped": 0} |
| 14 | C5 bob received m3 | pass | 0.0 | {"agent": "bob", "delivered": ["alice-000002"]} |
| 15 | D1 alice revokes bob's write right | pass | 0.048 | {"read_fenced": false, "revision": 22} |
| 16 | D2 alice re-grants write at generation 1: a rekey offer | pass | 0.022 | {"pending": true} |
| 17 | D3 bob commits a new signature key and incarnation | pass | 0.67 | {"branch": "f87e0e4e2a3a6aa690fac63cd25584ae7f2ca8be31478556f57bd66690414803", "epoch": 4, "generation": 1, "public_key": "2b7e2ad34f6d85679bd5f67bbd97b02639... |
| 18 | D4 bob sends m4 with his new key | pass | 0.171 | {"epoch": 4, "op": "bob-000002", "recipients": ["alice"], "sent_ms": 1792955877059} |
| 19 | D5 drain | pass | 0.41 | {"dropped": 0, "duplicated": 0, "fetched": 6, "held": [], "posted": 6, "queued": {}, "tapped": 0} |
| 20 | D6 alice received m4 | pass | 0.0 | {"agent": "alice", "delivered": ["bob-000002"]} |
| 21 | F1 sizes of KeyPackages, commits, acks and Welcome messages on the relay | pass | 0.001 | {"ack": {"count": 4, "max": 351, "min": 351}, "commit": {"count": 4, "max": 1130, "min": 537}, "key_package": {"count": 2, "max": 542, "min": 536}, "welcome"... |
| 22 | F2 the independent oracle accepts the exported APF trace | pass | 0.193 | {"events": 43, "kinds": {"admit": 2, "allocate": 4, "allow": 6, "confirm": 2, "consume": 6, "deny": 2, "emit": 4, "init": 1, "persist": 4, "recover": 5, "rek... |
| 23 | F3 the oracle rejects every broken control derived from the trace | pass | 0.203 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 24 | F4 the honest agents logged no processing error | pass | 0.001 | {"agents": ["alice", "bob"]} |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 6, fetched 6, dropped 0, duplicated 0, tapped 0
