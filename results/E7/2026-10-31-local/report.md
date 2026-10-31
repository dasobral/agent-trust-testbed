# E7: Confused deputy over the network: invocation provenance and attenuation

- Result: **PASS**
- Backend: `local`; topology `four-agents`; entropy `qrng`
- Duration: 6.614 s; steps run: 34; steps not run: 0

Hypothesis: (H1) A deputy can act for an origin only through an APF invocation, and only within the rights that both hold. (H2) An origin without a right cannot obtain it through a deputy. (H3) A nested invocation cannot widen its parent, and only the recorded executor can use an invocation. (H4) Revoking the origin's right stops every pending use of the invocation chain at release, while the deputy keeps its own authority. (H5) The APF records who executed each operation and for whom; the oracle accepts the trace.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 alice founds the group | pass | 0.65 | {"branch": "8f9296f9a4c44033297232c1d46015b37a6107ee7a6b92e84377669f539027fb", "epoch": 0} |
| 1 | A2 bob, the deputy: read, write, admit | pass | 0.038 | {"revision": 1} |
| 2 | A3 carol, an origin with write: read, write, admit | pass | 0.038 | {"revision": 2} |
| 3 | A4 dave, a reader only: read, admit | pass | 0.038 | {"revision": 3} |
| 4 | A5 admit bob | pass | 0.771 | {"branch": "468485d7bf8e0bc73b9efa28551ec415c75af02cfb1e84b3ea73104ba407f4e8", "epoch": 1, "index": 0, "subject": "bob"} |
| 5 | A6 admit carol | pass | 1.013 | {"branch": "f5c3921b37df52d503b8b5abce2134ab20ecceccbf067dc88943d94858c93e4f", "epoch": 2, "index": 1, "subject": "carol"} |
| 6 | A7 admit dave | pass | 1.054 | {"branch": "f17aeff29b0c9b5a1e29efde95720ff6e883029f5b390c7ef461e7ec42829f1d", "epoch": 3, "index": 2, "subject": "dave"} |
| 7 | B1 carol invokes bob to write for her | pass | 0.037 | {"invocation": "invocation:7"} |
| 8 | B2 bob sends m1 under carol's invocation | pass | 0.165 | {"epoch": 3, "op": "bob-000001", "recipients": ["alice", "carol", "dave"], "sent_ms": 1793481465496} |
| 9 | B3 the APF records bob as executor of m1, for carol's invocation | pass | 0.002 | {"actor": "bob", "invocation": "invocation:7", "op": "bob-000001", "resource": "lab", "right": "write", "status": "persisted"} |
| 10 | B4 drain | pass | 0.512 | {"dropped": 0, "duplicated": 0, "fetched": 3, "held": [], "posted": 3, "queued": {}, "tapped": 0} |
| 11 | B5 dave received m1 | pass | 0.0 | {"agent": "dave", "delivered": ["bob-000001"]} |
| 12 | C1 dave (read only) cannot ask bob to write for him | pass | 0.037 | {"error": "unauthorized", "refused": "unauthorized"} |
| 13 | C2 dave cannot write himself | pass | 0.074 | {"error": "unauthorized", "refused": "unauthorized"} |
| 14 | C3 bob cannot widen carol's invocation to admin | pass | 0.027 | {"error": "invalid_invocation", "refused": "invalid_invocation"} |
| 15 | C4 bob cannot pass carol's invocation to dave, who lacks write | pass | 0.028 | {"error": "unauthorized", "refused": "unauthorized"} |
| 16 | C5 bob delegates carol's invocation to alice, within its rights | pass | 0.078 | {"invocation": "invocation:15"} |
| 17 | C6 bob cannot use the nested invocation: alice is its executor | pass | 0.036 | {"error": "unauthorized", "refused": "unauthorized"} |
| 18 | C7 dave cannot use carol's invocation: bob is its executor | pass | 0.037 | {"error": "unauthorized", "refused": "unauthorized"} |
| 19 | C8 alice sends m2 under the nested invocation | pass | 0.166 | {"epoch": 3, "op": "alice-000001", "recipients": ["bob", "carol", "dave"], "sent_ms": 1793481466494} |
| 20 | C9 the APF records alice as executor of m2 under the nested invocation | pass | 0.003 | {"actor": "alice", "invocation": "invocation:15", "op": "alice-000001", "resource": "lab", "right": "write", "status": "persisted"} |
| 21 | D0 drain before the policy change | pass | 0.512 | {"dropped": 0, "duplicated": 0, "fetched": 6, "held": [], "posted": 6, "queued": {}, "tapped": 0} |
| 22 | D1 alice revokes carol's write | pass | 0.037 | {"read_fenced": false, "revision": 22} |
| 23 | D2 bob can no longer write for carol | pass | 0.074 | {"error": "unauthorized", "refused": "unauthorized"} |
| 24 | D3 alice can no longer write for carol through the nested invocation | pass | 0.074 | {"error": "unauthorized", "refused": "unauthorized"} |
| 25 | D4 bob keeps his own authority: m3 without invocation | pass | 0.166 | {"epoch": 3, "op": "bob-000004", "recipients": ["alice", "carol", "dave"], "sent_ms": 1793481467362} |
| 26 | D5 m3 is bob's own operation | pass | 0.005 | {"actor": "bob", "invocation": null, "op": "bob-000004", "resource": "lab", "right": "write", "status": "persisted"} |
| 27 | D6 drain | pass | 0.512 | {"dropped": 0, "duplicated": 0, "fetched": 9, "held": [], "posted": 9, "queued": {}, "tapped": 0} |
| 28 | E1 every released message reached every certified reader exactly once | pass | 0.001 | {"excluded": [], "optional": 0, "pairs": 9} |
| 29 | E2 no delivery without APF consumption | pass | 0.008 | {"consumed": 9, "deliveries": 9, "lost_after_consume": 0} |
| 30 | E3 every agent is on the APF frontier | pass | 0.003 | {"agents": ["alice", "bob", "carol", "dave"], "epoch": 3} |
| 31 | E4 the oracle accepts the trace | pass | 0.209 | {"events": 41, "kinds": {"admit": 3, "allocate": 6, "allow": 8, "consume": 9, "deny": 1, "emit": 3, "init": 1, "persist": 3, "recover": 4, "release": 3}, "or... |
| 32 | E5 the oracle rejects the broken controls that apply | pass | 0.205 | {"rejected": {"release cover that omits a reader": "unsound_cover", "replayed consumption": "replay", "second envelope under one operation": "binding_mismatc... |
| 33 | E6 no agent logged a processing error | pass | 0.002 | {"agents": ["alice", "bob", "carol", "dave"]} |

## Final state

- APF roster: ['alice', 'bob', 'carol', 'dave']; epoch 3; policy revision 6; read fenced False
- Relay: posted 9, fetched 9, dropped 0, duplicated 0, tapped 0
