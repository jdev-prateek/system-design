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
* [Q-6 What are Vector Clocks?](#q-6-what-are-vector-clocks)
* [Q-6 What are Vector Clocks?](#q-6-what-are-vector-clocks-1)

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

# Q-6 What are Vector Clocks?

# Q-6 What are Vector Clocks?

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