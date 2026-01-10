<!-- TOC -->
* [How to Design a Distributed Messaging System like Kafka?](#how-to-design-a-distributed-messaging-system-like-kafka)
  * [Resources](#resources)
  * [High-Level Architecture](#high-level-architecture)
  * [Step 0: What we are trying to store (clarify the object)](#step-0-what-we-are-trying-to-store-clarify-the-object)
  * [Step 1: Topic as a logical namespace](#step-1-topic-as-a-logical-namespace)
  * [Step 2: Why a topic cannot be a single file](#step-2-why-a-topic-cannot-be-a-single-file)
  * [Step 3: Partition = unit of storage](#step-3-partition--unit-of-storage)
  * [Step 4: How a partition is stored physically](#step-4-how-a-partition-is-stored-physically)
  * [Step 5: Partition file layout (important but simple)](#step-5-partition-file-layout-important-but-simple)
  * [Step 6: Offset assignment (critical)](#step-6-offset-assignment-critical)
  * [Step 7: Where partitions live (distributed aspect)](#step-7-where-partitions-live-distributed-aspect)
  * [Step 8: Metadata required to make this work](#step-8-metadata-required-to-make-this-work)
  * [One-sentence summary (lock this in)](#one-sentence-summary-lock-this-in)
  * [Step 9: How producers decide which partition to write to](#step-9-how-producers-decide-which-partition-to-write-to)
    * [First principle (very important)](#first-principle-very-important)
    * [There are only three valid strategies](#there-are-only-three-valid-strategies)
      * [1. Key-based partitioning (most important)](#1-key-based-partitioning-most-important)
      * [2. Round-robin partitioning (no key)](#2-round-robin-partitioning-no-key)
    * [3. Explicit partition (advanced / rare)](#3-explicit-partition-advanced--rare)
    * [Important correction (this matters)](#important-correction-this-matters)
    * [Why partition choice MUST be deterministic](#why-partition-choice-must-be-deterministic)
    * [What happens if partitions increase later?](#what-happens-if-partitions-increase-later)
  * [One-sentence summary (memorize)](#one-sentence-summary-memorize)
<!-- TOC -->

# How to Design a Distributed Messaging System like Kafka?

## Resources

* [System Design - Distributed Messaging Queue | Design Messaging Queue like Kafka, RabbitMQ](https://www.youtube.com/watch?v=oVZtzZVe9Dg)
* [Kafka: How to Design a Distributed Messaging System?](https://www.designgurus.io/course-play/grokking-the-advanced-system-design-interview/doc/messaging-systems-introduction)


## High-Level Architecture

## Step 0: What we are trying to store (clarify the object)

A messaging system stores **messages**, but not randomly.

Each message has:

* topic
* order
* durability requirement

So the **primary abstraction** is:
> An ordered, durable stream of messages
>

## Step 1: Topic as a logical namespace

What is a Topic?

A topic is NOT storage.

A topic is just:

* a name
* a logical grouping of messages

Example:

```text
orders
payments
logs
```

At this stage:

* topic has no location
* topic has no file
* topic has no machine

Think of it as a **directory name**, not a file.


## Step 2: Why a topic cannot be a single file

Suppose we store topic `orders` in one file:

```text
orders.log
```

Problems:

* single machine bottleneck
* limited throughput
* single disk
* no parallel writes
* no parallel reads

So we must split the topic.


## Step 3: Partition = unit of storage

What is a Partition?

> A partition is the actual storage unit.

Each partition is:

* append-only
* ordered
* stored independently

So a topic becomes:

```text
Topic: orders
  ├── Partition 0
  ├── Partition 1
  └── Partition 2
```

Now:

* partitions can live on different machines
* writes scale
* reads scale


## Step 4: How a partition is stored physically

Now we get concrete.

Each **partition is stored as a log**.

That means:
* new messages are appended
* nothing is modified in the middle

Example partition log:

```text
P0.log
[ offset 0 ][ offset 1 ][ offset 2 ][ offset 3 ] ...
```

This is the core design decision.

Why?

* sequential disk writes
* OS page cache friendly
* crash-safe


## Step 5: Partition file layout (important but simple)

A partition is not one giant file forever.

Instead:

```text
Partition P0
  ├── segment-000000.log
  ├── segment-000128.log
  ├── segment-000256.log
```

Each segment:

* fixed size (e.g., 1GB)
* append-only
* immutable once closed

Why segments?

* easier deletion (retention)
* faster recovery
* bounded file size


## Step 6: Offset assignment (critical)

Offsets are:

* per partition
* monotonically increasing

When a message is appended:

```text
next_offset = last_offset + 1
```

Offsets are:

* logical positions
* not array indices
* never reused

This is what enables:

* replay
* consumer independence
* crash recovery


## Step 7: Where partitions live (distributed aspect)

Each partition:

* is assigned to one broker initially
* later replicated (we’ll get to that)

Example:

```text
Broker 1 → orders-P0
Broker 2 → orders-P1
Broker 3 → orders-P2
```

This is how:

* load is distributed
* throughput scales


## Step 8: Metadata required to make this work

To store topics and partitions, the system must track:

* topic name
* number of partitions
* partition → broker mapping
* partition → segment files
* current end offset

This metadata must be:

* durable
* shared across brokers

(We'll discuss later how to store this — ZooKeeper / Raft / KRaft.)


## One-sentence summary (lock this in)

> Topics are logical names; partitions are the actual append-only logs stored on disk, split into segments and 
> distributed across brokers for scale.
>


## Step 9: How producers decide which partition to write to

At this point we have:

* Topic = logical name
* Partitions = actual logs
* Each partition lives on some broker

Now the key question:
> When a producer sends a message to a topic, how does the system choose the partition?

This decision is crucial, because it determines:

* ordering
* load balancing
* scalability

### First principle (very important)
> Partition choice happens BEFORE the message is written.
>

Once written:

* the message belongs permanently to that partition
* its order is fixed relative to other messages in that partition

### There are only three valid strategies

Kafka (and any sane messaging system) allows **exactly these patterns**.

#### 1. Key-based partitioning (most important)

**Idea**

If a message has a **key**, we use it.

Rule:

```text
partition = hash(key) % number_of_partitions
```

**Example:**

```text
Topic: orders
Partitions: 3  (P0, P1, P2)

Message:
  key = order_id = 123
```

```text
hash(123) % 3 = 1
```

→ Message goes to **Partition 1**

**Why this exists**

This guarantees:
> All messages with the same key go to the same partition.
>

Which means:

* ordering per key is preserved
* stateful processing becomes possible

Example:

* all updates for order `123`
* all events for user `456`


**Trade-off**

* If one key is “hot”, one partition becomes hot
* Load may be uneven

But correctness > perfect balance.


#### 2. Round-robin partitioning (no key)

Idea

If the producer does not care about ordering per key:

* distribute messages evenly
* maximize throughput

**Example**

Messages arrive:

```text
M1 → P0
M2 → P1
M3 → P2
M4 → P0
```

**Why this exists**

* best load balancing
* avoids hot partitions
* great for logs, metrics, events

**Trade-off**

* No ordering guarantees across messages
* Related messages may go to different partitions

### 3. Explicit partition (advanced / rare)

Producer explicitly says:

```text
send(topic="orders", partition=2)
```



Used when:

* producer knows exactly what it’s doing
* very controlled pipelines

Dangerous if misused.

### Important correction (this matters)

> The broker does NOT randomly choose partitions.

Partitioning is decided by:

* producer-side logic (partitioner)
* using cluster metadata

This avoids:

* broker bottlenecks
* extra hops

### Why partition choice MUST be deterministic

Imagine two producers sending related messages.

If partitioning were random:

* ordering breaks
* state consistency breaks

So the rule is:

> Given the same key and metadata, all producers must compute the same partition.

### What happens if partitions increase later?

This is a subtle but important point.

If you go from:

```text
3 partitions → 6 partitions
```

Then:

```text
hash(key) % partitions
```

changes.

Result:

* same key may map to a different partition
* ordering per key breaks across time

This is why:

* partition count is usually fixed early
* increasing partitions is done carefully

## One-sentence summary (memorize)

> Producers choose partitions deterministically—usually by hashing the message key—to preserve
> ordering while enabling parallelism.
> 


