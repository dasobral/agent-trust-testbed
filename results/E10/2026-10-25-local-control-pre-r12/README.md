# E10 control: agent-trust without R-12

This run uses the testbed code later committed as `7857d11`, which was still uncommitted on `251286e`
(hence the report stamp `251286e…-dirty`); the experiment and the sidecar are unchanged,
with agent-trust resolved to branch `feat/r11-r12` (`0db5f6f`: R-11 without R-12)
instead of `feat/r12-expiry` (`e774ba2`). Only R-12 differs. The binary was built in a
separate target directory and selected with `ATT_NODE_BIN`; the manifest and lock file
were restored afterwards.

Result: FAIL at C1, `expected read_fenced, got unauthorized`. This is the stall that
alignment note A-5 predicted. The release is refused because carol's delegation chain
has expired, but nothing records the expiry, so the fence stays open and no repair can
remove her.
