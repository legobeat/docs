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

At he beginning of each epoch, managers will query their own DHT buckets to obtain keys they are providing. Then, they can query the DHT network for figure out who is managing the neighbours of the peers associated with these keys. When they collect all that data, they can start providing the IVPs and CVPs.

### What happens when they misbehave?

After every iteration interval, peers will keep track who responded with invalid IVPs. Then they will cache their peer id in memory, so that they don't make a request to them in the next iteration and give them `0` reputation at the end of the last iteration.

To the nodes that behaved correctly, peers will distribute their scores to each of them equally.
