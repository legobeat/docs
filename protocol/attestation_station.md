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

// Point to attestation page to see what are the bytes inside val
