# Roadmap

| Item | Kind | Status |
|---|---|---|
| E1 revocation fence, repair, rekey | Experiment | Done: local, compose, k3s; also over mTLS |
| E2 entropy outage | Experiment | Done |
| E3 APF and agent restarts | Experiment | Done |
| E4 hostile commit distribution | Relay feature and experiment | Done |
| E5 scale | Topology and experiment | Done at 8 agents in both suites; found F-8, fixed by agent-trust R-13 (branch `feat/r13-history`): in the `pq` suite E5 passes in 39 s with a flat APF cost per request |
| E8 suites | Experiment | Done: the same scenario in `pq` and `classical`, with sizes |
| E7 confused deputy over the network | Experiment | Done |
| E9 policy in the canonical ack (agent-trust R-11) | Experiment and sidecar route | Done: local, compose, k3s (CI run `40226788501`) |
| E10 delegation expiry as a ledger event (agent-trust R-12) | Experiment | Done: local, compose, k3s (CI run `40226788501`), with a control run against agent-trust without R-12 |
| Agent persistence | Upstream (F-2) and sidecar | Done; upstream on branch `feat/durable-lab-provider`, waiting for merge |
| mTLS between services | Services | Done, with transport identity bound to protocol identity |
| Kubernetes backend | Backend | Done, tested on k3s, with NetworkPolicy checks (`experiments/k8s-isolation.toml`) |
| E6 live Entropy Core | Topology | **Open**: needs a reachable Entropy Core. Set `entropy.url` (and the `AGENT_TRUST_QRNG_*` secrets) in a topology; no code change. The link uses `pq-preferred` (D-14); record `GET /control/entropy` to show whether the Entropy Core negotiates `X25519MLKEM768`. |
| F-8 kernel history cost | Upstream and testbed | **Implemented** (agent-trust R-13, D-17, 2026-10-31, branch `feat/r13-history`, not merged): compact events and a bounded live state, with the store hardened against tampering, rollback and APF key theft. The sidecar sends `known_head` and the APF returns the signed head. E5 in the `pq` suite passes |
| F-1, F-3, F-4, F-5 | Upstream | Open proposals in `docs/findings.md` |
| F-9 post quantum | Upstream and testbed | **Done** (D-13, 2026-10-25): PQ-1 and PQ-3 in agent-trust C-42 (`pq` suite, the default), PQ-2 in openmls-qrand and `att-node` (X25519MLKEM768 on every link), PQ-4 in `att-node pki` (ML-DSA-44 certificates). E8 compares both suites. E5 in the `pq` suite waits for F-8 |

Next experiments that the infrastructure supports without new code: larger groups
(`scripts/make-topology.py N`), now that F-8 is resolved, E1 to E7 on a multi-node k3s
cluster (push the image to a registry), and mTLS with a real Entropy Core.
