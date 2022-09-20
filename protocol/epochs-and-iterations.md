---
description: >-
  This page explains the intervals at which the network ticks. On every tick,
  requests are made between peers, and rules enforced.
---

# Epochs and Iterations

There are two types of intervals in Eigen Trust:

### Iterations - short intervals:

They are ticking every \~10s and in each tick one iteration of Eigen Trust algorithm is calculated. The number of tick is fixed (depending on how many iterations we need for convergence).

At every iteration nodes will send requests to each other for Iteration Validity Proofs (IVPs). Since every peer will have `n` managers, the requests will be sent to all `n`, assuming that at least one of them will provide correct proof, and be on time. After receiving those proofs, the peer will aggregate them and calculate the proof for their own score in that iteration. The proof will be cached and served in the next iteration to whomever makes a request to them.

<figure><img src="../.gitbook/assets/Iterations-image.jpeg" alt=""><figcaption><p>Illustration of iteration intervals.</p></figcaption></figure>

### Epochs - long intervals:

At the start of an epoch, we are kickstarting the convergence process by starting the Iteration interval. Epoch last much longer than iterations. e.g. one epoch might last 24 hours, while the whole 10 iteration can last 30 minutes. So, most of the time available in the epoch, the network will do nothing. Allowing the nodes to prepare for the next epoch, respond to RPC requests or similar.

<figure><img src="../.gitbook/assets/Epochs-image.jpeg" alt=""><figcaption><p>Illustration of Epoch interval.</p></figcaption></figure>

In order for the network to work properly, the nodes have to start the epoch at the same time, otherwise their iterations will not run in parallel, therefore blocking the whole network from progressing.

To solve that, every node will use UNIX timestamps to calculate the interval times. The epoch starting time is based on the UNIX timestamp from the beginning (00:00:00 UTC on 1 January 1970):

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

### Future optimisation

Not requesting IVPs from every manager of the peer N would reduce the computational load on every node. We can create a third class of intervals that are running within the Iteration interval. We can call it Session intervals - Instead of requesting the IVPs from every manager, we request from just one, at each session.

The process of choosing to which manager we will send requests to should be calculated simply - e.g. sorting them by public keys.

Also, instead of sending requests at the beginning of each session, we could broadcast the IVP as soon as we receive the required IVPs from our neighbours. This way the network will converge much faster, and we will use interval as timeouts - in case the current manager doesn't reply we will send request to another manager.

TBA:

* Better illustrations
* Explain future optimisations more clearly
* Explain that the managers should calculate initial proof in the iteration 0 after they query their bucket - so no requests in iteration 0, the requests start at iteration 1
* Explain what happens after the last iteration - preparing for the next epoch (collecting the manager set from the DHT and the signatures they hold, to figure out who is tinkering with signatures)
* Explain that every manager will query the DHT at slightly separate times, which means that two managers of peer n could get different set of the managers for peer `n` neighbours
