---
description: This page explains the lifecycle of the network convergence.
---

# Epochs and Iterations

There are two types of intervals in Eigen Trust:

### Iterations:

At iteration `0` peers will query their own DHT bucket for signatures to figure out what peers they are managing (See [Managers](managers.md) and [Signatures](distributing-signatures.md)). calculate the initial IVP (Iteration Validity Proof), but they will have reputation score of `0` or default bootstrap score if they are a bootstrap peer. At every other iteration their score will be calculated from the neighbours opinions received in the previous iteration. See [IVPs](../zk-circuits/iteration-validity-proofs-ivps.md) on how the proofs are aggregated.

At every iteration nodes will send Iteration Validity Proofs (IVPs) to each other. Since every peer will have `n` managers, the proof will be sent to all `n`. The receiving node will assume that at least one peer will give them the correct proof. After receiving those proofs, the peer will aggregate them and calculate the proof for their own score in that iteration. The proof will be sent immediately after they are created, which means that slow nodes will take longer to provide the proofs. This means that the IVPs will be sent asynchronously, and all we have to make sure is that the network converges during one epoch.

<figure><img src="../.gitbook/assets/Iterations-image.jpeg" alt=""><figcaption><p>Illustration of iteration intervals.</p></figcaption></figure>

### Epoch intervals:

At the start of an epoch, we are kickstarting the convergence process by calculating the `0` Iteration. Epoch last longer than iterations. i.e. as long as they need to, in order for the network to converge (24h or similar). Before every epoch, here are 2 things a peers need to do to prepare:

* Query their own DHT bucket (see how [Managers](managers.md) and [Signatures](distributing-signatures.md) work) to find signatures of the  peers they are managing. They should store them in cache, since their bucket will change over time, so we need to lock on certain peers for a particular epoch.
* Query the DHT network for the providers of the keys of the neighbours of the peers they are managing. The neighbours will be found in each signature object queried from the DHT bucket. These providers should be cached and the proofs will be requested from them in the next epoch.

Every manager will query the DHT at slightly different times (we are talking about milliseconds in difference), which means there is a potential that some peers will register a neighbouring manager that went offline during this time window. We think that the chances of this are minimal, and even if that happens, there are will be plenty of other managers to chose from.

The managers assigned at the start of the epoch will be expected to be online for the whole epoch. We could also decide to refresh the local manager set more regularly, e.g. at the beginning of each iteration.

<figure><img src="../.gitbook/assets/Epochs-image.jpeg" alt=""><figcaption><p>Illustration of Epoch interval.</p></figcaption></figure>

In order for the network to work properly, the nodes have to start the epoch at the same time, otherwise their iterations will not run in parallel, therefore blocking the whole network from progressing.

To solve that, we are NOT relying on nodes communicating to each other to agree on that time, instead the nodes will use UNIX timestamps to calculate the interval times. The epoch starting time is based on the UNIX timestamp from the beginning (00:00:00 UTC on 1 January 1970):

```rust
let until_now = now - UNIX_TIME;
let current_epoch = unix_timestamp.as_secs() / interval.as_secs();
```

If we want to calculate when is the next epoch starting:

```rust
let secs_until_next_epoch = (current_epoch + 1) * interval.as_secs() - until_now.as_secs();
```

Then we want to start an interval starting at `now + secs_until_next_epoch`, which will repeat every `interval` seconds.

```rust
let start = now.as_secs() + secs_until_next_epoch
let mut interval = time::interval_at(start, interval.as_secs());
```

Visual illustration:

```rust
0   3600s  1          2          3          4          5          6          7
|----------|----------|----------|----------|----------|----------|----------|
                                    now^    ^start of the interval
```

This way we are ensuring that every node has the same starting time on every Epoch.

