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

