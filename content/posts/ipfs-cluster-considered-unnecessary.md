---
title: ipfs-cluster Considered Unnecessary
date: 2018-09-15
---

`ipfs-cluster` is a layer built upon IPFS, which defines its goal as "pinset orchestration". More specifically, it uses [raft algorithm](https://raft.github.io/) to track IPFS peers and pinned file blocks, providing the ability to decide 1) how to split a large file into blocks 2) how these blocks are distributed among IPFS nodes and command those nodes to actually pin them. In `ipfs-cluster`, raft is backed by `libp2p`, a library abstracts out complex network topologies.

But after thinking for a while, I still can't seem to find a suitable use case for it. In every scenario I could imagine, there's a better solution. To elaborate my points:

- Inside data center: Direct network connection between nodes is nearly always possible, so `libp2p` seems not very necessary. Moreover, we already have many mature distributed storage solutions in data center today. It would be easier and more effective to port those solutions to IPFS as storage backends than reinventing the wheel.
- At home: It's rare for a home user to manage so many storage nodes. A NAS or IPFS over NAS is sufficient.
- Between data centers: Raft would suffer from performance issues, as at least 2 RTTs are required for a log to be committed(we might have no better choice for consensus algorithm in this case, though). But why strong consistency is required between data centers, to store information like pins? Generally the best practice to synchronize between data centers is to aim eventual consistency.
- P2P storage network management: Peers may come and go in a P2P network, again raft can't handle this kind of situation well, as for raft to work, all peers should be known to each other.

Meanwhile, the functionality of `ipfs-cluster` intersects [`Filecoin`](https://filecoin.io/), the core project in IPFS ecosystem, so I assume `ipfs-cluster` be a incubator project and finally replaced. But I might be wrong. If you have any thoughts, welcome to join the discussion [here](https://github.com/ipfs/ipfs-cluster/issues/538).