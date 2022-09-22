---
description: >-
  This page describes the reputation decay over time, resulting from peers being
  un-active.
---

# Reputation decay - Optional

Reputation decay is what happens to your reputation if you were un-active for longer periods of time. Before we dive into how the decay is calculated, let's point out a few things about signatures on opinions:

* DHT can only hold one value per key, meaning that every time you broadcast a new signature, it will overwrite the old one.
* Values will have a parameter called `epoch`, which is an epoch at which this value was broadcasted to the network.

When calculating the score for a particular peer, we will calculate the difference between the current epoch and the epoch the neighbour's signature was added into the bucket. We will use this difference to calculate the decay. The following function can be used to decay calculation:

$$
scale-\left(scale/(1+offset*\exp(-\frac{x}{slope}))\right)
$$

<figure><img src="../.gitbook/assets/Screenshot 2022-09-20 at 12.13.41.png" alt=""><figcaption><p>scale = 100, offset = 100, slope = 100</p></figcaption></figure>

We could also use different functions for different domains. For example the manager domain should have much steeper slope, and the cliff should start sooner, since we want to incentivise managers to constantly be online to keep their reputation at a maximum level.

_<mark style="color:red;">**IMPORTANT NOTE**</mark>_: This is just a proposal, but explicit reputation decay might not be necessary since the reputation will decay naturally for 2 reasons:

* Number of neighbours are limited to 256, peers will occasionally remove some neighbours from slots, automatically giving them 0 reputation. They will probably remove someone with whom they did not interact in a while.
* As soon as the neighbour increases the opinion towards one neighbour, all the other neighbours will have their score diluted by the same amount. The scores of neighbours with whom they interact more ofter, will be updated more ofter, leaving other neighbours score getting diluted more and more over time.
