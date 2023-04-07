---
This page describes the AttestationStation smart contract and how its used within the EigenTrust protocol context
---

# AttestationStation

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

The structure of `bytes val;` are described in [Attestations](https://app.gitbook.com/o/8CNhhLC4GayMSznMxZt1/s/CH5I1CKimkNiX7iqcLcK/protocol/attestations).
