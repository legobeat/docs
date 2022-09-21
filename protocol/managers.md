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

Before the start of each epoch, managers will query their own DHT buckets to obtain keys they are providing.&#x20;

Then, they can query the DHT network for figure out who is managing the neighbours of the peers associated with these keys using the GET\_PROVIDERS command.

Then, they should query the signature of every peer they manage. This signature will be retrieved using GET\_VALUE command. They will use this signature to verify proofs received from their managers.

Due to the difference in connections speeds and computer performance, and unintended random delays in the network, there is a potential that every peer will receive a slightly different provider set for a particular peer, due to the fact that anyone can go offline at any time, resulting in change in DHT network.

When they collect all that data, they can start requesting the IVPs.

### Proposal

Currently the managers don't have anything motivating them to do this computation, that's why real incentives need to be put in place. The only reward we could give the manager that is providing a valid proof, is the reputation, but in a particular domain.&#x20;
