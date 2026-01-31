<!-- TOC -->
* [Designing a Dynamo-Style Distributed Key–Value Store](#designing-a-dynamo-style-distributed-keyvalue-store)
  * [Resource](#resource)
  * [Goal](#goal)
  * [High-Level Architecture](#high-level-architecture)
  * [1. Storage Engine: LSM Trees](#1-storage-engine-lsm-trees)
    * [Why LSM Trees?](#why-lsm-trees)
    * [Write Path](#write-path)
    * [Trade-offs](#trade-offs)
  * [2. Partitioning: Consistent Hashing](#2-partitioning-consistent-hashing)
    * [How it works](#how-it-works)
    * [Ownership Rule](#ownership-rule)
    * [Virtual Nodes (vnodes)](#virtual-nodes-vnodes)
    * [Benefits](#benefits)
  * [3. Replication: Leaderless Replication](#3-replication-leaderless-replication)
    * [Key Properties](#key-properties)
    * [Quorum Parameters](#quorum-parameters)
    * [Trade-offs](#trade-offs-1)
  * [4. Request Coordination: Coordinator Logic](#4-request-coordination-coordinator-logic)
    * [Key Idea](#key-idea)
    * [Write Flow (PUT(key, value))](#write-flow-putkey-value)
    * [Read Flow (GET(key))](#read-flow-getkey)
    * [Why Coordinator Logic Matters](#why-coordinator-logic-matters)
  * [5. Failure Detection: Gossip Protocol](#5-failure-detection-gossip-protocol)
    * [What Gossip Does](#what-gossip-does)
    * [Why Gossip?](#why-gossip)
    * [Trade-offs](#trade-offs-2)
  * [6. Concurrent Writes: Last-Write-Wins (LWW)](#6-concurrent-writes-last-write-wins-lww)
    * [How LWW Works](#how-lww-works)
    * [Why LWW?](#why-lww)
    * [Trade-offs](#trade-offs-3)
  * [7. Consistency Maintenance](#7-consistency-maintenance)
    * [7.1 Read Repair](#71-read-repair)
    * [7.2 Anti-Entropy (Background Repair)](#72-anti-entropy-background-repair)
  * [Putting It All Together](#putting-it-all-together)
<!-- TOC -->

# Designing a Dynamo-Style Distributed Key–Value Store

## Resource

* Neetcode - Design a Key-Value Store

## Goal

Design a **highly available, write-optimized, distributed key–value store**, inspired by **Amazon Dynamo**, that can 
tolerate node failures and network partitions while remaining operational.

The system prioritizes:

* availability
* scalability
* operational simplicity

over strong consistency.

## High-Level Architecture

The system consists of the following core components:

1. Storage Engine (LSM Trees)
2. Partitioning (Consistent Hashing)
3. Replication (Leaderless Replication)
4. Request Coordination (Coordinator Logic)
5. Failure Detection (Gossip Protocol)
6. Concurrent Write Handling (Last-Write-Wins)
7. Consistency Maintenance (Read Repair & Anti-Entropy)

Each component is described below.

## 1. Storage Engine: LSM Trees

**Choice**

Use **Log-Structured Merge Trees (LSM Trees)** as the storage engine.

### Why LSM Trees?

* Optimized for **write-heavy workloads**
* Sequential disk writes → high throughput
* Works well with commodity hardware

### Write Path

1. Append write to **Write-Ahead Log (WAL)** for durability
2. Insert key-value into **MemTable** (in-memory, sorted)
3. Periodically flush MemTable to disk as immutable **SSTables**
4. Background **compaction** merges SSTables

### Trade-offs

* Reads may touch multiple SSTables
* Compaction causes write amplification
* Deletes require tombstones


## 2. Partitioning: Consistent Hashing

**Choice**

Use **consistent hashing** to partition data across nodes.

### How it works

* Hash space is treated as a ring
* Each node is assigned one or more positions on the ring
* Keys are hashed to a position on the same ring

### Ownership Rule

> A key is owned by the **first node encountered when moving clockwise** from the key’s hash.
> 

### Virtual Nodes (vnodes)

* Each physical node owns multiple virtual positions
* Improves load balancing
* Smooths rebalancing when nodes join/leave

### Benefits

* Minimal key movement on topology changes
* No centralized directory
* Scales horizontally



## 3. Replication: Leaderless Replication

**Choice**

Use **leaderless replication** (no single primary).


### Key Properties

* Any node can accept reads or writes
* Data is replicated to `N` nodes
* System remains available during node failures


### Quorum Parameters

* `N`: replication factor
* `W`: write quorum
* `R`: read quorum


Typical configuration:

```text
N = 3, W = 2, R = 2
```

This allows:

* writes to succeed if 2 replicas are reachable
* reads to see the latest value with high probability

### Trade-offs

* Eventual consistency
* Requires conflict resolution
* Replicas may temporarily diverge



## 4. Request Coordination: Coordinator Logic

### Key Idea

There is **no dedicated router**.

> Any node receiving a request acts as the coordinator for that request.

This avoids single points of failure.

### Write Flow (PUT(key, value))

1. Client sends request to **any node**
2. That node becomes the **coordinator**
3. Coordinator:
    * hashes the key
    * finds `N` responsible replicas using the hash ring
4. Coordinator sends write requests to all `N` replicas
5. Each replica:
    * appends to WAL
    * updates its MemTable
6. Coordinator waits for `W` acknowledgements
7. Once `W` ACKs are received, coordinator returns success to client


### Read Flow (GET(key))

1. Client sends request to any node
2. That node becomes the coordinator
3. Coordinator:
    * hashes the key
    * sends read requests to `R` replicas
4. Coordinator compares versions (timestamps)
5. Returns the latest value (LWW)
6. Optionally triggers read repair for stale replicas


### Why Coordinator Logic Matters

* Decouples clients from cluster topology
* Enables retries on failures
* Centralizes quorum logic per request


## 5. Failure Detection: Gossip Protocol

**Choice**

Use **gossip protocol** instead of a centralized coordinator (e.g., ZooKeeper).

### What Gossip Does

* Nodes periodically exchange membership information
* Failure suspicion spreads gradually
* Each node maintains an eventually consistent view of the cluster

### Why Gossip?

* Fully decentralized
* Scales with cluster size
* No single point of failure

### Trade-offs

* Failure detection is probabilistic
* Membership views may be slightly stale

This is acceptable in an eventually consistent system.


## 6. Concurrent Writes: Last-Write-Wins (LWW)

**Problem**

With leaderless replication, concurrent writes to the same key may occur.

**Choice**

Use **Last-Write-Wins (LWW)** for conflict resolution.

### How LWW Works

* Each write is tagged with a timestamp
* When replicas see multiple versions:
    * the version with the **largest timestamp wins**
    * older versions are discarded


### Why LWW?

* Very simple to implement
* No vector clock metadata
* No client-side conflict resolution
* Predictable operational behavior


### Trade-offs

* Concurrent updates may be lost
* Clock skew can cause anomalies
* Deletes may overwrite newer writes

This is an **explicit design trade-off** in favor of simplicity.  


## 7. Consistency Maintenance

Leaderless replication does not automatically converge.

Two mechanisms are required to ensure eventual consistency.

### 7.1 Read Repair

**What it is**

Fixes stale replicas **during read operations**.

**How it works**

1. Coordinator reads from R replicas
2. Detects if some replicas have older versions
3. Writes the latest version back to stale replicas

**Benefits**

* Cheap
* Piggybacks on normal reads
* Repairs frequently accessed ("hot") keys


### 7.2 Anti-Entropy (Background Repair)

**Problem**

Read repair does not fix **cold data** (rarely read keys).

**Solution**

Use **anti-entropy** via background synchronization.

**How it works**

* Nodes periodically compare data summaries (e.g., Merkle trees)
* Detect divergent key ranges
* Exchange and repair mismatched data

**Benefits**

* Ensures full convergence over time
* Repairs data even without reads


## Putting It All Together

Why the System Works

* LSM Trees → fast, durable writes
* Consistent hashing → scalable partitioning
* Leaderless replication → high availability
* Coordinator logic → request orchestration
* Gossip → decentralized failure detection
* LWW → simple conflict resolution
* Read repair + anti-entropy → eventual consistency

