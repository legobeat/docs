---
description: >-
  This page describes the reputation decay over time, resulting from peers being
  un-active.
---

# Reputation decay

Reputation decay is what happens to your reputation if you were un-active for longer periods of time. Before we dive into how the decay is calculated, let's point out a few things about signatures on opinions:

* DHT can only hold one value per key, meaning that every time you broadcast a new signature, it will overwrite the old one.
* Values will have a parameter called `epoch`, which is an epoch at which this value was added into the bucket.

When calculating the score for a particular peer, we will calculate the difference between the current epoch and the epoch the signature was added into the bucket. We will use this difference to calculate the decay. The following function will be used to decay calculation:

TBA - inverse sigmoid like function

We could also use different functions for different domains. For example the manager domain should have much steeper slope, and the cliff should start sooner, since we want to incentivise managers to constantly be online to keep their reputation at a maximum level.

The reputation decay calculation will be applied in CVPs, after aggregating the IVPs and taking the reputation score from the last iteration.
