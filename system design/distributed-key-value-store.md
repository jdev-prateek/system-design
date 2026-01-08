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
  * [Overview](#overview)
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





















## Overview

1. Indexing data
2. Replication
3. Partitioning
4. Node Failure
5. Concurrent Writes

