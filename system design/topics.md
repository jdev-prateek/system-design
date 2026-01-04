 <!-- TOC -->
* [Q-1 What is Partition Tolerance?](#q-1-what-is-partition-tolerance)
* [Q-2 What is CAP theorem?](#q-2-what-is-cap-theorem)
  * [Why Partition Tolerance is mandatory](#why-partition-tolerance-is-mandatory)
  * [Let’s see the choice (ELI5)](#lets-see-the-choice-eli5)
    * [Case 1: Choose Consistency](#case-1-choose-consistency)
    * [Case 2: Choose Availability](#case-2-choose-availability)
  * [Important correction (many people get this wrong)](#important-correction-many-people-get-this-wrong)
  * [Real-world mapping (intuition)](#real-world-mapping-intuition)
<!-- TOC -->

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

Neither is "better".  They solve different problems.


