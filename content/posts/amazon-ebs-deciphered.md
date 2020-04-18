---
title: Amazon EBS deciphered
date: 2020-04-18
---

Recently Amazon has published a paper about the design of the control plane of their block storage service, EBS, we could finally peek what's under the hood.

The paper is called [_Millions of Tiny Databases_](https://scholar.google.com/scholar?q=Millions+of+Tiny+Databases). It introduces _Physalia_, "a transactional key-value store optimized for use in large-scale cloud control planes", "offer both high availability and strong consistency to millions of clients".

To understand what lead to such a kind of control plane design, we need to first look at EBS data plane, which I personally find more interesting. Unlike many storage systems, EBS data plane doesn't employ a quorum-based replication scheme, like Raft or Paxos, but instead uses [_chain replication_](https://scholar.google.com/scholar?q=Chain+Replication+for+Supporting+High+Throughput+and+Availability). 

![chain-replication](/chain-replication.jpg)

Chain replication arranges peers in a replication group into a chain. Clients would send update queries to the head node, head node then forward the queries to the next node in chain in a FIFO manner, and finally to the tail node. The tail node would send an acknowledgment back to the client, because when the tail finishes processing an update, all nodes in the chain have already processed the query. In contrast, read queries are solely served by the tail node.

Chain replication could tolerate N-1 node failure, where N is the total number of nodes in chain. It claims to have better throughput than quorum-based schemes because there's not a "leader" node as the bottleneck. Surprisingly Amazon chooses N=2, as all figures in the paper have 2 replicas, and I get confirmations [here](https://scholar.google.com/scholar?q=Amazon+Aurora%3A+Design+Considerations+for+High+Throughput+Cloud-Native+Relational+Databases) and [here](https://pdos.csail.mit.edu/6.824/notes/l-aurora.txt). I guess this is for better latency.

Chain replication requires a control plane to group storage nodes into chains, so comes Physalia. The key observation of Physalia is that not all keys are needed to be available to all clients for block storage services. For example, it's OK for storage to die when the corresponding virtual machine is dead. Thus Physalia employs a datacenter-aware method to place metadata replicas both near clients and have enough tolerance to failures.

Amazon chooses 7 for metadata replica number, to balance durability, latency, availability and resource consumption. The replica is small, stores data for a partition key, whereas every block storage volume is also assigned a unique partition key, so I guess a configurable N volume per replica. And every replica could be moved around "within a minute", likely tens to hundreds of MBs. These 7 replicas form a "cell", to serve queries of corresponding volumes. For the whole EBS, there're "millions of tiny databases" in its control plane.

More references:

[1] [AWS re:Invent keynote speech, begins at 40:30](https://youtu.be/OdzaTbaQwTg?t=2430)

[2] [Paper author's blog post](https://brooker.co.za/blog/2020/02/17/physalia.html)
