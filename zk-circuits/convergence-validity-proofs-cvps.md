---
description: CVPs do additional checks after the peers score has converged.
---

# Convergence validity proofs (CVPs)

There are 2 things that CVPs are doing:

1. That the peers and peer's neighbour message hash has remained unchanged in every iteration during the convergence.
2. Exposed the final score to be constrained to the public input.

Circuit inputs:

```rust
*private - ivp_proofs[NUM_ITERATIONS] // ivp proofs from `epoch`
*private - message_hashes[NUM_NEIGHBOURS] // Message hashes from all the neighbours
*private - scores[NUM_ITERATIONS] // Score of peer i in all iteration
*private - message_hash // Message hash of the peer
public   - pubkey_i // Public key of the peer
```

Circuit description:

```rust
for i in 0..NUM_ITERATIONS {
    
}
```
