---
This page describes the AttestationStation smart contract and how its used within the EigenTrust protocol context
---

# AttestationStation

Another name for attestation is the opinion or rating one peer gives to another.
Also, attestations are given for one transaction/interaction between peers.

Examples (Let's assume we are doing ratings from 0-5):
- Alice attests Bob with a rating of 5
- Alice attests to Carol with a rating of 2
- Bob attests to Alice with a rating of 3
- Carol says Bob with a rating of 4
- Alice attests Bob with a rating of 1

Map of attestations
```solidity
mapping(address => mapping(address => mapping(bytes32 => bytes))) public attestations;
```

Attestation structure:
```solidity
struct AttestationData {
    address about;
    bytes32 key;
    bytes val;
}
```

The structure of `bytes val;` are described in [Attestations](../1_attestations.md).
