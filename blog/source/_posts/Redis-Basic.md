---
title: Redis Basic
date: 2022-12-06 16:28:35
tags:
---
# Redis
* One thread, I/O multiplexing
# String
*  GETSET command sets a key to a new value, returning the old value as the result
* Set value if key not exist
* Set value if key exists
```
> set mykey newval nx
(nil)
> set mykey newval xx
OK
```
* Set value with expiration
```
> set key 100 ex 10
OK
> ttl key
(integer) 9
```
* mset
```
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```
* atomic: That even multiple clients issuing INCR against the same key will never enter into a race condition
 - incr
 - incrby
 - decr
 - decrby
```
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```
* exists
* del
* type
```
> exists mykey
(integer) 1
> del mykey
(integer) 1
```
```
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```
* expire in 5 seconds
```
> expire key 5
(integer) 1
```
Note: In order to set and check expires in milliseconds, check the PEXPIRE and the PTTL commands, and the full list of SET options.
# List
* Redis lists are implemented via Linked Lists
```
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```
* lrange is for display
```
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```
* ltrim will change the list
```
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```
* blpop and brpop will block if the list is empty
```
> brpop tasks 5
1) "tasks"
2) "do_something"
```
"wait for elements in the list tasks, but return NULL if after 5 seconds no element is available". 0 means waiting forever
- the first client that blocked waiting for a list, is served first when an element is pushed by some other client, and so forth.

* BLMOVE
* Automatic creation and removal of keys
1. When we add an element to an aggregate data type, if the target key does not exist, an empty aggregate data type is created before adding the element.
2. When we remove elements from an aggregate data type, if the value remains empty, the key is automatically destroyed. The Stream data type is the only exception to this rule.
3. Calling a read-only command such as LLEN (which returns the length of the list), or a write command removing elements, with an empty key, always produces the same result as if the key is holding an empty aggregate type of the type the command expects to find.

# Hash
```
> hset user:1000 username antirez birthyear 1977 verified 1
(integer) 3
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```
```
> hmget user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```
```
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```
# Set
```
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 3
2. 1
3. 2
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0

```
* intersection between different sets
```
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
```
* SPOP command removes a random element, returning it to the client
* SRANDMEMBER will return a random element without removing it
* Number of elements
```
> scard game:1:deck
(integer) 47
```
# Sorted sets
* zadd key score value
```
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer) 1
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```
- If B and A are two elements with a different score, then A > B if A.score is > B.score.
- If B and A have exactly the same score, then A > B if the A string is lexicographically greater than the B string. B and A strings can't be equal since sorted sets only have unique elements.
```
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```
```
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```
```
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```
```
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```
```
> zremrangebyscore hackers 1940 1960
(integer) 4
```
```
> zrank hackers "Anita Borg"
(integer) 4
```
ZREVRANK
* Lexicographical scores
```
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
  "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
  0 "Linus Torvalds" 0 "Alan Turing"
```
```
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```
```
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds"
```
Just calling ZADD against an element already included in the sorted set will update its score (and position) with O(log(N)) time complexity.
# Bitmap
a set of bit-oriented operations defined on the String type
```
> setbit key 10 1
(integer) 1
> getbit key 10
(integer) 1
> getbit key 11
(integer) 0
```
The SETBIT command takes as its first argument the bit number, and as its second argument the value to set the bit to, which is 1 or 0.
GETBIT just returns the value of the bit at the specified index. Out of range bits (addressing a bit that is outside the length of the string stored into the target key) are always considered to be zero.

There are three commands operating on group of bits:

BITOP performs bit-wise operations between different strings. The provided operations are AND, OR, XOR and NOT.
BITCOUNT performs population counting, reporting the number of bits set to 1.
BITPOS finds the first bit having the specified value of 0 or 1.

# HyperLogLogs
* to estimate unique things with standard error of < 1%.
```
> pfadd hll a b c d
 (integer) 1
 > pfcount hll
 (integer) 4
```

# Stream

# Persistence
* RDB (Redis Database): point-in-time snapshots of your dataset at specified intervals.
  - pro
    - single file, good for backup and disaster recovery
    - maximize performance because parent process forks a child to persist
    - faster restart compared to AOF
    - On replicas, RDB supports partial resynchronizations after restarts and failovers.
  - con
    - may have data loss
    - fork() can be time consuming if the dataset is big, and may result in Redis stopping serving clients for some milliseconds or even for one second if the dataset is very big and the CPU performance is not great. AOF also needs to fork() but less frequently
* AOF (Append Only File): AOF persistence logs every write operation received by the server. These operations can then be replayed again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself.
  - con
    - AOF files are usually bigger than the equivalent RDB files for the same dataset.
    - AOF can be slower than RDB depending on the exact fsync policy.
* No persistence
* RDB + AOF

* In the case both AOF and RDB persistence are enabled (recommended) and Redis restarts the AOF file will be used to reconstruct the original dataset since it is guaranteed to be the most complete

# Issues
* Cache avalanche
  - cache is not available and all requests reach DB
  - solution
    - circuit breaker
    - master-slave or cluster to ensure avaibility
* cache penetration
  - frequenly query a data (e.g. ID = -1) that does not exist in both DB and cache, crush the DB
  - solution
    - check with codes (e.g. id cannot be less than 0)
    - save to Redis with null as value, TTL 30
    - Bloom Filter // TODO
* expired hotspot
  - solution
    - set hotspot to “never expire”
    - do not make a lot of keys expire at the same time
    - lock to ensure requests regarding the same data will not reach DB at the same time
# Cache elimination strategy
noeviction
allkeys-lru
volatile-lru
allkeys-random
volatile-random
volatile-ttl

# Implement queue
* delayed queue: sorted set, timestamp as score, zrangebyscore to get data added n seconds ago
