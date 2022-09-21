---
description: >-
  This page explains the intervals at which the network ticks. On every tick,
  requests are made between peers, and rules enforced.
---

# Epochs and Iterations

There are two types of intervals in Eigen Trust:

### Iterations - short intervals:

They are ticking every \~10s and in each tick one iteration of Eigen Trust algorithm is calculated. The number of ticks is fixed (depending on how many iterations we need for convergence).

At every iteration nodes will send requests to each other for Iteration Validity Proofs (IVPs). Since every peer will have `n` managers, the requests will be sent to all `n`, assuming that at least one of them will provide correct proof, and be on time. After receiving those proofs, the peer will aggregate them and calculate the proof for their own score in that iteration. The proof will be cached and served in the next iteration to whomever makes a request to them.

At iteration `0` peers will query their own DHT bucket for signatures to figure out what peers they are managing (See [Managers](managers.md) and [Signatures](distributing-signatures.md)). calculate the initial IVP (Iteration Validity Proof), but they will have reputation score of `0` or default bootstrap score if they are a bootstrap peer. At every other iteration their score will be calculated from the neighbours opinions received in the previous iteration. See [IVPs](../zk-circuits/iteration-validity-proofs-ivps.md) on how the proofs are aggregated.

<figure><img src="../.gitbook/assets/Iterations-image.jpeg" alt=""><figcaption><p>Illustration of iteration intervals.</p></figcaption></figure>

### Epochs - long intervals:

At the start of an epoch, we are kickstarting the convergence process by starting the Iteration interval. Epoch last longer than iterations. e.g. one epoch might last 1 hour, while the whole 10 iteration can last 30 minutes. In the other half the peers will prepare for the next epoch. There are 3 things a peers need to do to prepare for the epoch:

* Query their own DHT bucket (see how [Managers](managers.md) and [Signatures](distributing-signatures.md) work) to find signatures of the  peers they are managing. They should store them in cache, since their bucket will change over time, so we need to lock on certain peers for a particular epoch.
* Query the DHT network for the providers of the keys of the neighbours of the peers they are managing. The neighbours will be found in each signature object queried from the DHT bucket. These providers should be cached and the proofs will be requested from them in the next epoch.
* Query the signatures of all the neighbours of the peers they are managing, and use DHT Quorum to get the correct value. These signatures will be cached and the message hashes derived from them will be passed in every proof that neighbour provides, to make sure they are using the latest signature and not some older one.

Every manager will query the DHT at slightly different times (we are talking about milliseconds in difference), which means there is a potential that some peers will register a neighbouring manager that went offline during this time window. We think that the chances of this are minimal, and even if that happens, there are will be plenty of other managers to chose from.

The managers assigned at the start of the epoch will be expected to be online for the whole epoch. We could also decide to refresh the local manager set more regularly, e.g. at the beginning of each iteration.

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

### Future optimisation - Proposal

Not requesting IVPs from every manager of the peer N would reduce the computational load on every node. We can create a third class of intervals that are running within the Iteration interval. We can call it Session intervals - Instead of requesting the IVPs from every manager, we request from just one, at each session.

The process of choosing to which manager we will send requests to should be calculated simply - e.g. sorting them by public keys x value.

Also, instead of sending requests at the beginning of each session, we could broadcast the IVP as soon as we receive the required IVPs from our neighbours - on every response we will check if we got the proof from every neighbour, in which case we will calculate the IVP for the next iteration and broadcast it immediately.

This way the network will converge much faster, and we will use interval as timeouts - in case the current manager doesn't reply we will send request to another manager.
