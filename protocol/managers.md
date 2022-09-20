---
description: >-
  This page describes how do we make sure the system is always online by
  delegating the reputation calculation to other peers.
---

# Managers

### Why we need managers?

If we required each peer to calculate their own scores, then the whole network would need to be online at the same time each epoch, until the convergence. This is an unreasonable request, so instead we are incentivising peers to stay online and calculate the score for other pees.&#x20;

### How are managers picked?

When peer comes online the DHT network will assign them keys that they should provide. Managers use this functionality to figure out who they should manage.

At he beginning of each epoch, managers will query their own DHT buckets to obtain keys they are providing. Then, they can query the DHT network for figure out who is managing the neighbours of the peers associated with these keys. When they collect all that data, they can start providing the IVPs.

TBA:

* Explain how managers figure out if are selected every epoch. Explain that managers can have a key in their bucket, but might not be asked for their proofs, this is because of the delay in DHT requests
* Add illustration on how managers figure out who they manage
* Explain how managers are finding the managers of the peers they manage.
* Explain that managers don't have anything to gain in this model, and how many of them is needed to keep the network running - explain why the incentives are needed (not possible to implement without domains)
