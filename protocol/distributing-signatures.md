---
description: >-
  This page describes how we use DHT to distribute signatures which contain
  opinion our opinions toward other peers (our neighbours).
---

# Distributing Signatures



We will use DHT network like so:

* Our assumption is that, in a DHT network, when you insert a key, the protocol will find the `n` closest peers to that key, and make them providers for that key. In our case the keys will be the public keys of everyone in the network, and the values will be latest signatures from the peers we are managing.
* When a peer connects to a DHT network, it will put its own public key as a key in the network using PUT\_VALUE command. The protocol will automatically assign the providers for the key.
* When a peer that is providing some key (in DHT network) disconnects, the DHT should update the neighbouring peers to make them providers of the keys that were previously provided by that peer.
* Peers should be able to find the providers of trust scores for a given neighbouring peer by using GET\_PROVIDERS command.
* Peers should be able to query their own buckets to see which keys they are providing. They should compute IVPs and provide them to the peers requesting it.

Requirements for values to be added into the bucket:

* Signatures that cannot be verified should not be added into the bucket.
* Only signatures that are signing messages containing the current epoch should be added.

We should assume that there will always be nodes will malicious intentions and try to provide invalid signatures. For that reason we will use mechanisms like quorums, to make sure that we get keys stored by the majority (we are assuming that at least half of the nodes providing the key are honest).\
More on how DHT works: [https://github.com/libp2p/specs/blob/master/kad-dht/README.md](https://github.com/libp2p/specs/blob/master/kad-dht/README.md)
