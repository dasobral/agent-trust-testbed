# E1-mtls: Revocation fence, repair and rekey across a faulty network

- Result: **PASS**
- Backend: `local`; topology `three-agents-mtls`; entropy `qrng`
- Duration: 15.79 s; steps run: 57; steps not run: 0

Hypothesis: With the APF, the agents and the relay as separate network services, and the relay dropping, duplicating and reordering traffic: (H1) every authorized message reaches each current reader exactly once and only after an APF consumption; (H2) a read revocation stops new releases and in-flight deliveries until a repair commit removes the reader; (H3) a removed member that keeps its keys can neither decrypt nor consume later traffic; (H4) a re-granted write right needs fresh keys; (H5) the independent oracle accepts the exported trace and rejects every broken control.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group with the APF | pass | 0.625 | {"branch": "138c56f29cd056127373bfd3b8fee8f99d5b609a0f4d271a4c80d1f94b3932de", "epoch": 0} |
| 1 | A2 alice delegates read, write, admit to bob | pass | 0.04 | {"revision": 1} |
| 2 | A3 alice delegates read, write, admit to carol | pass | 0.039 | {"revision": 2} |
| 3 | A4 bob joins through a certified KeyPackage | pass | 0.794 | {"branch": "cc2631e382c749dc5d44cba75f1bfd4b2fce196975b8197cb26c1dc76822dc62", "epoch": 1, "index": 0, "subject": "bob"} |
| 4 | A5 carol joins through a certified KeyPackage | pass | 1.127 | {"branch": "f8271534434e839973911469167391fd9ec61d0c941544718f9f5500bffbd0c8", "epoch": 2, "index": 1, "subject": "carol"} |
| 5 | A6 APF roster and frontier after admission | pass | 0.002 | {"epoch": 2, "policy_revision": 2, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 6 | A7 every agent is on the APF frontier | pass | 0.008 | {"agents": ["alice", "bob", "carol"], "epoch": 2} |
| 7 | B1 relay drops 25%, duplicates 25%, reorders | pass | 0.002 | {"drop": 25, "duplicate": 25, "reorder": true, "seed": 11} |
| 8 | B2 alice sends m1 | pass | 0.171 | {"epoch": 2, "op": "alice-000001", "recipients": ["bob", "carol"], "sent_ms": 1792955805066} |
| 9 | B3 bob sends m2 | pass | 0.244 | {"epoch": 2, "op": "bob-000001", "recipients": ["alice", "carol"], "sent_ms": 1792955805310} |
| 10 | B4 carol sends m3 | pass | 0.366 | {"epoch": 2, "op": "carol-000001", "recipients": ["alice", "bob"], "sent_ms": 1792955805676} |
| 11 | B5 burst: 4 interleaved messages from each agent | pass | 4.493 | {"seconds": 4.493, "sends_per_second": 2.7, "sent": 12} |
| 12 | B5b relay drains (retransmissions and duplicates included) | pass | 0.557 | {"dropped": 13, "duplicated": 4, "fetched": 34, "held": [], "posted": 30, "queued": {}, "tapped": 0} |
| 13 | B6 alice received m2 and m3 exactly once | pass | 0.002 | {"agent": "alice", "delivered": ["bob-000001", "carol-000001"]} |
| 14 | B7 bob received m1 and m3 exactly once | pass | 0.002 | {"agent": "bob", "delivered": ["alice-000001", "carol-000001"]} |
| 15 | B8 carol received m1 and m2 exactly once | pass | 0.002 | {"agent": "carol", "delivered": ["alice-000001", "bob-000001"]} |
| 16 | B9 every burst message reached every reader exactly once | pass | 0.006 | {"excluded": [], "optional": 0, "pairs": 30} |
| 17 | B10 every rejected copy was a duplicate of a delivered message | pass | 0.011 | {"alice": {}, "bob": {"undecryptable": 3}, "carol": {"undecryptable": 1}} |
| 18 | B11 no duplicate delivery, no delivery without APF consumption | pass | 0.062 | {"consumed": 30, "deliveries": 30, "lost_after_consume": 0} |
| 19 | C1 relay holds carol's mailbox (message in flight) | pass | 0.003 | {"held": true, "mailbox": "carol"} |
| 20 | C2 alice sends m4 before the revocation | pass | 0.319 | {"epoch": 2, "op": "alice-000006", "recipients": ["bob", "carol"], "sent_ms": 1792955811133} |
| 21 | C3 bob's copy drains; carol's copy stays in flight | pass | 0.558 | {"dropped": 13, "duplicated": 4, "fetched": 35, "held": ["carol"], "posted": 32, "queued": {"carol": 1}, "tapped": 0} |
| 22 | C4 alice revokes carol's read right | pass | 0.104 | {"read_fenced": true, "revision": 84} |
| 23 | C5 the APF read fence is closed | pass | 0.003 | {"epoch": 2, "policy_revision": 3, "read_fenced": true, "repair_pending": null, "roster": ["alice", "bob", "carol"]} |
| 24 | C6 alice cannot release while fenced | pass | 0.179 | {"error": "read_fenced", "refused": "read_fenced"} |
| 25 | C7 bob cannot release while fenced | pass | 0.195 | {"error": "read_fenced", "refused": "read_fenced"} |
| 26 | C8 relay releases carol's mailbox | pass | 0.002 | {"held": false, "mailbox": "carol"} |
| 27 | C9 carol processes the in-flight m4 | pass | 0.452 | {"dropped": 13, "duplicated": 4, "fetched": 36, "held": [], "posted": 32, "queued": {}, "tapped": 0} |
| 28 | C10 carol decrypts m4 but the APF refuses its consumption | pass | 0.002 | {"agent": "carol", "duplicates": [], "first": "read_fenced", "op": "alice-000006"} |
| 29 | C11 carol's application never sees m4 | pass | 0.002 | {"agent": "carol", "not_delivered": ["alice-000006"]} |
| 30 | C12 bob received m4 | pass | 0.002 | {"agent": "bob", "delivered": ["alice-000006"]} |
| 31 | D1 carol turns adversary: keeps its epoch keys, ignores its removal | pass | 0.019 | {"retain": true} |
| 32 | D2 alice commits the repair; bob confirms; the fence reopens | pass | 1.071 | {"branch": "38e61c3811329e537e37ab4dd2ff373cdfba66d9aa6fd835f249136b8d6d3732", "epoch": 3, "evidence": "pair", "index": 2, "opened": false, "read_fenced_afte... |
| 33 | D3 carol is out of the roster and the fence is open | pass | 0.002 | {"epoch": 3, "policy_revision": 3, "read_fenced": false, "repair_pending": null, "roster": ["alice", "bob"]} |
| 34 | D4 the relay leaks every copy for bob to carol | pass | 0.002 | {"enabled": true, "observer": "carol", "victim": "bob"} |
| 35 | D5 alice sends m6 after the repair | pass | 0.367 | {"epoch": 3, "op": "alice-000008", "recipients": ["bob"], "sent_ms": 1792955814095} |
| 36 | D6 relay drains | pass | 0.557 | {"dropped": 13, "duplicated": 4, "fetched": 38, "held": [], "posted": 33, "queued": {}, "tapped": 1} |
| 37 | D7 bob received m6 | pass | 0.003 | {"agent": "bob", "delivered": ["alice-000008"]} |
| 38 | D8 carol's retained keys cannot decrypt m6 and the APF refuses it | pass | 0.002 | {"agent": "carol", "attempts": [{"consume": "unauthorized", "decrypted": false, "from": "alice", "mls": "refused: ValidationError(WrongEpoch)", "op": "alice-... |
| 39 | E1 alice revokes bob's write right | pass | 0.117 | {"read_fenced": false, "revision": 93} |
| 40 | E2 bob cannot release without write | pass | 0.201 | {"error": "unauthorized", "refused": "unauthorized"} |
| 41 | E3 alice re-grants write at generation 1: a rekey offer only | pass | 0.098 | {"pending": true} |
| 42 | E4 the offer alone does not restore write | pass | 0.205 | {"error": "unauthorized", "refused": "unauthorized"} |
| 43 | E5 bob commits a new signature key and incarnation | pass | 0.859 | {"branch": "c01b2753c0a6eba48da6617bdd41da9e8ddbc8e8179001bd6d23c22f3aff3405", "epoch": 4, "generation": 1, "public_key": "de7e603c22585abcb575388e82f4a84562... |
| 44 | E6 bob sends m8 with its new key | pass | 0.401 | {"epoch": 4, "op": "bob-000009", "recipients": ["alice"], "sent_ms": 1792955816537} |
| 45 | E7 relay drains | pass | 0.556 | {"dropped": 14, "duplicated": 4, "fetched": 39, "held": [], "posted": 34, "queued": {}, "tapped": 1} |
| 46 | E8 alice received m8 | pass | 0.002 | {"agent": "alice", "delivered": ["bob-000009"]} |
| 47 | F1 alice and bob are on the APF frontier | pass | 0.006 | {"agents": ["alice", "bob"], "epoch": 4} |
| 48 | F2 every message reached every certified reader exactly once, except the fenced m4 | pass | 0.007 | {"excluded": ["carol:alice-000006"], "optional": 0, "pairs": 33} |
| 49 | F2b the honest agents rejected only duplicates | pass | 0.008 | {"alice": {}, "bob": {"undecryptable": 3}} |
| 50 | F2c no delivery without APF consumption, over the whole run | pass | 0.094 | {"consumed": 33, "deliveries": 33, "lost_after_consume": 0} |
| 51 | F3 the independent oracle accepts the exported APF trace | pass | 0.237 | {"events": 130, "kinds": {"admit": 2, "allocate": 22, "allow": 6, "confirm": 2, "consume": 33, "deny": 2, "emit": 18, "init": 1, "persist": 18, "recover": 5,... |
| 52 | F4 the oracle rejects every broken control derived from the trace | pass | 0.28 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "rekey that reuses another... |
| 53 | F5 every node drew its randomness from the QRNG service, with no failure | pass | 0.01 | {"nodes": {"alice": {"bytes": 556, "failures": 0, "key_exchange": "X25519MLKEM768", "post_quantum": true, "requests": 19, "source": "qrng"}, "apf": {"bytes":... |
| 54 | F6 the honest agents logged no processing error | pass | 0.004 | {"agents": ["alice", "bob"]} |
| 55 | F7 transport controls: no certificate, wrong peer, foreign CA, classical key exchange and a classical certificate for a service name are all refused | pass | 0.293 | {"a certificate from a foreign CA": "refused: Network Error", "a classical certificate that claims alice": "refused: [SSL: CERTIFICATE_VERIFY_FAILED] certifi... |
| 56 | F8 every service link, the QRNG link included, is ML-DSA-44 over X25519MLKEM768 | pass | 0.012 | {"connections": {"alice": {"attctl": ["pq/X25519MLKEM768"]}, "apf": {"alice": ["pq/X25519MLKEM768"], "attctl": ["pq/X25519MLKEM768"], "bob": ["pq/X25519MLKEM... |

## Final state

- APF roster: ['alice', 'bob']; epoch 4; policy revision 5; read fenced False
- Relay: posted 34, fetched 39, dropped 14, duplicated 4, tapped 1
