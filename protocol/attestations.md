---
This page describes the start used for attestations
---

# Attestations

Another name for attestation is the opinion or rating one peer gives to another.
Also, attestations are given for one transaction/interaction between peers.

Examples (Let's assume we are doing ratings from 0-5):
- Alice attests Bob with a rating of 5
- Alice attests to Carol with a rating of 2
- Bob attests to Alice with a rating of 3
- Carol says Bob with a rating of 4
- Alice attests Bob with a rating of 1

The structure of the attestation:
```rust
struct Attestation<F: FieldExt> {
    about: F,
    key: F,
    value: F,
    message: F
}
```

In the context of EigenTrust:
- `about` is the address we are making an attestation about. This could be an EOA, a Safe wallet, a DAO, etc.
- `key` is the unique identifier for the transaction we are attesting
- `value` is the score we are giving out, could be any value from 0 to F::FIELD_SIZE
- `message` is an additional message which could be a domain in which a rating was given or a message or a content hash attached to this attestation.

Then we use Poseidon to hash the attestation and get a single 32-byte value, which we sign using the ECDSA signing algorithm:
```rust
let att_hash = Poseidon::hash(attestation);
let sig = ECDSA::sign(att_hash, keys);
```
The signature represented in bytes, along with the attestation hash bytes are stored in AttestationStation smart contract. See [AttestationStation](../protocol/attestation_station.md).

The reason we are signing the attestation is in case we want to verify the validity of the attestation in an off-chain environment.
