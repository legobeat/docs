---
description: >-
  This page describes the proofs provided in each iteration during the
  convergence process.
---

# Iteration validity proofs (IVPs)

First, let's define circuit inputs:

```rust
// public key of neighbors
*private - pubkeys[NUM_NEIGHBOURS]
// local scores towards the neighbors j
*private - op_ij[NUM_NEIGHBOURS]
// all neighbors opinions towards peer i
*private - op_ji[NUM_NEIGHBOURS]
// neighbour reputation scores in the previous iteration
*private - neighbour_scores[NUM_NEIGHBOURS]
// merkle paths needed for calculating the message hashes of all the neighbours
*private - merkle_paths[NUM_NEIGHBOURS]
// Signature on message_hash
*private - signature_i
// public key of peer i
*private - pubkey_i
// Our score in previous iteration
*private - previous_score 

// Message hash of peer i
public   - message_hash
// Score of peer i in the iteration
public   - score
// Current iteration
public   - iteration

// pubkeys of bootstrap nodes
constant - bootstrap_pubkeys[NUM_BOOTSTRAP_PEERS]
// Score of the bootstrap peers
constant - bootstrap_score
// We have to scale the opinions (which are between 0 and 1)
// to do the arithmetic inside the circuit
constant - scale
```

First thing we want to do is aggregate the proofs we got from our neighbours. In order to do that we would first need to construct the message hash. We do that by re-creating the Merkle root, using the Merkle path provided by the neighbours. At the end we calculate the score given to use by multiplying the neighbour global score with their opinion about us.

```rust
let score_ji = []
for j in 0..NUM_NEIGHBOURS {
  // Here we are re-constructing the message hash
  let m_hash_j = validate_merkle_path(merkle_paths[j], op_ji[j], pubkey_i)
  // Aggregate all the opinion proofs you got from the neighbors
  agreggate_neighbour_proof(m_hash_j, neighbour_scores[j], iteration - 1)
  // Pushing to scores array
  score_ji.push(neighbour_scores[j] * op_ji[j])
}
```

Next we want to aggregate our proof from previous iteration. The reason for this is to prove that our message hash has not changes since previous iteration.

```rust
// Aggregate previous proof to make sure we are using the same message_hash
aggregate_self_proof(message_hash, previous_score, iteration - 1)
```

Next, we want to do some validation regarding our message hash. There are 4 things we need to check:

* That all of our neighbours public keys are unique. We do this to ensure we are not giving reputation to the same neighbour more than once, to avoid some funny business coordinated between managers.
* Make sure our own public key is not in the set. We can't give back reputation to ourselves.
* Make sure our opinions scores add up to one (scaled).
* Make sure those public keys and opinions that we checked previously are actually the ones that are used in our message hash.

```rust
// Ensure uniqueness of neighbours
assert_unique(pubkeys[..])
// Assert that we are not our own neighbour
assert_eq(set_membership(pubkeys[..], pubkey_i), F::zero())
// Assert that the sum of your opinions is equal to 1.0 * scale
assert_eq(sum(op_ij[..]), scale)
// Reconstruct the whole merkle tree
let rec_message_hash = construct_merkle_tree(pubkeys[..], op_ij[..])
// Make sure its the same as public input
assert_eq(rec_message_hash, message_hash)
```

Next, we are checking if we are bootstrap peer in the initial iteration. If we are, we should have a bootstrap score:

```rust
// Check if we are at zero-th iteration
let is_zero_it = is_zero(iteration)
// Check if we are a bootstrap peer
let is_bootstrap = set_membership(bootstrap_pubkeys[..], pubkey_i)
// Is bootstrap peer at zero iteration
let is_bootstrap_at_zero = and(is_bootstrap, is_zero_it)
// Calculate the sum of all the scored you got from all the neighbors
let sum_ji = sum(score_ji[..]) / scale
// If we are at iteration zero, we should have zero score
let iter_score = conditionally_select(
  is_zero_it,
  F::zero(),
  sum_ji,
)
// Except if we are a bootstrap peer, than we should have a bootstrap score
let final_score = conditionally_select(
  is_bootstrap_at_zero,
  iter_score,
  bootstrap_score
)
// Check if score is equal to public input
assert_eq(final_score, score)
```

And finally, we verify our EDDSA signature, providing the message hash:

```rust
// Verify the signature on message_hash
verify_eddsa_signature(signature_i, pubkey_i, message_hash)
```

Putting it all together:

<pre class="language-rust"><code class="lang-rust">
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
*private - previous_score // Our score in previous iteration

public   - message_hash // Message hash of peer i
public   - score // Score of peer i in the iteration
public   - iteration // Current iteration

constant - bootstrap_pubkeys[NUM_BOOTSTRAP_PEERS] // pubkeys of bootstrap nodes
constant - bootstrap_score // Score of the bootstrap peers
constant - scale // We have to scale the opinions (which are between 0 and 1)
                     // to do the arithmetic inside the circuit

// ---------------- CONSTRAINTS --------------
let score_ji = []
for j in 0..NUM_NEIGHBOURS {
  // Here we check that each merkle_path at index n results in a
  // merkle root that is equal to message_hash at the same index
  let m_hash_j = validate_merkle_path(merkle_paths[j], op_ji[j], pubkey_i)
  // Aggregate all the opinion proofs you got from the neighbors
  agreggate_neighbour_proof(m_hash_j, neighbour_scores[j], iteration - 1)
  // Pushing to scores array
  score_ji.push(neighbour_scores[j] * op_ji[j])
}
// Aggregate previous proof to make sure we are using the same message_hash
aggregate_self_proof(message_hash, previous_score, iteration - 1)
// Ensure uniqueness of neighbours
assert_unique(pubkeys[..])
// Assert that we are not our own neighbour
assert_eq(set_membership(pubkeys[..], pubkey_i), F::zero())
// Assert that the sum of your opinions is equal to 1.0 * scale
assert_eq(sum(op_ij[..]), scale)
// Reconstruct the whole merkle tree
let rec_message_hash = construct_merkle_tree(pubkeys[..], op_ij[..])
// Make sure its the same as public input
assert_eq(rec_message_hash, message_hash)
<strong>// Check if we are at zero-th iteration
</strong>let is_zero_it = is_zero(iteration)
// Check if we are a bootstrap peer
let is_bootstrap = set_membership(bootstrap_pubkeys[..], pubkey_i)
// Is bootstrap peer at zero iteration
let is_bootstrap_at_zero = and(is_bootstrap, is_zero_it)
// Calculate the sum of all the scored you got from all the neighbors
let sum_ji = sum(score_ji[..]) / scale
// If we are at iteration zero, we should have zero score
let iter_score = conditionally_select(
  is_zero_it,
  F::zero(),
  sum_ji,
)
// Except if we are a bootstrap peer, than we should have a bootstrap score
let final_score = conditionally_select(
  is_bootstrap_at_zero,
  iter_score,
  bootstrap_score
)
// Check if score is equal to public input
assert_eq(final_score, score)
// Verify the signature on message_hash
verify_eddsa_signature(signature_i, pubkey_i, message_hash)</code></pre>

TBA:

* Figure out how to handle 0 iteration - there are no proofs from neighbours to aggregate so everyone should have reputation 0, except bootstrap peers.
