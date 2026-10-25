# E9: Policy binding in the canonical ack (agent-trust R-11)

Spec: [`experiments/e9-ack-policy.toml`](../../experiments/e9-ack-policy.toml) ·
Topology: [`topologies/three-agents.toml`](../../topologies/three-agents.toml) ·
Evidence: [`results/E9/`](../../results/E9/)

## Question

agent-trust R-11 (D-11) moved the policy binding out of the MLS GroupContext. The
`apf_anchor` extension is now a constant set at genesis (`{version 2, apf_key_id,
group}`). No commit carries a GroupContextExtensions proposal, and the APF's canonical
ack carries `policy_revision` and `policy_digest` instead. With this change, every commit
the system produces is an operation that the ETK model covers. Do the members see this
across the network, for every commit kind? Does the APF refuse a repair that tries to
rewrite the GroupContext?

## Setup

alice (root); bob and carol delegated `read, write, admit`. The sidecar exposes
`GET /control/anchor`, which returns the member's current GroupContext anchors, read
from its exported GroupInfo, and the ack it last merged or joined under. The hostile
`POST /control/repair_with_extensions` builds the planned repair with raw OpenMLS and
adds a GroupContextExtensions proposal that repeats the current extensions exactly.

## Hypotheses and results

| | Hypothesis | Steps | Observed |
|---|---|---|---|
| H1 | The genesis `apf_anchor` is the same bytes at every member after admission, repair and rekey | A2, A8, B6, C4 | One anchor, `02…` (version 2), identical at every member and to genesis |
| H2 | Every ack carries the APF's policy at the canonical step; a repair ack is at or above the fence revision | A8, B5, B6, C4 | Admit, repair and update acks equal the APF's `policy_revision` and `policy_digest`; repair ack revision 3, fence revision 3 |
| H3 | A repair with a GroupContextExtensions proposal is refused and changes nothing | B3, B4 | `noncanonical_commit: proposals`; still fenced, roster unchanged; the plain repair then succeeds |
| H4 | The oracle accepts the trace | D3, D4 | Accepted; the broken controls are rejected |

## Limits

The anchor is read from a GroupInfo that the member exports. That is the member's own
view, which is what a member acts on. The client's refusal of an ack with a lower policy
revision is not exercised here: forging one needs the APF key. agent-trust's
`adapter_admission` tests cover it.
