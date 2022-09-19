---
description: >-
  This page describes the proofs provided in each iteration during the
  convergence process.
---

# Iteration validity proofs (IVPs)

Circuit description:

```rust

// ------------------ INPUTS -------------------------
*private - pubkeys[NUM_NEIGHBOURS]  // public key of neighbors
*private - op_ij[NUM_NEIGHBOURS] // local scores towards the neighbors j
*private - op_ji[NUM_NEIGHBOURS]  // all neighbors opinions towards peer i
*private - neighbour_scores[NUM_NEIGHBOURS] // neighbour reputation scores
                                    // in the previous iteration
*private - merkle_paths[NUM_NEIGHBOURS] // merkle paths needed for calculating
                                    // the message hashes of all the neighbours
*private - signature_i // Signature on message_hash
*private - pubkey_i // public key of peer i

public   - message_hash // Message hash of peer i
public   - message_hashed[NUM_NEIGHBOURS] // message hashes for all neighbours j
public   - score // Score of peer i in the iteration
public   - pubkey_v // Public key of the verifier
public   - iteration // Current iteration

constant - bootstrap_pubkeys[NUM_BOOTSTRAP_PEERS] // pubkeys of bootstrap nodes
constant - bootstrap_score // Score of the bootstrap peers
constant - scale // We have to scale the opinions (which are between 0 and 1)
                     // to do the arithmetic inside the circuit

// i - the prover
// j - neighbors of i

// The amount of reputation getting into the circuit has to equal
// the amount getting out of the circuit.

// The neighbors keys you received reputation from should equal
// the keys you are giving reputation to

// ---------------- CONSTRAINTS --------------
let score_ji = []
for j in 0..NUM_NEIGHBOURS {
  // Here we check that each merkle_path at index n results in a
  // merkle root that is equal to message_hash at the same index
  let m_hash_j = validate_merkle_path(merkle_paths[j], op_ji[j], pubkey_i)
  // Check neighbour message hash
  assert_eq(message_hashes[j], m_hash_j)
  // Aggregate all the opinion proofs you got from the neighbors
  agreggate_neighbour_proof(m_hash_j, neighbour_scores[j], pubkeys[j], iteration - 1)
  // Pushing to scores array
  score_ji.push(neighbour_scores[j] * op_ji[j])
}
// Aggregate previous proof to make sure we are using the same message_hash
aggregate_self_proof(message_hash, previous_score, pubkey_i, iteration - 1)
// Check if we are at zero-th iteration
let is_zero_it = is_zero(iteration)
// Is bootstrap peer at zero iteration
let is_bootstrap_at_zero = and(is_bootstrap, is_zero_it)
// Calculate the sum of all the scored you got from all the neighbors
let sum_ji = sum(score_ji[..]) / scale
// Assign bootstrap score if we at bootstrap peer at iter zero
let iter_score = conditionally_select(
  is_bootstrap_at_zero,
  sum_ji,
  bootstrap_score
)
// Check if score is equal to public input
assert_eq(iter_score, score)
// Verify the signature on message_hash
verify_eddsa_signature(signature_i, pubkey_i, message_hash)
```

