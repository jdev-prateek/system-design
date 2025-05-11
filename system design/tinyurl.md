This is read heavy system with i.e. R:W = 5:1

Assume we generate 2B urls every year:

URLs created in a day:

```text
1 month  =  2*10**9 / 365
1 day = (2*10**9 / 365) / 30 = 5.5M ~ 6M
```

URLs created in a sec:

```text
=> 24 * 60 * 60 seconds = 6M
=> 1 sec = 6M/(24 * 60 * 60)
=> 1 sec = 70 rps ~ 100 rps
```

URLs created in 10 year:

```python
>>> 
>>> format(100 * 60 * 60 * 24 * 365 * 10, "_")
'31_536_000_000'
>>> 
```

which is almost 32B.

Using base 64 chars, we can use 6 digits to store 68B urls:

```python
>>> 
>>> format(64**6, "_")
'68_719_476_736'
>>>
```

```text
=> 1 unique id = 6 byte
=> 32B unique id = 6 * 32B
=> 192GB
```

192GB of data is not much. Hence, we will use use a single db to store all unique ids along with status (used or unused).

## keys-db schema

```text
id long
unique_id varchar
status (used, unused)
```

## Key generation service

To insert the unique ids in the `keys-db` we will use key-generation-service. It will be a simple application that will generate base 64 keys and saves it to the db.

Note that data of 192GB is manageable in a single db. If you face any bottlenecks or want to distribute unique id for better latency. You can distribute the data across multiple databases by sharding based on `hash(unique_id) % shard_count`.

## key-sync service

Now we have unique keys in db. Next, we need to load them up into the redis server. We do this to reduce latency.

The key-sync service will fetch rows from keys-db in batch and distribute it among 4 redis shards (i.e. `hash(unique_id) % 4`).

You have a system with multiple key-sync services and multiple Redis shards, where each key-sync can manage up to a limited number of Redis instances (e.g., 2). To avoid overlapping work, each key-sync service attempts to acquire a distributed lock (using Redis SET NX EX) on available Redis shards. If it successfully acquires a lock (e.g., with a 5-minute TTL), it monitors the corresponding Redis instance for the number of unique IDs. If the count drops below a defined threshold, it pulls more IDs from the keys-db and pushes them into Redis. The key-sync should hold the lock for the full duration or renew it periodically to maintain shard ownership; releasing it too early could cause coordination issues. Importantly, the ID count should only be checked after acquiring the lock, to avoid race conditions where multiple services attempt to refill the same shard. This orchestration ensures exclusive, safe, and scalable ID refilling across multiple Redis shards.

## mapping-service

When request comes to generate new short url it comes to mapping-service, the application will load balance among redis servers.

1. Check which shard has more IDs (use LLEN).
1. Pick shard dynamically.
1. Priority to healthier shards.

Otherwise the depletion of one shard can starve your system.

The following is the schema of db used to store url mapping and user.

users table
-------------------------
user_id
first_name
middle_name
last_name
email
password
created_at
updated_at

urls table
-------------------------
user_id
original_url
unique_id
created_at


Assume each row in `users` table take 1 KB and we have 10M users.

```python
>>> 
>>> format(10*10**6 * 10**3, "_")
'10_000_000_000'
>>> 
>>> 
```

that means to store 10M users we need 10GB. This much data can be easily put in a single db.

Assume each row in `urls` table take 2 KB and in 10 years we create 32B urls.

```python
>>> 
>>> format(2 * 10**3 * 32*10**9, "_")
'64_000_000_000_000'
>>> 
```

We would need 64TB of space to save urls mappings.

Lets say a commodity machine has 2TB of storage. Then, we will need 32 (64/2) servers.Thus we can shard the data on `hash(unique_id) % 32`.

Note: As per our calculation, users data can be stored in a single db. If this is not the case, then  mapping-service instances can generate snowflake like id that will be used as primary key for users.

## Caching

Assume 20% of total url mappings (64TB) gives us the hit rate of 98-99%.

A commodity caching server has 128GB of space.

20% of 64TB is:

```python
>>> 
>>> 64* 0.2
12.8
>>> 
```

Means we need 94 cache servers.

```python
>>> 
>>> (12 * 10**3) / 128
93.75
>>> 
```



## v3

You have multiple workers (key-producer services) and multiple boxes of keys (your keys-db shards).

You want each worker to pick a different box — no two workers should grab the same one!

Zookeeper is like a referee that watches over the workers and says:

"This box is already taken. Try another one."

### Step-by-Step:

1. Each worker goes to Zookeeper and says:

    > “Can I have keys-db-1 please?”

2. Zookeeper tries to create a lock like:

    ```bash
    /shard-locks/keys-db-1
    ```    
3. If no one has that lock, Zookeeper says:

    >  “Yes! You got it!”

    And that worker gets exclusive access to that DB shard.

4. If someone else already has it, Zookeeper says:

    > “Nope, someone else has it. Try another one.”

5. Now the worker tries the next one: `keys-db-2`, `keys-db-3`, etc., until it finds an available one.

6. Bonus: Zookeeper creates a special type of lock (called an ephemeral node).
    * If the worker dies or crashes, Zookeeper automatically deletes the lock, freeing the shard.
    * Another worker can then grab it!



## Use a single db to store all unique ids along with status (used or unused).

### keys table

* short_id
* status

Keys consists of following characters

1. `A-Z`
1. `a-z`
1. `0-9`
1. `-`,`_`

In total, we have 64 characters. That means, using 8 characters we can generate `64**8` (`281_474_976_710_656` which is 281 trillion) unique short keys. Using utf-8, 1 character would take 1 byte, so `64**8` urls would take:

=> 281_474_976_710_656
=> 281 * 10^12
=> 200 TB


### Key generation service

The key generation service will fetch records from keys db in batch and distribute it amount 4 redis shards (i.e. unique_id  % 4).

When request comes to generate new short url, the BE application will load balance amount redis servers.

1. Check which shard has more IDs (use LLEN).
1. Pick shard dynamically.
1. Priority to healthier shards.

Otherwise the depletion of one shard can starve your system.

