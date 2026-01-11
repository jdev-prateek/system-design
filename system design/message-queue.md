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
  * [Step 10: How partitions are assigned to brokers (and leaders are chosen)](#step-10-how-partitions-are-assigned-to-brokers-and-leaders-are-chosen)
    * [Step 10.1: Brokers (what they are)](#step-101-brokers-what-they-are)
    * [Step 10.2: Partition → broker mapping](#step-102-partition--broker-mapping)
    * [Step 10.3: Why a leader is required](#step-103-why-a-leader-is-required)
    * [Step 10.4: What followers are (preview)](#step-104-what-followers-are-preview)
    * [Step 10.5: How partition placement is decided](#step-105-how-partition-placement-is-decided)
    * [Step 10.6: How producers use this information](#step-106-how-producers-use-this-information)
    * [Step 10.7: What happens if a leader fails (high-level)](#step-107-what-happens-if-a-leader-fails-high-level)
    * [One-sentence summary (memorize)](#one-sentence-summary-memorize-1)
  * [Step 11: Replication — how partitions survive failures](#step-11-replication--how-partitions-survive-failures)
    * [Step 11.1: Replication factor (RF)](#step-111-replication-factor-rf)
    * [Step 11.2: Leader and followers](#step-112-leader-and-followers)
    * [Step 11.3: How replication works (mechanically)](#step-113-how-replication-works-mechanically)
  * [Followers replicate](#followers-replicate)
      * [Followers replicate](#followers-replicate-1)
    * [Step 11.4: In-Sync Replicas (ISR)](#step-114-in-sync-replicas-isr)
    * [Step 11.5: Why ISR exists](#step-115-why-isr-exists)
    * [Step 11.6: Acknowledging writes (high-level)](#step-116-acknowledging-writes-high-level)
    * [Step 11.7: Leader failure and recovery](#step-117-leader-failure-and-recovery)
    * [Step 11.8: Why leader must be chosen from ISR](#step-118-why-leader-must-be-chosen-from-isr)
    * [Step 11.9: What replication guarantees](#step-119-what-replication-guarantees)
    * [One-sentence summary (memorize)](#one-sentence-summary-memorize-2)
  * [Step 12: High-Water Mark (HWM) — why consumers don't see unsafe data](#step-12-high-water-mark-hwm--why-consumers-dont-see-unsafe-data)
    * [Step 12.1: The problem illustrated](#step-121-the-problem-illustrated)
    * [Step 12.2: What can go wrong without protection](#step-122-what-can-go-wrong-without-protection)
    * [Step 12.3: Kafka’s solution — High-Water Mark](#step-123-kafkas-solution--high-water-mark)
    * [Step 12.4: Visibility rule (this is the key)](#step-124-visibility-rule-this-is-the-key)
    * [Step 12.5: How HWM moves forward](#step-125-how-hwm-moves-forward)
    * [Step 12.6: Why this guarantees safety](#step-126-why-this-guarantees-safety)
    * [Step 12.7: Important clarification (very common confusion)](#step-127-important-clarification-very-common-confusion)
    * [Step 12.8: Producer acknowledgments vs HWM (brief preview)](#step-128-producer-acknowledgments-vs-hwm-brief-preview)
    * [One-sentence summary (memorize)](#one-sentence-summary-memorize-3)
  * [Step 13: Producer acknowledgments (acks) — write safety vs performance](#step-13-producer-acknowledgments-acks--write-safety-vs-performance)
    * [First principle (anchor this)](#first-principle-anchor-this)
    * [There are only 3 valid acks settings](#there-are-only-3-valid-acks-settings)
    * [1. acks = 0 (fire-and-forget)](#1-acks--0-fire-and-forget)
    * [2. acks = 1 (leader-only acknowledgment)](#2-acks--1-leader-only-acknowledgment)
    * [3. acks = all (ISR acknowledgment)](#3-acks--all-isr-acknowledgment)
    * [How acks and HWM work together (THIS is key)](#how-acks-and-hwm-work-together-this-is-key)
    * [Why Kafka separates these concerns](#why-kafka-separates-these-concerns)
    * [Real-world defaults (important)](#real-world-defaults-important)
    * [Common interview pitfall (avoid this)](#common-interview-pitfall-avoid-this)
    * [One-sentence summary (memorize)](#one-sentence-summary-memorize-4)
  * [Step 14: Consumer Groups, Partition Assignment, and Rebalancing](#step-14-consumer-groups-partition-assignment-and-rebalancing)
    * [Step 14.1: Why consumers need coordination](#step-141-why-consumers-need-coordination)
    * [Step 14.2: Consumer group (what it really is)](#step-142-consumer-group-what-it-really-is)
    * [Step 14.3: Partition assignment rule (core rule)](#step-143-partition-assignment-rule-core-rule)
    * [Step 14.4: Why partitions ≥ consumers matters](#step-144-why-partitions--consumers-matters)
    * [Step 14.5: Offset tracking (important recap)](#step-145-offset-tracking-important-recap)
    * [Step 14.6: What is rebalancing?](#step-146-what-is-rebalancing)
    * [Step 14.7: Rebalancing — step by step](#step-147-rebalancing--step-by-step)
    * [Step 14.8: Why rebalance pauses consumption](#step-148-why-rebalance-pauses-consumption)
    * [Step 14.9: Who coordinates this?](#step-149-who-coordinates-this)
    * [Step 14.10: Failure handling](#step-1410-failure-handling)
    * [Step 14.11: Why Kafka chose this model](#step-1411-why-kafka-chose-this-model)
    * [One-sentence summary (memorize)](#one-sentence-summary-memorize-5)
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


## Step 10: How partitions are assigned to brokers (and leaders are chosen)

At this point we have:

* Topics
* Partitions
* Producers choosing partitions

Now the question is:

> Where does a partition actually live, and who accepts writes for it?

### Step 10.1: Brokers (what they are)

A broker is just a server in the messaging cluster.

Each broker:

* has disk
* stores partition logs
* serves read/write requests

Example cluster:

```text
Broker 1
Broker 2
Broker 3
```

### Step 10.2: Partition → broker mapping

Each **partition is assigned to exactly one broker as its leader**.

Example:

```text
Topic: orders

Partition 0 → Broker 1
Partition 1 → Broker 2
Partition 2 → Broker 3
```

This mapping is **metadata** stored in the cluster.

Important:

* partitions are the unit of placement
* brokers do NOT store entire topics
* each broker stores some partitions from many topics

### Step 10.3: Why a leader is required

For **each partition**, one broker is chosen as the **leader**.

Rule:

> All writes go to the partition leader.

Why?

* single writer → total ordering
* simple offset assignment
* avoids conflicts

Without a leader:

* two brokers could assign offsets concurrently
* ordering would break


### Step 10.4: What followers are (preview)

Other brokers may store replicas of the same partition:

```text
Partition 0:
  Leader   → Broker 1
  Follower → Broker 2
  Follower → Broker 3  
```

But:

* only the leader accepts writes
* followers replicate from leader

(We’ll go deep into replication next.)

### Step 10.5: How partition placement is decided

When a topic is created, the system decides:

* number of partitions
* replication factor
* which broker hosts which partition

Goal:

* spread load evenly
* avoid hot brokers
* maximize fault tolerance

Example strategy:

* round-robin placement
* rack-awareness (advanced)


### Step 10.6: How producers use this information

Producers:

1. Fetch metadata from the cluster
2. Learn:
    * partition count
    * partition → leader broker mapping
3. Send writes **directly to the leader broker**

This avoids:

* central routers
* extra hops


### Step 10.7: What happens if a leader fails (high-level)

If a leader broker crashes:

* System detects failure
* One of the replicas is promoted as new leader
* Metadata is updated
* Producers refresh metadata and continue

No manual intervention required.

(We'll cover this in detail soon.)


### One-sentence summary (memorize)

> Partitions are distributed across brokers, and each partition has a single leader broker that handles 
> all writes to preserve ordering.
> 

## Step 11: Replication — how partitions survive failures

At this point we have:

* Topics
* Partitions
* Brokers
* One leader per partition

Now the problem:
> What happens if the leader broker crashes?

Without replication:

* data loss
* system downtime

So replication is not optional.


### Step 11.1: Replication factor (RF)

When creating a topic, we choose:

```text
replication.factor = N
```

Example:

```text
RF = 3
```

Meaning:

* each partition exists on 3 brokers
* 1 leader
* 2 followers


### Step 11.2: Leader and followers

For a single partition:

```text
Partition P0:
Leader   → Broker 1
Follower → Broker 2
Follower → Broker 3
```

Rules:

* Leader handles **all writes**
* Followers **replicate from leader**
* Followers never accept writes from producers

This is crucial for **ordering and correctness**.


### Step 11.3: How replication works (mechanically)

**Write arrives**

1. Producer sends message to **leader**
2. Leader:
    * appends message to its local log
    * assigns next offset

Example:

```text
Leader log: [0][1][2][3]
```

## Followers replicate

#### Followers replicate

Followers:

* pull data from leader
* append the same bytes in the same order

```text
Follower logs:
[0][1][2][3]
```

Important:

* followers do NOT reassign offsets
* they copy exactly


### Step 11.4: In-Sync Replicas (ISR)

Not all replicas are always healthy.

Kafka defines:

> ISR = replicas that are fully caught up with the leader

Example:

```text
Leader (B1):    [0][1][2][3][4]
Follower (B2):  [0][1][2][3][4]  ← in sync
Follower (B3):  [0][1][2]        ← lagging
```

ISR = `{B1, B2}`

Lagging replicas are:

* temporarily excluded
* not trusted for safety decisions


### Step 11.5: Why ISR exists

If a follower is:

* slow
* partitioned
* overloaded

You **cannot** wait for it forever.

So the system says:

> "Only replicas in ISR count toward durability."

This allows:

* progress
* availability


### Step 11.6: Acknowledging writes (high-level)

A write is considered **successful** when:

* leader has written it
* required number of ISR replicas have it

This depends on configuration (`acks`).

We'll go deep into this later.


### Step 11.7: Leader failure and recovery

Now the important part.

**Leader crashes**

```text
Leader B1 dies
```


Cluster reacts:

* Detect leader failure
* Choose new leader from ISR
* Promote follower (e.g., B2)
* Update metadata
* Producers reconnect

Result:

* no data loss (because ISR had the data)
* minimal downtime


### Step 11.8: Why leader must be chosen from ISR

If you choose a non-ISR replica:

* it may miss messages
* consumers may see gaps
* data loss possible

This is why:
> Only ISR replicas are eligible leaders
> 


### Step 11.9: What replication guarantees

With proper configuration:

* ✔ Data survives broker failures
* ✔ Ordering is preserved
* ✔ Writes are durable
* ✔ System remains available

Trade-off:

* some latency
* some complexity

### One-sentence summary (memorize)

Each partition is replicated across brokers, with one leader handling writes and followers replicating, while ISR 
ensures safe leader election and durability.


## Step 12: High-Water Mark (HWM) — why consumers don't see unsafe data

At this point we have:

* Leader
* Followers
* ISR (in-sync replicas)
* Replication in progress

Now the **core problem**:

> A leader may accept a write that followers have not yet replicated.
If the leader crashes now, should consumers have seen that message?

If yes → **data loss on leader failover**
If no → **safer reads**

Kafka chooses safety


### Step 12.1: The problem illustrated

Assume:

```text
Partition P0
ISR = {B1 (Leader), B2 (Follower)}
```

Current logs:

```text
B1 (Leader):   [0][1][2][3][4]
B2 (Follower): [0][1][2][3]
```

Message `4`:

* exists only on the leader
* not yet replicated to all ISR members


### Step 12.2: What can go wrong without protection

If Kafka allowed consumers to read offset `4`:

1. Consumer reads message `4`
2. Leader crashes
3. Follower B2 becomes leader
4. B2 does not have message 4

Now:

* consumer has read data that no longer exists
* non-repeatable read
* correctness broken

This is **not acceptable** in a durable messaging system.


### Step 12.3: Kafka’s solution — High-Water Mark

Kafka introduces:

> High-Water Mark (HWM)
= highest offset that all ISR replicas have safely stored

In our example:

```text
Leader:   [0][1][2][3][4]
Follower: [0][1][2][3]

HWM = 3
```


### Step 12.4: Visibility rule (this is the key)

> Consumers can only read messages with offset ≤ HWM

So:

* offset 3 → visible
* offset 4 → hidden

Even though:

* leader has it
* producer may have been acknowledged (depending on acks)


### Step 12.5: How HWM moves forward

When follower catches up:

```text
Follower: [0][1][2][3][4]
```

Now all ISR replicas have offset `4`.

So:

```text
HWM = 4
```

Kafka:

* advances HWM
* exposes message 4 to consumers


### Step 12.6: Why this guarantees safety

Because:

* any message visible to consumers
* is guaranteed to exist on **all ISR replicas**

So if leader crashes:

* new leader already has all visible messages
* no data disappears


### Step 12.7: Important clarification (very common confusion)

HWM controls:

* what consumers can read

It does NOT directly control:

* what producers can write
* how fast replication happens

Those are separate concerns.


### Step 12.8: Producer acknowledgments vs HWM (brief preview)

* Producer ACKs depend on `acks` setting
* Consumer visibility depends on HWM

These two are **related but not identical**.

We'll cover producer ACKs next.


### One-sentence summary (memorize)

> The High-Water Mark ensures consumers only see messages that are safely replicated on 
> all in-sync replicas, preventing data loss on leader failure.
> 


## Step 13: Producer acknowledgments (acks) — write safety vs performance

So far, we've discussed:

* partitions
* leaders and followers
* ISR
* High-Water Mark (HWM)

Now the key question:

> When does Kafka tell the producer: “Your write succeeded”?

That answer is controlled by `acks`.


### First principle (anchor this)

> `acks` controls when the PRODUCER is unblocked.
HWM controls what the CONSUMER can see.

They are related, but **not the same thing**.

### There are only 3 valid acks settings

```text
acks = 0
acks = 1
acks = all   (or -1)
```

We'll go one by one.

### 1. acks = 0 (fire-and-forget)

**What happens**

1. Producer sends message to leader
2. Producer does NOT wait for any response
3. Producer assumes success immediately

Leader behavior:

* may write the message
* may crash before writing
* producer will never know


**Guarantees**

* ❌ No durability guarantee
* ❌ No retry possible
* ❌ Messages can be lost silently


**Why this exists**

* ultra-low latency
* metrics, logs, telemetry where loss is acceptable

**Mental model**

> "I don't care if the message arrives."
> 


### 2. acks = 1 (leader-only acknowledgment)

**What happens**

1. Producer sends message to leader
2. Leader appends message to its local log
3. Leader replies ACK
4. Producer considers write successful
5. Followers replicate asynchronously


**Failure window (important)**

If:

* leader crashes after ACK
* but before followers replicate

Then:

* message is lost
* consumers will never see it

Why?

* message never reached ISR fully
* HWM never advanced


**Guarantees**

* ✔ Fast
* ✔ Most common default
* ❌ Possible data loss on leader failure

**Mental model**

> "I trust the leader."
> 


### 3. acks = all (ISR acknowledgment)

**What happens**

1. Producer sends message to leader
2. Leader writes message locally
3. Leader waits until **all ISR replicas** have replicated the message
4. Leader sends ACK to producer

Only after this:
* message is guaranteed to survive leader failure


**Guarantees**

* ✔ Strong durability
* ✔ No data loss if ISR members stay alive
* ✔ Safe with HWM

Trade-off:

* higher latency  
* dependent on slowest ISR replica

**Mental model**
> "Don't tell me success until it's safe."
> 

### How acks and HWM work together (THIS is key)

Let's align them:

| Step              | Leader   | Followers | Producer       | Consumer     |
|-------------------|----------|-----------|----------------|--------------|
| Write arrives     | Has data | Not yet   | Waiting        | Cannot see   |
| Replicated to ISR | Has data | Has data  | ACK (acks=all) | HWM advances |
| Visible           | Safe     | Safe      | Done           | Can read     |

Important insight:

> With `acks=all`, producer ACK ≈ consumer visibility

With `acks=1`, that alignment breaks.


### Why Kafka separates these concerns

Kafka deliberately separates:

* producer success
* consumer visibility

Why?

* flexibility
* performance tuning
* different workloads have different needs


### Real-world defaults (important)

Most production systems use:

```text
acks = all
min.insync.replicas = 2
replication.factor ≥ 3
```

This ensures:

* tolerate 1 broker failure
* no data loss
* reasonable performance


### Common interview pitfall (avoid this)

❌ "acks=all means consumers can read immediately"

Wrong.

✔ Consumers read based on `HWM`, not `acks`.

### One-sentence summary (memorize)

> acks defines when a producer is told a write succeeded, while the High-Water Mark defines when that write becomes 
> visible to consumers.
> 



## Step 14: Consumer Groups, Partition Assignment, and Rebalancing

Up to now:

* Producers write to partitions
* Brokers store and replicate data
* HWM controls visibility

Now we answer:
> How do consumers safely and scalably read data?
> 


### Step 14.1: Why consumers need coordination

Imagine:

* Topic orders
* 10 partitions
* 20 consumer processes

Questions:

* Who reads which partition?
* How do we avoid two consumers reading the same partition?
* What happens if one consumer dies?

This **cannot** be solved by consumers independently.

So Kafka introduces **consumer groups**.


### Step 14.2: Consumer group (what it really is)

A consumer group is:
> A logical subscriber that cooperatively consumes a topic.

Properties:
* one offset per partition per group
* partitions are exclusively owned within a group


### Step 14.3: Partition assignment rule (core rule)

> Within a consumer group, a partition is assigned to at most one consumer at a time.
>

Example:

```text
Topic: orders
Partitions: P0 P1 P2 P3

Consumer Group G1:
  C1
  C2
```

Assignment:

```text
C1 → P0, P1
C2 → P2, P3
```

This guarantees:

* no duplicate processing
* ordering per partition


### Step 14.4: Why partitions ≥ consumers matters

If:

```text
Consumers > Partitions
```

Then:

* some consumers are idle

Kafka does this intentionally to preserve ordering


### Step 14.5: Offset tracking (important recap)

Offsets are:

* stored per consumer group
* per partition
* in Kafka itself

Each consumer:

* reads from assigned partitions
* periodically commits offsets


### Step 14.6: What is rebalancing?

Now the critical part.
> Rebalancing is the process of redistributing partitions among consumers in a group.

Triggers:

* consumer joins group
* consumer leaves / crashes
* partitions added
* configuration change


### Step 14.7: Rebalancing — step by step

Let's walk through a concrete example.

**Initial state**

```text
Topic: orders
Partitions: P0 P1 P2 P3

Group G1:
C1
C2
```

Assignment:

```text
C1 → P0 P1
C2 → P2 P3
```

**A new consumer joins**

```text
Group G1:
  C1
  C2
  C3
```

Kafka triggers rebalance.


**Rebalance steps (simplified)**

* All consumers pause consumption
* Current assignments are revoked
* Coordinator computes new assignments
* New assignments are sent to consumers
* Consumers resume from last committed offsets

New assignment:

```text
C1 → P0
C2 → P1
C3 → P2, P3
```


### Step 14.8: Why rebalance pauses consumption

Because:

* partitions must not be processed by two consumers simultaneously
* offsets must remain consistent

This pause is intentional for correctness.


### Step 14.9: Who coordinates this?

Each consumer group has a group coordinator:

* a broker elected to manage the group
* tracks members and assignments
* triggers rebalances

(We'll later connect this to ZooKeeper / KRaft.)


### Step 14.10: Failure handling

If a consumer crashes:

1. Heartbeats stop
2. Coordinator detects timeout
3. Rebalance triggered
4. Partitions reassigned
5. New consumer resumes from committed offset

Result:

* no data loss
* possible reprocessing (at-least-once)


### Step 14.11: Why Kafka chose this model

Trade-offs:

* ✔ Simple correctness model
* ✔ Horizontal scalability
* ✔ Strong ordering guarantees

Cost:

* rebalance pauses
* offset management complexity

### One-sentence summary (memorize)

> Consumer groups coordinate partition ownership so that each partition is consumed by only one 
> consumer at a time, with rebalancing ensuring fault tolerance and scalability.
> 


