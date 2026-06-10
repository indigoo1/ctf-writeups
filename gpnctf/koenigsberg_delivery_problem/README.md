# Königsberg Delivery Problem

| Field | Details |
|-------|---------|
| **Challenge** | Königsberg Delivery Problem |
| **CTF** | GPN CTF 2024 |
| **Category** | Reverse Engineering |
| **Flag** | `GPNCTF{s4y_EulER_The_Owl_OWls_In_kön1g5BeR6_10_7iM3S_FAst!}` |

---

## Overview

A Linux x86-64 PIE binary (`cartographer`) that reads 250 signed bytes via `scanf("%hhd;", ...)`, runs them through a 250-state DFA implemented as a giant jump table in `cfg()`, then calls `check_instance(stack_buf, 250)`.

---

## Reversing

`cfg()` allocates `0x108` bytes on the stack used as a visit-count buffer. Each state block executes `incb [rsp + state_id]`, incrementing that state's counter.

`check_instance` loops over all 250 bytes with `cl=1`; if any byte is `0`, `cl` is set to `0`. The final `test $1, %cl` + `je failure` means **every state must be visited at least once**.

250 inputs, 250 states → **Hamiltonian path required**.

---

## Graph Extraction

Scan the binary for the pattern `fe 44 24 <state_id>` (`incb [rsp+N]`), then follow the `movzbl`, `cmp $max_val`, `ja`-to-fail, `inc rcx`, `lea <table_rip>` sequence, and read 4-byte relative offsets from the jump table to resolve next-state VAs.

Result: **250 nodes, ~24,987 directed edges, average out-degree ≈ 100**.

---

## Solving

With average out-degree ~100 the graph is dense — backtracking with a heuristic works fast.

**Warnsdorff's heuristic:** at each step, move to the neighbour with the fewest onward neighbours. A valid path from node 0 to node 1 was found with at most 2 visits on any state, and all 250 visit-counts verified nonzero locally.

### Submission

Encode the path as decimal integers joined by `;` and pipe to the server:

```bash
echo "0;12;7;..." | ncat --ssl <host> 443
```

---

## Flag

```
GPNCTF{s4y_EulER_The_Owl_OWls_In_kön1g5BeR6_10_7iM3S_FAst!}
```
