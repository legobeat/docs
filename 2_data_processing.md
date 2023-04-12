---
This page describes how data is processed after being pulled from AttestationStation.
---

Data that is kept in AttestationStation is submitted from one user to another.\
But EigenTrust algorithm processes opinions from one user to the whole group.

First we have to define the group, e.g.:
```
group_id = 1377
group = [peer1, peer2, peer3, peer4, peer5]
```


And lets assume we have some attestations in AS already:
```
peer1 => peer2 => 1377 => 5
peer2 => peer3 => 1377 => 7
peer4 => peer2 => 1377 => 3
```

Next, we search the AS to construct the opinion map. We do this by going through the `group` and searching for relevant attestations:

```rust
for i in 0..group.len() {
    let peer_i = group[i];
    if peer_i == null {
        continue;
    }
    for j in 0..group.len() {
        let peer_j = group[j];

        let is_null = peer_j == null;
        let is_self = peer_i == peer_j;
        if is_null || is_self  {
            continue;
        }

        let att = AS.attestations(peer_i, peer_j, group_id);
        op_map[peer_i][j] = (peer_j, att);
    }
}
```

The resulting map looks like this:
```
peer1_op => [(peer1, 0), (peer2, 5), (peer3, 0), (peer4, 0), (peer5, 0)]
peer2_op => [(peer1, 0), (peer2, 0), (peer3, 7), (peer4, 0), (peer5, 0)]
peer4_op => [(peer1, 0), (peer2, 3), (peer3, 0), (peer4, 0), (peer5, 0)]
```

This map is then passed to a filtering algorithm before being passed to EigenTrust algorithm.\
See [Dynamic Sets](../3_dynamic_sets.md).
