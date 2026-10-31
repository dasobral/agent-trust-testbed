# E5: Eight agents in parallel: latency, APF cost growth and safety at scale

- Result: **PASS**
- Backend: `local`; topology `scale-8`; entropy `qrng`
- Duration: 38.965 s; steps run: 22; steps not run: 0

Hypothesis: (H1) Admission, release and delivery keep working as the group grows to eight members; their cost is measured, not assumed. (H2) With all eight agents sending in parallel, with and without relay faults, every message reaches every reader exactly once and only after an APF consumption. (H3) A revocation and repair with seven continuing members completes and excludes the revoked member. (H4) The APF's time per request grows with its kernel history, because the kernel validates its whole hash chain of full snapshots on every request; E5 measures the growth. (H5) The oracle accepts the full trace.

| # | Step | Result | Seconds | Detail |
|---|------|--------|---------|--------|
| 0 | A1 agent01 founds the group | pass | 0.667 | {"branch": "4ce2320a28828585668df6b82d9297e89232e2b7a8a38bdf82f78d713b19ad78", "epoch": 0} |
| 1 | A2 delegate and admit the seven other agents, one at a time | pass | 8.123 | {"admissions": [{"group_size": 2, "member": "agent02", "ms": 807.3}, {"group_size": 3, "member": "agent03", "ms": 1040.1}, {"group_size": 4, "member": "agent... |
| 2 | A3 eight members at epoch 7 | pass | 0.001 | {"epoch": 7, "policy_revision": 7, "read_fenced": false, "repair_pending": null, "roster": ["agent01", "agent02", "agent03", "agent04", "agent05", "agent06",... |
| 3 | A4 every agent is on the APF frontier | pass | 0.006 | {"agents": ["agent01", "agent02", "agent03", "agent04", "agent05", "agent06", "agent07", "agent08"], "epoch": 7} |
| 4 | B1 four rounds: all eight agents send one message each in parallel (32 messages, 224 deliveries) | pass | 17.48 | {"apf_p50_growth": 1.0, "rounds": [{"apf_p50_ms": 54.6, "apf_requests": 88, "history_from": 15, "history_to": 94, "round": 1, "seconds": 4.42}, {"apf_p50_ms"... |
| 5 | B2 every message reached its seven readers exactly once | pass | 0.003 | {"excluded": [], "optional": 0, "pairs": 224} |
| 6 | B3 no delivery without APF consumption | pass | 0.368 | {"consumed": 224, "deliveries": 224, "lost_after_consume": 0} |
| 7 | B4 latency without faults | pass | 0.005 | {"admit_ms": {"max": 1288.1, "n": 7, "p50": 1160.8, "p95": 1250.6}, "apf_ms_by_history": [{"history_up_to": 33, "p50": 27.82, "requests": 40}, {"history_up_t... |
| 8 | C1 relay drops 10%, duplicates 10%, reorders | pass | 0.0 | {"drop": 10, "duplicate": 10, "reorder": true, "seed": 3} |
| 9 | C2 one round under faults | pass | 4.487 | {"apf_p50_growth": 1.0, "rounds": [{"apf_p50_ms": 52.9, "apf_requests": 88, "history_from": 335, "history_to": 414, "round": 1, "seconds": 4.49}]} |
| 10 | C3 every message reached every reader exactly once despite the faults | pass | 0.004 | {"excluded": [], "optional": 0, "pairs": 280} |
| 11 | C4 every refused copy was a duplicate | pass | 0.006 | {"agent01": {"undecryptable": 1}, "agent02": {"undecryptable": 3}, "agent03": {"undecryptable": 1}, "agent04": {"undecryptable": 1}, "agent05": {"undecryptab... |
| 12 | D1 agent01 revokes agent08's read right | pass | 0.046 | {"read_fenced": true, "revision": 415} |
| 13 | D2 repair with seven continuing members | pass | 1.73 | {"branch": "5afb39d0a5322b2d27d44e54e4e299768bd7cb92396d067a12c78381f2ac6476", "epoch": 8, "evidence": "pair", "index": 7, "opened": false, "read_fenced_afte... |
| 14 | D3 one round from the seven continuing members | pass | 3.546 | {"apf_p50_growth": 1.0, "rounds": [{"apf_p50_ms": 48.3, "apf_requests": 70, "history_from": 418, "history_to": 480, "round": 1, "seconds": 3.54}]} |
| 15 | D4 agent08 applied its own removal | pass | 0.0 | {"epoch": null, "future": 0, "membership": "removed", "restarts": 0, "skipped_entries": 6, "waiting": []} |
| 16 | D5 completeness over the whole run | pass | 0.004 | {"excluded": [], "optional": 0, "pairs": 322} |
| 17 | D6 no duplicate, no delivery without consumption, over the whole run | pass | 0.554 | {"consumed": 322, "deliveries": 322, "lost_after_consume": 0} |
| 18 | D7 the oracle accepts the full trace | pass | 0.635 | {"events": 559, "kinds": {"admit": 7, "allocate": 47, "allow": 21, "confirm": 2, "consume": 322, "deny": 1, "emit": 47, "init": 1, "persist": 47, "recover": ... |
| 19 | D8 the oracle rejects the broken controls | pass | 1.291 | {"rejected": {"fence opened by API only (no repair commit)": "invalid_repair", "fence opened without confirmation": "unconfirmed", "release after the read de... |
| 20 | D9 every node drew from the QRNG service without failure | pass | 0.004 | {"nodes": {"agent01": {"bytes": 1032, "failures": 0, "key_exchange": "none (plain HTTP)", "post_quantum": false, "requests": 28, "source": "qrng"}, "agent02"... |
| 21 | D10 latency over the whole run, and APF time per request against history length | pass | 0.005 | {"admit_ms": {"max": 1288.1, "n": 7, "p50": 1160.8, "p95": 1250.6}, "apf_ms_by_history": [{"history_up_to": 48, "p50": 36.27, "requests": 63}, {"history_up_t... |

## Final state

- APF roster: ['agent01', 'agent02', 'agent03', 'agent04', 'agent05', 'agent06', 'agent07']; epoch 8; policy revision 8; read fenced False
- Relay: posted 322, fetched 335, dropped 8, duplicated 13, tapped 0
