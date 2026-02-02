<!-- TOC -->
* [Q-1 What is Partition Tolerance?](#q-1-what-is-partition-tolerance)
* [Q-2 What is CAP theorem?](#q-2-what-is-cap-theorem)
  * [Why Partition Tolerance is mandatory](#why-partition-tolerance-is-mandatory)
  * [Let’s see the choice (ELI5)](#lets-see-the-choice-eli5)
    * [Case 1: Choose Consistency](#case-1-choose-consistency)
    * [Case 2: Choose Availability](#case-2-choose-availability)
  * [Important correction (many people get this wrong)](#important-correction-many-people-get-this-wrong)
  * [Real-world mapping (intuition)](#real-world-mapping-intuition)
* [Q-3 What are two common Replication Models?](#q-3-what-are-two-common-replication-models)
  * [Primary–Replica (Leader–Follower)](#primaryreplica-leaderfollower)
  * [Quorum-based Replication (Leaderless / Multi-Replica)](#quorum-based-replication-leaderless--multi-replica)
* [Q-4 How to tune consistency in Leaderless Replication](#q-4-how-to-tune-consistency-in-leaderless-replication)
  * [1. The Variables](#1-the-variables)
  * [2. The Scenario: `R + W ≤ N`](#2-the-scenario-r--w--n)
  * [3. Why would anyone do this?](#3-why-would-anyone-do-this)
  * [4. The "Fixed" Formula (Strong Consistency)](#4-the-fixed-formula-strong-consistency)
* [Q-5 What are Bloom Filters?](#q-5-what-are-bloom-filters)
  * [1. The problem (ELI5)](#1-the-problem-eli5)
  * [2. What a Bloom Filter is (ELI5)](#2-what-a-bloom-filter-is-eli5)
  * [3. The light-bulb board (visual example)](#3-the-light-bulb-board-visual-example)
  * [4. Adding an item (example 1)](#4-adding-an-item-example-1)
  * [5. Adding another item (example 2)](#5-adding-another-item-example-2)
  * [6. Checking if an item exists (example)](#6-checking-if-an-item-exists-example)
  * [7. Checking something that is NOT there](#7-checking-something-that-is-not-there)
  * [8. The golden rule (important)](#8-the-golden-rule-important)
  * [9. Why false positives are okay](#9-why-false-positives-are-okay)
* [Q-6 What are LSM Trees?](#q-6-what-are-lsm-trees)
  * [Step 1: The problem LSM Trees solve (ELI5)](#step-1-the-problem-lsm-trees-solve-eli5)
  * [Step 2: The key rule (lock this in)](#step-2-the-key-rule-lock-this-in)
  * [Step 3: Start with memory (MemTable)](#step-3-start-with-memory-memtable)
    * [MemTable (sorted in RAM)](#memtable-sorted-in-ram)
  * [Step 4: MemTable fills up → flush to disk](#step-4-memtable-fills-up--flush-to-disk)
  * [Step 5: First SSTable on disk](#step-5-first-sstable-on-disk)
  * [Step 6: More writes come in](#step-6-more-writes-come-in)
  * [Step 7: Flush again → another SSTable](#step-7-flush-again--another-sstable)
  * [Step 8: How READ works (very important)](#step-8-how-read-works-very-important)
  * [Step 9: Bloom Filters help speed reads](#step-9-bloom-filters-help-speed-reads)
  * [Step 10: Deletes (tombstones)](#step-10-deletes-tombstones)
  * [Step 11: Problem: Too many SSTables](#step-11-problem-too-many-sstables)
  * [Step 12: Compaction (cleanup)](#step-12-compaction-cleanup)
  * [Step 13: Why LSM Trees are fast](#step-13-why-lsm-trees-are-fast)
  * [Step 14: One-sentence ELI5 summary (memorize this)](#step-14-one-sentence-eli5-summary-memorize-this)
* [Q-7 MemTable and SSTable: Structure and Layout](#q-7-memtable-and-sstable-structure-and-layout)
  * [What a MemTable looks like (in memory)](#what-a-memtable-looks-like-in-memory)
    * [How it’s actually implemented](#how-its-actually-implemented)
    * [What happens on write](#what-happens-on-write)
  * [2. What happens when MemTable is flushed](#2-what-happens-when-memtable-is-flushed)
  * [3. What an SSTable looks like (on disk)](#3-what-an-sstable-looks-like-on-disk)
    * [SSTable (high-level layout)](#sstable-high-level-layout)
    * [3.1 Data Blocks (the actual data)](#31-data-blocks-the-actual-data)
    * [3.2 Index Block (how SSTable is searched)](#32-index-block-how-sstable-is-searched)
    * [3.3 Bloom Filter (quick "not here" check)](#33-bloom-filter-quick-not-here-check)
    * [3.4 Footer](#34-footer)
  * [4. SSTable example (full picture)](#4-sstable-example-full-picture)
  * [5. How reads actually happen (step-by-step)](#5-how-reads-actually-happen-step-by-step)
  * [6. Deletes (tombstones) in MemTable and SSTable](#6-deletes-tombstones-in-memtable-and-sstable)
  * [7. Key difference (very important)](#7-key-difference-very-important)
* [Q-8 What is Consistent hashing?](#q-8-what-is-consistent-hashing)
  * [Consistent hashing: the core idea](#consistent-hashing-the-core-idea)
  * [The hash space (horizontal line)](#the-hash-space-horizontal-line)
  * [Place shards on the line](#place-shards-on-the-line)
  * [Place keys on the same line](#place-keys-on-the-same-line)
  * [Assign keys to shards (very important)](#assign-keys-to-shards-very-important)
  * [Adding a shard (this is where consistent hashing shines)](#adding-a-shard-this-is-where-consistent-hashing-shines)
  * [So far so good… but here's the problem](#so-far-so-good-but-heres-the-problem)
  * [Enter virtual nodes (this is the key)](#enter-virtual-nodes-this-is-the-key)
  * [Setup: hash space and virtual nodes](#setup-hash-space-and-virtual-nodes)
  * [Ownership rule (recap)](#ownership-rule-recap)
  * [Initial ownership ranges](#initial-ownership-ranges)
  * [Add a new physical node D](#add-a-new-physical-node-d)
  * [What moves when node D is added?](#what-moves-when-node-d-is-added)
  * [Ownership after adding D](#ownership-after-adding-d)
  * [Remove physical node B](#remove-physical-node-b)
  * [What happens to B’s ranges?](#what-happens-to-bs-ranges)
  * [Why virtual nodes make this smooth](#why-virtual-nodes-make-this-smooth)
  * [Key insight (this is the “aha”)](#key-insight-this-is-the-aha)
* [Q-9 What is Gossip Protocol?](#q-9-what-is-gossip-protocol)
  * [Step 0: Bootstrap (master list loaded)](#step-0-bootstrap-master-list-loaded)
  * [Step 1: Node A starts](#step-1-node-a-starts)
  * [Step 2: A picks initial peers](#step-2-a-picks-initial-peers)
  * [Step 3: A contacts C and E](#step-3-a-contacts-c-and-e)
  * [Step 4: What do C and E know at this moment?](#step-4-what-do-c-and-e-know-at-this-moment)
  * [Step 5: C and E reply to A (this is the key fix)](#step-5-c-and-e-reply-to-a-this-is-the-key-fix)
  * [Step 6: A’s state after first gossip round](#step-6-as-state-after-first-gossip-round)
  * [Step 7: Next gossip round (time passes)](#step-7-next-gossip-round-time-passes)
  * [Step 8: A contacts B for the first time](#step-8-a-contacts-b-for-the-first-time)
  * [Step 9: A merges again](#step-9-a-merges-again)
  * [Step 10: How D is eventually discovered (or marked dead)](#step-10-how-d-is-eventually-discovered-or-marked-dead)
  * [One-sentence corrected summary](#one-sentence-corrected-summary)
* [Q-7 What are Vector Clocks?](#q-7-what-are-vector-clocks)
* [Q-8 What are different cache eviction policies?](#q-8-what-are-different-cache-eviction-policies)
* [Q-9 What are different Rate Limiting Algorithms?](#q-9-what-are-different-rate-limiting-algorithms)
<!-- TOC -->

# Q-1 What is Partition Tolerance?

Partition Tolerance is the ability of a distributed system to survive a communications
breakdown between its internal servers.




# Q-2 What is CAP theorem?

When a network partition happens, you must choose:

* Consistency OR
* Availability

You cannot have both. Partition tolerance is not optional in distributed systems.

**Note:** ✅ CAP is only about distributed systems

## Why Partition Tolerance is mandatory

If your system has:

* more than one machine
* connected by a network

Then:

* network failures will happen

So CAP really means:

```text
During a partition:
Choose C or A
```

Not all three.

## Let’s see the choice (ELI5)

### Case 1: Choose Consistency

Network breaks.

User asks Computer A:

```text
GET balance
```

Computer A says:

> "I'm not sure what B has. I'll wait."
>

Result:

* ❌ User may get no response
* ✅ Data stays correct

This is CP (Consistency + Partition Tolerance).

### Case 2: Choose Availability

Network breaks.

User asks Computer A:

```text
GET balance
```

Computer A says:
> "Here's what I have."

Even if B has different data.

Result:

* ✅ User gets a response
* ❌ Data may be stale or inconsistent

This is AP (Availability + Partition Tolerance).

## Important correction (many people get this wrong)

CAP does NOT say:
> "You can never have all three"

It says:
**During a partition**, you must choose

When the network is healthy:

* you can have C + A

## Real-world mapping (intuition)

| System choice | What it values                  |
|---------------|---------------------------------|
| CP            | Correctness over responsiveness |
| AP            | Responsiveness over correctness |

Neither is "better". They solve different problems.





# Q-3 What are two common Replication Models?

1. Primary–Replica (Leader–Follower)
2. Quorum-based Replication (Leaderless / Multi-Replica)

## Primary–Replica (Leader–Follower)

How it works

* One node is the Primary (Leader)
* All writes go to the Primary
* Replicas copy data from the Primary
* Reads can go to:
    * Primary only (stronger consistency)
    * Replicas (faster, possibly stale)

Key properties

* Simple mental model
* Easy conflict resolution (single writer)
* Replication lag is common
* Consistency depends on where reads go

Typical systems

* MySQL primary/replica
* PostgreSQL streaming replicas
* Redis primary + replica
* Kafka partitions (leader/followers)

Consistency story

* Strong consistency if reads go to primary
* Eventual consistency if reads go to replicas

## Quorum-based Replication (Leaderless / Multi-Replica)

How it works

* No single “primary” for correctness
* Data is written to multiple replicas
* Reads consult multiple replicas
* Majority agreement determines the result

Key properties

* No single leader bottleneck
* More complex logic
* Tunable consistency
* Handles failures gracefully

Typical systems

* Dynamo / DynamoDB (conceptually)
* Cassandra
* Riak

Consistency story

* Controlled by `R` and `W`
* If `R + W > N` → latest write is guaranteed
* If not → stale reads possible




# Q-4 How to tune consistency in Leaderless Replication

In distributed systems (like Cassandra or DynamoDB), this specific formula (`R + W ≤ N`) means you
are **NOT guaranteed to see the latest data**.

Here is the ELI5 breakdown using The Notebook Analogy.

## 1. The Variables

* N (Total Copies): You have 3 notebooks (Replicas) where you store secrets.
* W (Write Quorum): When you want to save a secret, you write it in W of them.
* R (Read Quorum): When you want to read a secret, you check R of them.

## 2. The Scenario: `R + W ≤ N`

Let's say `N=3` (3 notebooks). You decide: "I want to be fast, so I will only write to 1 notebook (`W=1`) and
read from 1 notebook (`R=1`)."
* The Math: `1+1=2`. And `2≤3`. The formula holds.

What happens?

* **The Write:** You write "The sky is Green" into **Notebook A**. (You ignore B and C because `W=1`).
* **The Read:** Your friend comes along and picks **Notebook C** to read (`R=1`).
* **The Result:** Notebook C is blank (or says "The sky is Blue"). Your friend sees stale (old) data.

## 3. Why would anyone do this?

If this formula leads to errors, why do engineers use it? Speed.

* Writing to 1 notebook is faster than writing to 3.
* Reading from 1 notebook is faster than comparing 3.
* This is called **Eventual Consistency**. You are betting that eventually Notebook A will copy
  the data to B and C in the background. But for a few milliseconds, the data is wrong.

## 4. The "Fixed" Formula (Strong Consistency)

If you want to guarantee that your friend always sees "The sky is Green," you must use the opposite formula:

>
> R+W > N
>

* **Example:** `N=3`. You write to 2 (`W=2`). You read from 2 (`R=2`).
* **Math:** `2+2=4`. And `4>3`.
* **Why it works:** Because 4 is bigger than the total number of notebooks, there must be an overlap.
  At least one of the notebooks you read must be one of the notebooks I wrote to.

| Formula       | Name                        | Meaning                                                                      | Use Case                        |
|:--------------|:----------------------------|:-----------------------------------------------------------------------------|:--------------------------------|
| $R + W \le N$ | Weak / Eventual Consistency | The Read group and Write group might not overlap. Risk of stale data.        | High speed, Likes, Feed counts. |
| $R + W > N$   | Strong Consistency          | The Read group and Write group are guaranteed to overlap. Always fresh data. | Passwords, Payments, Inventory. |





# Q-5 What are Bloom Filters?

## 1. The problem (ELI5)

Imagine you have a huge toy box with millions of toys.

A kid asks:

> "Is the red dinosaur toy inside?"

You don't want to:

* open the box every time (slow)
* guess randomly (wrong)

You want a quick hint that tells you:

* ❌ “Definitely NOT inside”
* 🤔 “Maybe inside”

That hint is a Bloom Filter.

## 2. What a Bloom Filter is (ELI5)

> A Bloom Filter is a quick checker that can tell you
"Definitely no" or "Maybe yes."
>

It never says:

* "Yes, for sure"

## 3. The light-bulb board (visual example)

Imagine a board with 8 light bulbs:

```text
[ 0 0 0 0 0 0 0 0 ]
```

* `0` = OFF
* `1` = ON

All bulbs start OFF.

## 4. Adding an item (example 1)

You add the word:

```text
"apple"
```

You pass "apple" through 2 magic machines (hash functions).

They say:

* Turn ON bulb #2
* Turn ON bulb #5

Board becomes:

```text
[ 0 0 1 0 0 1 0 0 ]
```

You did not store "apple" — you only flipped bulbs.

## 5. Adding another item (example 2)

Add:

```text
"banana"
```

Magic machines say:

* Turn ON bulb #3
* Turn ON bulb #5

Board becomes:

```text
[ 0 0 1 1 0 1 0 0 ]
```

Note:

* Bulb #5 was already ON — that's fine.

## 6. Checking if an item exists (example)

Someone asks:
> "Do you have ‘apple’?"
>

You:

1. Run "apple" through the same machines
2. Check bulbs #2 and #5

Both are ON →
👉 Maybe yes

You now go check the real toy box.

## 7. Checking something that is NOT there

Someone asks:
> "Do you have ‘dragon’?"
>

Machines say:

* Check bulb #1
* Check bulb #6

Board:

```text
[ 0 0 1 1 0 1 0 0 ]
```

Bulb #1 is OFF →
👉 Definitely NOT there

You skip checking the box entirely.

## 8. The golden rule (important)

| Bloom Filter answer | Meaning                |
|---------------------|------------------------|
| ❌ No                | Definitely not present |
| 🤔 Yes              | Might be present       |

It never lies about "no."
It can lie about "yes."

This lie is called a false positive.

## 9. Why false positives are okay

If Bloom Filter says:

> "Maybe yes"

Worst case:

* You checked the box unnecessarily

If Bloom Filter says:

> "No"

You saved time and avoided work.

So Bloom Filters trade:

* tiny uncertainty
* for huge speed gains





# Q-6 What are LSM Trees?

## Step 1: The problem LSM Trees solve (ELI5)

Imagine you keep a notebook on disk.

Every time you want to update a value, you must:

* open the notebook
* erase a line
* rewrite it

Disk hates this.


So databases asked:

> "What if we never erase and only append?"

That idea leads to **LSM Trees**.

## Step 2: The key rule (lock this in)

> Never update data in place. Always write new data.
>

Old data stays. New data is written on top.

## Step 3: Start with memory (MemTable)

When a write comes in:

```text
PUT(cat, 10)
PUT(dog, 20)
PUT(apple, 5)
```

Instead of disk, we write to memory.

### MemTable (sorted in RAM)

```text
apple → 5
cat   → 10
dog   → 20
```

Fast. No disk yet.

##  Step 4: MemTable fills up → flush to disk

Memory is limited.

When MemTable is full:

* freeze it
* write it to disk
* start a new MemTable

The disk file is called an SSTable.


## Step 5: First SSTable on disk

```text
SSTable_1 (on disk)
------------------
apple → 5
cat   → 10
dog   → 20
```

Important:

* Sorted
* Immutable (never changes)


## Step 6: More writes come in

Now we write more data:

```text
PUT(cat, 15)   // update
PUT(egg, 3)
```

New MemTable:

```text
cat → 15
egg → 3
```

## Step 7: Flush again → another SSTable

```text
SSTable_2 (newer)
-----------------
cat → 15
egg → 3
```

Disk now has:

```text
SSTable_2 (newest)
SSTable_1 (older)
```

## Step 8: How READ works (very important)

Suppose we do:

```text
GET(cat)
```

Database checks:

* MemTable (if exists)
* SSTable_2
* SSTable_1

Finds:

```text
cat → 15
```

Stops immediately. 

Newer data always wins.


## Step 9: Bloom Filters help speed reads

Before reading an SSTable:

* Bloom Filter says:
    * ❌ "Not here" → skip
    * 🤔 "Maybe here" → check

So we don't scan every file.


## Step 10: Deletes (tombstones)

If you do:

```text
DELETE(dog)
```

The MemTable records a tombstone entry:

```text
dog → TOMBSTONE
```

Later SSTable:

```text
SSTable_3
---------
dog → <tombstone>
```

This hides old values.

## Step 11: Problem: Too many SSTables

Over time:

* Many SSTables
* Reads get slower
* Disk usage grows

## Step 12: Compaction (cleanup)

Background process:

1. Read multiple SSTables
2. Merge them (like merge sort)
3. Keep only latest value per key
4. Remove tombstones
5. Write a new SSTable

Example:

Before compaction:

```text
SSTable_2: cat → 15
SSTable_1: cat → 10
```

After:

```text
SSTable_compacted: cat → 15
```

Old files are deleted.


## Step 13: Why LSM Trees are fast

Writes:

* In-memory
* Sequential disk writes
* No random IO

Reads:

* More complex
* Fixed by Bloom filters + compaction

LSM trades:

> Write speed for read complexity

## Step 14: One-sentence ELI5 summary (memorize this)

> An LSM Tree stores new data in memory, writes it to disk as immutable sorted files, and later merges 
> those files to keep reads fast.
> 




# Q-7 MemTable and SSTable: Structure and Layout

## What a MemTable looks like (in memory)

A MemTable is just an in-memory sorted map.

Think of it as:

```text
SortedMap<Key, Value>
```

Example: MemTable contents

```text
MemTable
--------
apple   → 10
banana  → 20
cat     → 30
dog     → 40
```

Important properties:

* Sorted by key
* Lives entirely in RAM
* Mutable (can change)
* Fast inserts and updates

### How it’s actually implemented

In real systems (RocksDB, Cassandra):

* Skip List (most common)
* Red-Black Tree
* AVL Tree

You don't need to know how its implemented — the key point is:
> Keys are always kept sorted.

### What happens on write

```text
PUT(cat, 99)
```

MemTable becomes:

```text
apple   → 10
banana  → 20
cat     → 99   (newer value)
dog     → 40
```

No disk touched yet.

## 2. What happens when MemTable is flushed

When MemTable gets full:

* It is frozen (read-only)
* Written to disk
* Turned into an `SSTable`


## 3. What an SSTable looks like (on disk)

An SSTable is **immutable** and **sorted**.

But it’s not just a flat file. It has **structure**.

### SSTable (high-level layout)

```text
+--------------------+
| Data Blocks        |
+--------------------+
| Index Block        |
+--------------------+
| Bloom Filter       |
+--------------------+
| Footer             |
+--------------------+
```

Let's go piece by piece.

### 3.1 Data Blocks (the actual data)

Data is stored in **blocks**, not one giant list.

Example:

```text
Data Block 1
------------
apple  → 10
banana → 20

Data Block 2
------------
cat → 99
dog → 40
```

Properties:

* Sorted within the block
* Fixed-size (e.g., 4 KB)
* Read sequentially from disk


### 3.2 Index Block (how SSTable is searched)

The index tells you which block to read.

Example:

```text
Index Block
-----------
apple → Block 1
cat   → Block 2
```

Meaning:

* If key < cat → look in Block 1
* Else → Block 2

This avoids scanning the whole file.


### 3.3 Bloom Filter (quick "not here" check)

Each SSTable has a Bloom filter.

Example logic:

```text
GET("elephant")
Bloom Filter → definitely NOT present
→ skip this SSTable entirely
```

This avoids disk reads.

### 3.4 Footer

Stores:

* Pointers to index
* Metadata
* Version info

Used when opening the file.


## 4. SSTable example (full picture)

```text
SSTable-42
----------
Data Blocks:
  [apple → 10, banana → 20]
  [cat → 99, dog → 40]

Index:
  apple → block-0
  cat   → block-1

Bloom Filter:
  bits = 101011001...

Footer:
  index_offset = 8192
```

Once written:

* ❌ Cannot be modified
* ❌ Cannot be appended
* ✅ Can be read efficiently


## 5. How reads actually happen (step-by-step)

Let's do:

```text
GET(cat)
```

**Step 1: Check MemTable**

* If found → return immediately

**Step 2: Check newest SSTable**

* Bloom filter → maybe present
* Index → find correct block
* Read block → binary search inside block

**Step 3: Stop at first match**

* Newest value wins


## 6. Deletes (tombstones) in MemTable and SSTable

Delete is just another entry.

**MemTable after delete**

```text
cat → TOMBSTONE
```

**SSTable on disk**

```text
SSTable (on disk)
-----------------
apple → 10
banana → 20
cat → TOMBSTONE
dog → 40
```



During compaction:

* Older values are removed
* Tombstone may disappear


## 7. Key difference (very important)

| MemTable      | SSTable               |
|---------------|-----------------------|
| In memory     | On disk               |
| Mutable       | Immutable             |
| Fast writes   | Fast sequential reads |
| One at a time | Many files            |
| Temporary     | Long-lived            |





# Q-8 What is Consistent hashing?

Imagine you have 10,000 users and 3 servers.

You need to decide:

> "Which user goes to which server?"
>

Naive idea (bad)

Use:

```text
userId % numberOfServers
```

If servers = 3 → works.

But now one server crashes and servers = 2.

💥 Disaster

Almost every user moves to a different server.

That means:

* Cache misses
* Data reshuffling
* Massive load spike

What problem Consistent Hashing solves
> When servers are added or removed, only a small portion of keys should move.
>

That's it. That’s the whole reason it exists.

## Consistent hashing: the core idea

Instead of mapping **keys to shard numbers**, we map:

* shards → positions on a number line
* keys → positions on the same number line

Then we use a simple rule:
> A key goes to the next shard on the right.

That's it.

## The hash space (horizontal line)

Assume a hash space from 0 to 99 (small for clarity):

```text
0 ------------------------------------------------------------ 99
```

This line wraps around:
* after 99 comes 0 again

## Place shards on the line

Suppose we have 3 shards.

We hash each shard’s ID and place it on the line:

```text
0 ------------------------------------------------------------ 99
|            |                      |
S1(10)       S2(40)                 S3(70)
```

Each shard "owns" keys to its left, up to the previous shard.


## Place keys on the same line

Now hash some keys:

```text
apple  → 12
cat    → 35
dog    → 55
zebra  → 90
```

```text
0 ------------------------------------------------------------ 99
|            |                      |
S1(10)       S2(40)                 S3(70)

 apple(12)          dog(55)                  zebra(90)
      cat(35)
```

## Assign keys to shards (very important)

Rule again:

> Key goes to the next shard on the right

| Key   | Hash | Assigned shard        |
|-------|------|-----------------------|
| apple | 12   | S2 (40)               |
| cat   | 35   | S2 (40)               |
| dog   | 55   | S3 (70)               |
| zebra | 90   | S1 (10) ← wrap around |

So shard ownership is:

```text
S1 owns (70 → 10]
S2 owns (10 → 40]
S3 owns (40 → 70]
```


## Adding a shard (this is where consistent hashing shines)

Now add S4, hashed to position 50.

```text
0 ------------------------------------------------------------ 99
|            |          |          |
S1(10)       S2(40)     S4(50)     S3(70)
```

What changes?

Only keys in one small range move:

```text
(40 → 50]
```

These keys move:

* from S3
* to S4

All other keys stay where they are.

* ✅ This is minimal movement
* ❌ No massive reshuffle

## So far so good… but here's the problem

What if shard positions are uneven?

Example:

```text
0 ------------------------------------------------------------ 99
|    |                                   |
S1(5) S2(15)                             S3(90)
```

Now:

* S3 owns a huge range
* S1 and S2 own tiny ranges

Result:

* one shard overloaded
* others idle

This happens because:
* Shard positions are random


## Enter virtual nodes (this is the key)

Instead of placing **one point per shard**, we place **many points per shard**.

These points are called **virtual nodes (vnodes)**.


## Setup: hash space and virtual nodes

Assume hash space 0–99.

We have 3 physical nodes, each with 3 virtual nodes.

Initial placement

```text
0 -------------------------------------------------------------------------------- 99

10     18     25     38     45     55     68     75     88
|      |      |      |      |      |      |      |      |
A1     B1     C1     A2     B2     C2     A3     B3     C3
```

Legend:

* `A1, A2, A3` → physical node A
* `B1, B2, B3` → physical node B
* `C1, C2, C3` → physical node C

## Ownership rule (recap)

> A key goes to the **next virtual node on the right**
(wrap around at 99 → 0)

Each virtual node owns the range from the previous vnode (exclusive) to itself (inclusive).


## Initial ownership ranges

Let's list them clearly:

```text
| Range     | Virtual Node | Physical Node |
| --------- | ------------ | ------------- |
| (88 → 10] | A1           | A             |
| (10 → 18] | B1           | B             |
| (18 → 25] | C1           | C             |
| (25 → 38] | A2           | A             |
| (38 → 45] | B2           | B             |
| (45 → 55] | C2           | C             |
| (55 → 68] | A3           | A             |
| (68 → 75] | B3           | B             |
| (75 → 88] | C3           | C             |
```

Each physical node owns **three small ranges**, spread across the space.

Balanced load.

## Add a new physical node D

Node D joins with 3 virtual nodes:

```text
D1 = 15
D2 = 50
D3 = 82
```

**New layout**

```text
0 -------------------------------------------------------------------------------- 99

10  15  18  25  38  45  50  55  68  75  82  88
|   |   |   |   |   |   |   |   |   |   |   |
A1  D1  B1  C1  A2  B2  D2  C2  A3  B3  D3  C3
```

## What moves when node D is added?

**Rule:**
Only ranges **immediately before D's virtual nodes** move.

**Affected ranges**

| New vnode | Range taken | Taken from |
|-----------|-------------|------------|
| D1(15)    | (10 → 15]   | A          |
| D2(50)    | (45 → 50]   | B          |
| D3(82)    | (75 → 82]   | B          |


Everything else stays exactly the same.

## Ownership after adding D

Now node D owns:

* `(10 → 15]`
* `(45 → 50]`
* `(75 → 82]`

Each existing node:

* loses small slices
* not a big chunk

This is the **core benefit of virtual nodes**.


## Remove physical node B

Node B goes down → all its virtual nodes disappear:

```text
B1(18), B2(45), B3(75)
```

New layout after removal

```text
0 -------------------------------------------------------------------------------- 99

10  15  25  38  50  55  68  82  88
|   |   |   |   |   |   |   |   |
A1  D1  C1  A2  D2  C2  A3  D3  C3
```


## What happens to B’s ranges?

Each range owned by B is taken over by the next vnode.

| Old B range | New owner |
|-------------|-----------|
| (10 → 18]   | C1        |
| (38 → 45]   | D2        |
| (68 → 75]   | D3        |

Again:

* no global reshuffle
* only local reassignment

## Why virtual nodes make this smooth

Without virtual nodes:

* Node B might own 40% of the data
* Removing B would overload one node

With virtual nodes:

* Node B owned many small slices
* Those slices are spread across:
    * A
    * C
    * D

Load redistribution is **even**.

## Key insight (this is the “aha”)

> Virtual nodes turn big, dangerous rebalances into many tiny, safe ones.

They are not an optimization.

They are what makes consistent hashing usable in production.





# Q-9 What is Gossip Protocol (SWIM-style)?

---

## 1. What problem Gossip solves

In a distributed system, every node must know:

* which other nodes are **alive**
* which nodes are **dead**

This must work:

* without a central coordinator
* under node crashes and network issues
* at large scale

Gossip achieves this using **peer-to-peer, probabilistic information exchange** with **eventual consistency**.

---

## 2. Core data structures (very important)

Each node maintains **one membership map** that tracks *other* nodes.

```text
membership = {
  node_id → {
    state: ALIVE | SUSPECT | DEAD,
    incarnation: integer,
    suspect_time: timestamp (only when state = SUSPECT)
  }
}
```

Key clarifications:

* There is **one map**, not separate lists
* `SUSPECT` is just a **state**, not a different structure
* A node does **not store itself** in the map
* “self” is implicit

---

## 3. Incarnation number — what it is and why it exists

### What is an incarnation?

An **incarnation number** is a **version counter owned by a node about itself**.

It answers:

> “Is this information newer or older than what I already know?”

### Critical rules

* Only the **node itself** can increment its incarnation
* Other nodes only **copy** the value they hear
* Higher incarnation **always wins**, regardless of state

Example:

```text
ALIVE (inc 6) overrides DEAD (inc 5)
```

This is how nodes safely recover from false death.

---

## 4. How a node gets its initial incarnation

When a node starts **for the very first time**:

```text
self = D
self_incarnation = 1
```

Important:

* This value is **locally initialized**
* It is **not learned from gossip**
* It is persisted to local storage

On restart:

* The node **loads the persisted value**
* Then increments it before gossiping again

---

## 5. Bootstrap phase (addresses only)

Every node is deployed with a static list:

```text
BOOTSTRAP_LIST = [A, B, C, D, E]
```

Important:

* This is **only a list of addresses**
* It contains **no liveness information**
* No node knows who is alive yet

---

## 6. Node A starts

Initial state of A:

```text
self = A
self_incarnation = 1

membership = {}   // empty
```

At this point:

* A knows no one is alive
* It only knows addresses from bootstrap

---

## 7. How gossip rounds work (high level)

Each gossip round:

1. Pick `k` random peers
2. Send current membership view
3. Receive reply
4. Merge received information

---

## 8. Message handling order (critical)

When a node receives gossip:

```text
RECEIVE → MERGE → UPDATE → REPLY
```

A node **never replies before merging**.
Replies always contain the **freshest known state**.

---

## 9. Membership discovery example

### A contacts C and E

```text
A → C : membership={}
A → E : membership={}
```

Explanation:

* A is announcing its existence
* It is not claiming anyone else is alive

---

### C and E process A’s message

C updates its membership:

```text
membership = {
  A → { state: ALIVE, incarnation: 1 }
}
```

Explanation:

* Receiving a message from A proves A is alive
* C records A with A’s incarnation

E does the same.

---

### C and E reply

```text
C → A : membership={A(1)}
E → A : membership={A(1)}
```

Replies reflect **post-merge state**, not stale state.

---

### A merges replies

```text
membership = {
  C → { state: ALIVE, incarnation: 1 },
  E → { state: ALIVE, incarnation: 1 }
}
```

Explanation:

* A infers C and E are alive because they replied
* A does not store itself

---

## 10. Discovery continues (A meets B)

```text
A → B : membership={C(1), E(1)}
```

B processes first, then replies.

B’s membership becomes:

```text
membership = {
  A → { state: ALIVE, incarnation: 1 },
  C → { state: ALIVE, incarnation: 1 },
  E → { state: ALIVE, incarnation: 1 }
}
```

B replies with this view.

A merges and adds B.

---

## 11. Stable cluster view

Eventually, after multiple rounds:

```text
membership = {
  B → { state: ALIVE, incarnation: 1 },
  C → { state: ALIVE, incarnation: 1 },
  D → { state: ALIVE, incarnation: 1 },
  E → { state: ALIVE, incarnation: 1 }
}
```

All nodes converge to the same view.

---

# FAILURE DETECTION (MOST IMPORTANT PART)

---

## 12. Node D crashes silently

* D stops responding
* No death message is sent

This is the hardest failure type.

---

## 13. A probes D

```text
A → D : ping
```

No response.

Explanation:

* A single missed probe is **not enough** to declare death
* Network issues and pauses are common

---

## 14. A marks D as SUSPECT

```text
membership[D] = {
  state: SUSPECT,
  incarnation: 1,
  suspect_time: t0
}
```

Explanation:

* This is a **local suspicion**
* Other nodes may still think D is alive

---

## 15. A gossips suspicion and performs indirect probes

A does two things in parallel.

### Gossip suspicion

```text
A → B : D is SUSPECT (1)
A → C : D is SUSPECT (1)
```

### Indirect probes

```text
A → B : "Ping D"
A → C : "Ping D"
```

Explanation:

* Indirect probes reduce false positives
* Other nodes try reaching D independently

---

## 16. Two possible outcomes

### Case 1: D responds to someone

If B receives:

```text
D → B : pong
```

Then B reports back, and A updates:

```text
membership[D] = {
  state: ALIVE,
  incarnation: 1
}
```

Explanation:

* Suspicion was false
* System recovers automatically

---

### Case 2: D responds to no one

* All indirect probes fail
* No ALIVE update arrives

---

## 17. Suspect timeout (`T_suspect`)

Each SUSPECT entry must remain suspected for a minimum time:

```text
T_suspect = e.g. 5 seconds
```

Condition:

```text
now() - suspect_time ≥ T_suspect
```

Explanation:

* Prevents killing slow but healthy nodes

---

## 18. A marks D as DEAD

```text
membership[D] = {
  state: DEAD,
  incarnation: 1
}
```

Explanation:

* Death is declared **only after time + corroboration**
* Never after a single failure

---

## 19. Death information is gossiped

```text
A → B : D is DEAD (1)
A → C : D is DEAD (1)
A → E : D is DEAD (1)
```

Other nodes merge this state.

---

## 20. Cluster converges again

Eventually all nodes have:

```text
membership[D] = {
  state: DEAD,
  incarnation: 1
}
```

D is removed from active membership.

---

# NODE REVIVAL (WHY INCARNATION MATTERS)

---

## 21. D restarts

On restart:

```text
self = D
self_incarnation = 1   // loaded from disk
self_incarnation = 2   // incremented
```

Explanation:

* Increment ensures newer truth
* Without this, D could be ignored forever

---

## 22. D gossips ALIVE with higher incarnation

```text
D → A : D is ALIVE (2)
```

---

## 23. A compares incarnation

Comparison:

```text
ALIVE (2) > DEAD (1)
```

So A updates:

```text
membership[D] = {
  state: ALIVE,
  incarnation: 2
}
```

This update spreads cluster-wide.

---

## 24. Final state machine (memorize)

```text
ALIVE
  ↓ (missed probes)
SUSPECT
  ↓ (timeout + no refutation)
DEAD
  ↑ (ALIVE with higher incarnation)
```

---

## 25. Why this design is correct

| Problem          | Mechanism                 |
| ---------------- | ------------------------- |
| False positives  | SUSPECT + indirect probes |
| Network blips    | Timeout before death      |
| Conflicting info | Incarnation numbers       |
| SPOF             | Fully decentralized       |

---

## 26. Perfect interview summary (one sentence)

> Gossip maintains a single membership map where nodes transition from ALIVE to SUSPECT to DEAD based on timed suspicion and corroborated failures, and incarnation numbers—owned and incremented by the node itself—ensure newer liveness information always overrides stale gossip.

---

## 27. Ultra-short version (interrupt-safe)

> Nodes gossip membership, mark failures as SUSPECT, declare DEAD only after a timeout, and use incarnation numbers to resolve conflicts and allow safe recovery.

---



# Q-7 What are Vector Clocks?

# Q-8 What are different cache eviction policies?

# Q-9 What are different Rate Limiting Algorithms?

1. db isolation level
1. normal forms
1. types of indexes
1. cap, pcap
1. locality of reference
1. etag
1. types of cache
1. materialized view
1. pub-sub vs message queues
1. how to processes can communicate, mq, sock
1. lock free data structures
1. parallel algorithms
1. long and short polling
1. database locks
1. what is defragmentation in os
1. ways to scale db
1. grub
1. https://www.linuxfromscratch.org/lfs/downloads/stable/LFS-BOOK-12.3.pdf


1. use soft delete to delete rows or push the delete message to mq and delete later

5. notification-system.md
6. news-feed.md
7. distributed-cache.md
8. file-storage.md
9. search-autocomplete.md
10. logging-metrics.md
11. payment-system.md
