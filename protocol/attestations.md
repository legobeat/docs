---
This page describes the start used for attestations
---

# Attestations

Another name for attestation is opinion or rating one peer gives to another.
Also, attestations are given for one transaction/interaction between the peers.

Examples (Lets assume we are doing ratings from 0-5):
Alice attests Bob with rating 5
Alice attests Carol with rating 2
Bob attests Alice with rating 3
Carol says Bob with rating 4
Alice attests Bob with rating 1

Now lets assign a unique identifier for each peer:
Alice = 1
Bob = 2
Carol = 3

The structure of the signature:
```rust
struct Attestation<F: FieldExt> {
    about: F,
    key: F,
    value: F,
    message: F
}
```

// TODO: Describe how attestation is hashed and stored in AttestationStation
// TODO: Describe about, key, value, message in EigenTrust context
