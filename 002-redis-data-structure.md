# 🗂️ Redis Data Structures

A practical reference guide for the Redis data structures — lists, sets, hashes, sorted sets, geospatial indexes, and streams — covering the commands that operate on each one, what they return, and the use cases each structure is actually built for, worked through hands-on in `redis-cli`.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [The Structures at a Glance](#the-structures-at-a-glance)
  - [Keys and Types](#keys-and-types)
- [Lists](#-lists)
  - [Core Commands](#core-commands)
  - [Queue (FIFO)](#queue-fifo)
  - [Stack (LIFO)](#stack-lifo)
  - [Reading a Range](#reading-a-range)
- [Sets](#-sets)
  - [Set Commands](#set-commands)
  - [Adding & Counting Members](#adding--counting-members)
  - [Reading & Removing Members](#reading--removing-members)
  - [Set Operations](#set-operations)
- [Hash](#-hash)
  - [Hash Commands](#hash-commands)
  - [Setting Fields](#setting-fields)
  - [Reading Fields](#reading-fields)
  - [Incrementing a Field](#incrementing-a-field)
- [Sorted Set](#-sorted-set)
  - [Sorted Set Commands](#sorted-set-commands)
  - [Adding Members](#adding-members)
  - [Reading by Index](#reading-by-index)
  - [Reading by Score](#reading-by-score)
  - [Removing Members](#removing-members)
- [Geospatial](#-geospatial)
  - [Geospatial Commands](#geospatial-commands)
  - [Adding Locations](#adding-locations)
  - [Position & Distance](#position--distance)
  - [Radius Search](#radius-search)
- [Streams](#-streams)
  - [Stream Commands](#stream-commands)
  - [Appending Entries](#appending-entries)
  - [Reading Entries](#reading-entries)
  - [Blocking Reads](#blocking-reads)
  - [Consumer Groups](#consumer-groups)
  - [Reading with a Consumer Group](#reading-with-a-consumer-group)
- [Practical Use Cases](#-practical-use-cases)
  - [Real-time Chat System](#real-time-chat-system)
  - [E-commerce Product Inventory](#e-commerce-product-inventory)
  - [Gaming Leaderboard](#gaming-leaderboard)
  - [Location-based Service](#location-based-service)
  - [Task Queue System](#task-queue-system)
  - [Session Management](#session-management)
  - [Rate Limiting](#rate-limiting)
  - [Tag System](#tag-system)
- [Common Pitfalls](#-common-pitfalls)
- [Choosing a Data Structure](#-choosing-a-data-structure)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### The Structures at a Glance

A Redis key is always a string, but the **value** behind it has a type — and that type decides which commands are legal and how fast they run.

| Structure | What it stores | Built for |
| --- | --- | --- |
| **List** | An ordered linked list of strings, addressable by index | Queues, task pipelines, logs |
| **Set** | An unordered collection of **unique** strings | Tags, unique IDs, de-duplication |
| **Hash** | Field-value pairs inside one key, like a JSON object | User profiles, products, config objects |
| **Sorted Set** | Unique members, each carrying a numeric **score** | Leaderboards, rankings, time-based queues |
| **Geospatial** | Longitude/latitude coordinates attached to members | Nearby stores, users, delivery tracking |
| **Stream** | An append-only log of entries, each with a unique ID | Event sourcing, message brokers, activity feeds |

### Keys and Types

- Every command is **type-specific** — `lpush` on a key holding a hash fails with a `WRONGTYPE` error rather than converting it
- The command prefix tells you the type: `l` for list, `s` for set, `h` for hash, `z` for sorted set, `geo` for geospatial, `x` for stream
- Naming follows the usual convention — `user:1001:profile`, `chat:room1`, `article:1001:tags`

> **Key Insight:** picking the structure *is* the design decision. A leaderboard built on a list means sorting in application code on every read; the same leaderboard on a sorted set is one `zrange`. Redis has no query planner to rescue a bad choice — the structure you pick is the performance you get.

---

## 📜 Lists

A **List** is a linked list of string values. It behaves much like an array — every element has an index — and because you can push and pop from either end, one structure covers both a queue and a stack.

**Common use cases:** message queues, task pipelines, and logs.

### Core Commands

| Command | Description |
| --- | --- |
| `lpush key value [value ...]` | Push one or more values onto the **left** (head) |
| `rpop key [count]` | Remove and return a value from the **right** (tail) |
| `lpop key [count]` | Remove and return a value from the **left** (head) |
| `lrange key start stop` | Get the elements between two indexes |

> **Correction:** `lpop` takes a key and an optional **count**, not a value — the original note's `lpop stack value` would be read as popping `value` elements. To pop one element it is simply `lpop history`.

### Queue (FIFO)

Push on the left, pop from the right: the first element added is the first one removed.

```bash
lpush queue value
```

```bash
lpush tasks "send email"
# (integer) 1

lpush tasks "process payment"
# (integer) 2

lpush tasks "generate report"
# (integer) 3

# Queue now: ["generate report", "process payment", "send email"]
```

```bash
rpop queue
```

```bash
rpop tasks
# "send email"  # First in, first out

rpop tasks
# "process payment"

rpop tasks
# "generate report"

rpop tasks
# (nil)  # Queue is empty
```

> **Note:** `lpush` returns the new **length** of the list, and `rpop` returns `nil` on an empty list — which is how a worker knows there is no work left rather than erroring out.

### Stack (LIFO)

Push and pop from the same end: the last element added is the first one removed.

```bash
lpush stack value
```

```bash
lpush history "page1.html"
# (integer) 1

lpush history "page2.html"
# (integer) 2

lpush history "page3.html"
# (integer) 3

# Stack now: ["page3.html", "page2.html", "page1.html"]
```

```bash
lpop key [count]
```

```bash
lpop history
# "page3.html"  # Last in, first out

lpop history
# "page2.html"

lpop history
# "page1.html"
```

> **Key Insight:** queue and stack are the *same* structure — only the end you pop from changes. `lpush` + `rpop` gives FIFO, `lpush` + `lpop` gives LIFO. Nothing about the list itself is different.

### Reading a Range

```bash
lrange key start stop
```

```bash
lpush mylist "one" "two" "three" "four" "five"
# (integer) 5

lrange mylist 0 -1
# 1) "five"
# 2) "four"
# 3) "three"
# 4) "two"
# 5) "one"

lrange mylist 0 2
# 1) "five"
# 2) "four"
# 3) "three"

lrange mylist 1 3
# 1) "four"
# 2) "three"
# 3) "two"
```

> **Tip:** both indexes are inclusive and negatives count from the tail, so `lrange key 0 -1` is the whole list. Note the ordering above — a single `lpush` with five values pushes them one at a time, so `"five"` ends up at the head.

---

## 🔷 Sets

A **Set** is an unordered collection of unique strings. Adding a member that already exists is silently ignored, which makes de-duplication free.

**Common use cases:** tags, unique IDs, removing duplicates.

### Set Commands

| Command | Description |
| --- | --- |
| `sadd key member [member ...]` | Add one or more members |
| `scard key` | Get the number of members |
| `smembers key` | Get every member |
| `srem key member [member ...]` | Remove one or more members |
| `sdiff key1 key2` | Members of the first set that are in none of the others |
| `sinter key1 key2` | Members present in **all** the given sets |
| `sunion key1 key2` | Every member across all the given sets |

### Adding & Counting Members

```bash
sadd tags "javascript" "redis" "database"
# (integer) 3

sadd tags "javascript"
# (integer) 0  # Already exists, not added

sadd tags "python" "golang"
# (integer) 2
```

```bash
scard tags
# (integer) 5
```

> **Note:** `sadd` returns how many members were **actually new**, not how many you passed. That return value is a ready-made "was this the first time?" check — useful for unique visitors, first-time votes, and idempotent processing.

### Reading & Removing Members

```bash
smembers tags
# 1) "redis"
# 2) "javascript"
# 3) "database"
# 4) "python"
# 5) "golang"
```

```bash
srem tags "golang"
# (integer) 1

srem tags "rust" "cpp"
# (integer) 0  # Neither exists
```

> ⚠️ **Warning:** a set is **unordered** — the order above is whatever Redis happened to return, and it may differ between calls. Never rely on it. `smembers` also returns the entire set in one reply, so on a large set use `sscan` instead.

### Set Operations

The three set operations are computed **on the server**, so comparing two collections never means shipping both to the client.

```bash
sadd skills:alice "python" "javascript" "redis" "docker"
sadd skills:bob "javascript" "golang" "redis"
```

```bash
sdiff skills:alice skills:bob
# 1) "python"
# 2) "docker"
# Skills that Alice has but Bob doesn't
```

```bash
sinter skills:alice skills:bob
# 1) "javascript"
# 2) "redis"
# Common skills between Alice and Bob
```

```bash
sunion skills:alice skills:bob
# 1) "python"
# 2) "javascript"
# 3) "redis"
# 4) "docker"
# 5) "golang"
# All skills combined
```

> **Key Insight:** `sdiff` is order-sensitive and the other two are not — `sdiff a b` answers "what does A have that B lacks", so swapping the arguments gives a different answer. `sinter` and `sunion` return the same result whichever way you list the keys.

---

## 🧩 Hash

A **Hash** stores field-value pairs inside a single key — the closest thing Redis has to a JSON object. Instead of `user:1001:name` and `user:1001:email` as two keys, one `user:1001` hash holds both.

**Common use cases:** user profiles, products, configuration objects.

### Hash Commands

| Command | Description |
| --- | --- |
| `hset key field value [field value ...]` | Set one or more fields |
| `hget key field` | Get the value of one field |
| `hgetall key` | Get every field and value |
| `hincrby key field increment` | Increment (or decrement) a numeric field |

### Setting Fields

```bash
hset user:1001 name "John Doe" email "john@example.com" age 30
# (integer) 3

hset user:1001 city "Jakarta"
# (integer) 1

hset product:2001 name "Laptop" price 15000000 stock 25
# (integer) 3
```

> **Note:** `hset` returns the number of **new** fields created — updating an existing field returns `0` even though the write succeeded. It is a "was this new?" signal, not a success flag.

### Reading Fields

```bash
hget user:1001 name
# "John Doe"

hget user:1001 email
# "john@example.com"

hget user:1001 nonexistent
# (nil)
```

```bash
hgetall user:1001
# 1) "name"
# 2) "John Doe"
# 3) "email"
# 4) "john@example.com"
# 5) "age"
# 6) "30"
# 7) "city"
# 8) "Jakarta"
```

> **Tip:** `hgetall` returns a **flat** list alternating field, value, field, value — client libraries usually fold it back into a map for you. Fetch only what you need with `hget` or `hmget` when the object is large.

### Incrementing a Field

```bash
hincrby key field increment
```

```bash
hset product:2001 stock 25
# (integer) 1

hincrby product:2001 stock 10
# (integer) 35  # Added 10 units

hincrby product:2001 stock -5
# (integer) 30  # Sold 5 units

hset user:1001 login_count 0
hincrby user:1001 login_count 1
# (integer) 1  # Increment login counter
```

> **Key Insight:** there is no `hdecrby` — a negative increment *is* the decrement, which is why `hincrby product:2001 stock -5` sells five units. And like `incr`, the arithmetic happens on the server, so concurrent stock updates cannot lose each other.

---

## 🏅 Sorted Set

A **Sorted Set** is a set where every member carries a numeric **score**. Members stay ordered by that score, ascending; ties are broken lexicographically by member name.

**Common use cases:** leaderboards, rankings, time-based queues.

### Sorted Set Commands

| Command | Description |
| --- | --- |
| `zadd key score member [score member ...]` | Add members with their scores |
| `zcard key` | Get the number of members |
| `zrange key start stop [withscores]` | Get members by **index** range |
| `zrange key min max byscore` | Get members by **score** range |
| `zrem key member [member ...]` | Remove one or more members |
| `zremrangebyscore key min max` | Remove every member within a score range |

> **Correction:** `zremrangebyscore` takes the **key** first, not a member — the original note's `zremrangebyscore member min max` is `zremrangebyscore leaderboard 0 100` in practice, exactly as the example below shows.

### Adding Members

```bash
zadd leaderboard 100 "player1"
# (integer) 1

zadd leaderboard 250 "player2"
# (integer) 1

zadd leaderboard 180 "player3"
# (integer) 1

# Add multiple members at once
zadd leaderboard 320 "player4" 150 "player5"
# (integer) 2
```

```bash
zcard leaderboard
# (integer) 5
```

> **Note:** re-adding an existing member **updates its score** and returns `0`, because no new member was created. That is what makes `zadd` the natural "record this player's latest score" command — no read-then-write needed.

### Reading by Index

```bash
zrange key start stop
```

```bash
zrange leaderboard 0 -1 withscores
# 1) "player1"
# 2) "100"
# 3) "player5"
# 4) "150"
# 5) "player3"
# 6) "180"
# 7) "player2"
# 8) "250"
# 9) "player4"
# 10) "320"
```

```bash
# Get top 3 lowest scores
zrange leaderboard 0 2
# 1) "player1"
# 2) "player5"
# 3) "player3"

# Get top 3 highest scores (using negative indices)
zrange leaderboard -3 -1
# 1) "player3"
# 2) "player2"
# 3) "player4"
```

> **Tip:** the index is the **rank**, and the default order is ascending — so the highest scores live at the end. Add the `rev` option (`zrange leaderboard 0 9 rev withscores`) when you want a top-10 that reads highest first, as in the [Gaming Leaderboard](#gaming-leaderboard) example.

### Reading by Score

```bash
zrange key min max byscore
```

```bash
# Get players with scores between 150 and 250
zrange leaderboard 150 250 byscore
# 1) "player5"
# 2) "player3"
# 3) "player2"

# Get all players with score above 200
zrange leaderboard 200 +inf byscore
# 1) "player2"
# 2) "player4"
```

> **Note:** `+inf` and `-inf` are valid bounds, which is how you express an open-ended range. The unified `zrange … byscore` / `rev` syntax requires **Redis 6.2 or newer** — on older servers the same queries are `zrangebyscore` and `zrevrange`.

### Removing Members

```bash
zrem leaderboard "player1"
# (integer) 1

zrem leaderboard "player2" "player5"
# (integer) 2
```

```bash
zremrangebyscore key min max
```

```bash
# Remove all players with scores below 100
zremrangebyscore leaderboard 0 100
# (integer) 1

# Remove players with scores between 150 and 200
zremrangebyscore leaderboard 150 200
# (integer) 2
```

> **Key Insight:** score ranges make a sorted set a time-based queue as well as a leaderboard — store a timestamp as the score, and `zremrangebyscore key 0 <one hour ago>` becomes a one-command cleanup of everything older than an hour.

---

## 📍 Geospatial

A **Geospatial** index stores longitude/latitude coordinates against members, so the server can answer distance and radius questions directly.

**Common use cases:** finding nearby stores, users, delivery tracking.

### Geospatial Commands

| Command | Description |
| --- | --- |
| `geoadd key longitude latitude member` | Add a location |
| `geopos key member [member ...]` | Get the coordinates of one or more members |
| `geodist key member1 member2 [unit]` | Get the distance between two members |
| `geosearch key fromlonlat lon lat byradius radius unit` | Find members within a radius |

> **Key Insight:** a geospatial key *is* a sorted set — the score is a geohash of the coordinates. That is why `zrem` removes a location and `zcard` counts them: everything you know about sorted sets still applies here.

### Adding Locations

```bash
geoadd key longitude latitude member
```

```bash
# Add locations in Jakarta
geoadd stores 106.8456 -6.2088 "Store A"
# (integer) 1

geoadd stores 106.8270 -6.1751 "Store B"
# (integer) 1

# Add multiple locations at once
geoadd stores 106.7650 -6.2293 "Store C" 106.8990 -6.2615 "Store D"
# (integer) 2
```

> ⚠️ **Warning:** the argument order is **longitude first, then latitude** — the opposite of how coordinates are usually written and copied from a map. Swap them and Redis stores a valid but completely wrong location, with no error to warn you.

### Position & Distance

```bash
geopos stores "Store A"
# 1) 1) "106.84559822082519531"
#    2) "-6.20880014472875656"

geopos stores "Store B" "Store C"
# 1) 1) "106.82699769735336304"
#    2) "-6.17509993682519115"
# 2) 1) "106.76499992609024048"
#    2) "-6.22930010320344569"
```

```bash
geodist key member1 member2 distance-unit
```

```bash
# Distance in kilometers
geodist stores "Store A" "Store B" km
# "4.0875"

# Distance in meters
geodist stores "Store A" "Store B" m
# "4087.4639"

# Distance in miles
geodist stores "Store C" "Store D" mi
# "10.2847"
```

> **Note:** the coordinates that come back are not byte-identical to what you stored — geohash encoding is lossy. The error is well under a metre, which matters for equality checks and not for anything else.

### Radius Search

```bash
geosearch key fromlonlat longitude latitude byradius distance unit
```

```bash
# Find stores within 5km of a specific coordinate
geosearch stores fromlonlat 106.8456 -6.2088 byradius 5 km
# 1) "Store A"
# 2) "Store B"

# Find stores within 10km with distances
geosearch stores fromlonlat 106.8456 -6.2088 byradius 10 km withdist
# 1) 1) "Store A"
#    2) "0.0000"
# 2) 1) "Store B"
#    2) "4.0875"
# 3) 1) "Store D"
#    2) "6.2341"

# Find stores within 15km with coordinates
geosearch stores fromlonlat 106.8456 -6.2088 byradius 15 km withcoord withdist
# 1) 1) "Store A"
#    2) "0.0000"
#    3) 1) "106.84559822082519531"
#       2) "-6.20880014472875656"
```

> **Tip:** `withdist` and `withcoord` are what make the result directly usable in a UI — you get "Store B, 4.09 km away" without a second round trip. `geosearch` (Redis 6.2+) supersedes the deprecated `georadius`.

---

## 🌊 Streams

A **Stream** is an append-only log. Each entry is a set of field-value pairs stamped with a unique ID, and entries are never modified — only appended and read.

**Common use cases:** event sourcing, message brokers, data pipelines, and activity feeds.

### Stream Commands

| Command | Description |
| --- | --- |
| `xadd key * field value [field value ...]` | Append an entry with an auto-generated ID |
| `xread streams key id` | Read entries from a given ID onwards |
| `xread count n streams key id` | Read at most `n` entries |
| `xread block ms streams key $` | Wait for new entries (`0` = forever) |
| `xgroup create key group $ mkstream` | Create a consumer group |
| `xgroup createconsumer key group consumer` | Register a consumer inside a group |
| `xreadgroup group group consumer streams key >` | Read undelivered messages as part of a group |

Three symbols do most of the work:

| Symbol | Meaning |
| --- | --- |
| `*` | In `xadd` — generate the ID from the current timestamp |
| `$` | Only entries added **from now on** |
| `>` | For a consumer group — only messages never delivered to any consumer |

### Appending Entries

```bash
xadd stream-name * key value
```

```bash
xadd events * action "user_login" user_id "1001" timestamp "2025-11-02T10:30:00"
# "1730548200000-0"

xadd events * action "purchase" user_id "1002" amount "150000"
# "1730548250000-0"

xadd events * action "logout" user_id "1001"
# "1730548300000-0"
```

> **Note:** the returned ID is `<milliseconds>-<sequence>`. The sequence counter is what keeps entries distinct when several land in the same millisecond, and because IDs only ever increase, they double as a cursor.

> **Gotcha:** a stream is append-only and grows **forever**. Cap it at write time with `xadd events maxlen ~ 10000 * …`, or trim it later with `xtrim` — otherwise it quietly consumes memory until `maxmemory` starts evicting.

### Reading Entries

```bash
xread streams stream-name 0
```

Reads everything from the beginning (ID `0`):

```bash
xread streams events 0
# 1) 1) "events"
#    2) 1) 1) "1730548200000-0"
#          2) 1) "action"
#             2) "user_login"
#             3) "user_id"
#             4) "1001"
#             5) "timestamp"
#             6) "2025-11-02T10:30:00"
#       2) 1) "1730548250000-0"
#          2) 1) "action"
#             2) "purchase"
#             3) "user_id"
#             4) "1002"
#             5) "amount"
#             6) "150000"
```

Add `count` to fetch a limited batch instead of the whole log:

```bash
xread count value streams stream-name 0
```

```bash
# Read only 2 entries
xread count 2 streams events 0
# 1) 1) "events"
#    2) 1) 1) "1730548200000-0"
#          2) 1) "action"
#             2) "user_login"
#             3) "user_id"
#             4) "1001"
#       2) 1) "1730548250000-0"
#          2) 1) "action"
#             2) "purchase"

# Read from a specific ID onwards
xread count 1 streams events 1730548200000-0
# Returns entries after the given ID
```

> **Key Insight:** `xread` is **non-destructive** — unlike `rpop` on a list, reading leaves the entries in place. Which is precisely why every consumer must remember the last ID it saw and pass it back on the next call.

### Blocking Reads

```bash
xread block 0 streams stream-name last-seen-id
```

Blocks the connection until new entries arrive — `0` means no timeout. This is how you build a real-time consumer without polling.

```bash
# Wait for new entries (blocks until new data arrives)
xread block 0 streams events $
# (blocks here until new entry is added)

# In another terminal, add a new entry:
# xadd events * action "comment" user_id "1003" text "Great product!"

# First terminal receives:
# 1) 1) "events"
#    2) 1) 1) "1730548400000-0"
#          2) 1) "action"
#             2) "comment"
#             3) "user_id"
#             4) "1003"

# Block with timeout (5 seconds)
xread block 5000 streams events $
# Returns (nil) after 5 seconds if no new entries
```

> **Note:** blocking ties up **that client connection**, not the server — Redis parks the client and keeps serving everyone else. Still, give a consumer its own connection rather than blocking the one your application uses for normal commands.

### Consumer Groups

A consumer group lets several workers **share** a stream: each message goes to exactly one member of the group, instead of every consumer seeing everything.

```bash
xgroup create stream-name group-name $ mkstream
```

`$` starts the group at new entries only; `mkstream` creates the stream if it doesn't exist yet.

```bash
# Create a consumer group for processing orders
xgroup create orders order-processors $ mkstream
# OK

# Create multiple consumer groups for different purposes
xgroup create events notification-service $ mkstream
# OK

xgroup create events analytics-service $ mkstream
# OK
```

```bash
xgroup createconsumer stream-name group-name member-group-name
```

```bash
# Create consumers in order-processors group
xgroup createconsumer orders order-processors worker-1
# (integer) 1

xgroup createconsumer orders order-processors worker-2
# (integer) 1

xgroup createconsumer orders order-processors worker-3
# (integer) 1
# Now you have 3 workers sharing the workload
```

> **Key Insight:** two groups on the same stream are independent — `notification-service` and `analytics-service` above each receive **every** event, while the three workers *inside* one group split the work between them. Groups fan out; consumers within a group divide up.

> **Note:** starting a group at `$` means it ignores everything already in the stream. Pass `0` instead when a new group should process the full history.

### Reading with a Consumer Group

```bash
xreadgroup group group-name member-group-name count 1 block 0 streams stream-name >
```

`count 1` limits the batch, `block 0` waits indefinitely, and `>` asks for messages never delivered to any consumer.

```bash
# Worker 1 reads from the group
xreadgroup group order-processors worker-1 count 1 block 0 streams orders >
# 1) 1) "orders"
#    2) 1) 1) "1730548500000-0"
#          2) 1) "order_id"
#             2) "ORD-001"
#             3) "amount"
#             4) "250000"

# Worker 2 reads from the group (gets different message)
xreadgroup group order-processors worker-2 count 1 block 0 streams orders >
# 1) 1) "orders"
#    2) 1) 1) "1730548501000-0"
#          2) 1) "order_id"
#             2) "ORD-002"
#             3) "amount"
#             4) "180000"

# Read multiple messages at once
xreadgroup group order-processors worker-3 count 5 block 0 streams orders >
# Returns up to 5 unprocessed messages

# Non-blocking read (returns immediately)
xreadgroup group order-processors worker-1 count 1 streams orders >
# Returns (nil) if no pending messages
```

> ⚠️ **Warning:** a message read with `>` moves into the group's **Pending Entries List** and stays there until the worker calls `xack`. Skip the acknowledgement and the message is never re-delivered and never cleared — it just accumulates. Inspect the backlog with `xpending`, and hand a crashed worker's messages to another one with `xclaim`.

---

## 🛠️ Practical Use Cases

### Real-time Chat System

```bash
# Add messages to chat stream
xadd chat:room1 * user "alice" message "Hello everyone!" timestamp "10:30:00"
xadd chat:room1 * user "bob" message "Hi Alice!" timestamp "10:30:15"

# Create consumer group for message delivery
xgroup create chat:room1 message-delivery $ mkstream

# Each client reads messages
xreadgroup group message-delivery client-alice count 10 streams chat:room1 >
```

### E-commerce Product Inventory

```bash
# Store product information
hset product:1001 name "Gaming Laptop" price "15000000" stock "50"
hset product:1002 name "Wireless Mouse" price "250000" stock "200"

# Update stock after purchase
hincrby product:1001 stock -1
hincrby product:1001 sold 1
```

### Gaming Leaderboard

```bash
# Add player scores
zadd game:leaderboard 1500 "player123"
zadd game:leaderboard 2300 "player456"
zadd game:leaderboard 1800 "player789"

# Get top 10 players
zrange game:leaderboard 0 9 rev withscores

# Get player rank
zrevrank game:leaderboard "player456"
```

### Location-based Service

```bash
# Add restaurant locations
geoadd restaurants 106.8456 -6.2088 "Restaurant A"
geoadd restaurants 106.8270 -6.1751 "Restaurant B"

# Find restaurants within 2km
geosearch restaurants fromlonlat 106.8456 -6.2088 byradius 2 km withdist
```

### Task Queue System

```bash
# Add tasks to queue
lpush task:queue "send-email:user123"
lpush task:queue "process-payment:order456"
lpush task:queue "generate-report:monthly"

# Worker processes tasks
rpop task:queue
# Process the task
```

### Session Management

```bash
# Store user session
setex session:abc123 3600 "user_id:1001"
hset user:session:abc123 user_id "1001" login_time "10:30:00" ip "192.168.1.100"

# Check session validity
ttl session:abc123
hgetall user:session:abc123
```

### Rate Limiting

```bash
# Track API requests
incr rate:api:user123
expire rate:api:user123 60

# Check if limit exceeded
get rate:api:user123
# If > 100, deny request
```

### Tag System

```bash
# Add tags to articles
sadd article:1001:tags "javascript" "redis" "tutorial"
sadd article:1002:tags "python" "redis" "guide"

# Find common tags
sinter article:1001:tags article:1002:tags
# 1) "redis"

# Find all unique tags
sunion article:1001:tags article:1002:tags
```

---

## 🆘 Common Pitfalls

**Avoid `keys` in production**

Use `scan` instead — it iterates with a cursor rather than blocking the server on the whole keyspace. The same applies per structure: `sscan`, `hscan`, `zscan`.

**Don't store oversized values**

A string value tops out at 512 MB, and a value anywhere near that blocks the single command thread while it is read or written. Split large data into chunks.

**Set appropriate TTLs**

Anything temporary needs an expiration, or it stays in memory forever. Note that a TTL belongs to the **key**, not to individual fields or members — you cannot expire one hash field or one set member.

**Handle empty results**

`nil`, an empty array, and `0` are normal replies, not errors. `rpop` on an empty list, `hget` on a missing field, and `xread block` on a timeout all return one of them.

**Use transactions carefully**

`MULTI`/`EXEC` runs a block without interleaving, but it is not a distributed lock and it does not roll back — a failing command inside the block leaves the others applied.

**Mind the type of an existing key**

Commands are type-specific. Running `lpush` against a key that already holds a hash fails with `WRONGTYPE` — Redis never converts a value between types.

---

## 🔁 Choosing a Data Structure

The whole guide collapses into one question: what shape is the data, and how will you read it back?

```
What are you storing?
  │
  ├─ An object with named fields         → Hash          hset / hget / hincrby
  │
  ├─ A collection of items
  │    ├─ Order matters, duplicates OK   → List          lpush / rpop / lrange
  │    ├─ Uniqueness matters, no order   → Set           sadd / sinter / sunion
  │    └─ Ranked by a number             → Sorted Set    zadd / zrange byscore
  │
  ├─ Points on a map                     → Geospatial    geoadd / geosearch
  │
  └─ A log of events, replayable         → Stream        xadd / xreadgroup
```

This explains the design decisions throughout this guide:

| Question | Answer |
| --- | --- |
| Why not build a leaderboard on a list? | A list has no scores — you would sort in application code on every read; `zrange` keeps the order maintained for you |
| Why a hash instead of many `user:1001:*` keys? | One key, one round trip, one TTL — and small hashes are stored in a memory-efficient encoding |
| Why a stream instead of a list for a queue? | A list pops destructively to one consumer; a stream keeps history, supports replay, and shares work across a consumer group |
| Why a set instead of de-duplicating in code? | `sadd` returns whether the member was new, so uniqueness is enforced server-side and atomically |
| Why is geospatial "just" a sorted set? | The coordinates are encoded into a geohash score, which is what makes radius searches a range query |
| Why does structure choice matter so much? | Redis has no query planner — the structure you pick *is* the access pattern you get |

---

## 🎯 Quick Reference

| Structure | Purpose | Key Commands |
| --- | --- | --- |
| **List** | Ordered sequence, push/pop from either end | `lpush`, `rpop`, `lpop`, `lrange` |
| **List as queue** | FIFO — first in, first out | `lpush tasks "job"` + `rpop tasks` |
| **List as stack** | LIFO — last in, first out | `lpush history "page"` + `lpop history` |
| **Set** | Unique, unordered members | `sadd`, `scard`, `smembers`, `srem` |
| **Set operations** | Compare collections on the server | `sdiff`, `sinter`, `sunion` |
| **Hash** | Field-value pairs in one key | `hset`, `hget`, `hgetall`, `hincrby` |
| **Sorted Set** | Members ranked by a numeric score | `zadd`, `zcard`, `zrem` |
| **Rank queries** | Read by position | `zrange key 0 9 [rev] [withscores]`, `zrevrank` |
| **Score queries** | Read or delete by score range | `zrange key 150 250 byscore`, `zremrangebyscore` |
| **Geospatial** | Coordinates and distance | `geoadd`, `geopos`, `geodist` |
| **Radius search** | Find what is nearby | `geosearch key fromlonlat lon lat byradius 5 km withdist` |
| **Stream** | Append-only event log | `xadd key * field value`, `xread` |
| **Stream cursors** | Where to start reading | `0` (beginning), `$` (new only), `>` (undelivered) |
| **Consumer group** | Share a stream across workers | `xgroup create`, `xgroup createconsumer` |
| **Group read** | Take the next unprocessed message | `xreadgroup group g worker count 1 block 0 streams key >` |
| **Safe iteration** | Walk a big collection without blocking | `scan`, `sscan`, `hscan`, `zscan` |

---

## 💡 Best Practices

**✅ Do This**

- **Pick the structure that matches the access pattern** — lists for queues and stacks, sets for unique collections, hashes for objects, sorted sets for rankings, geospatial for location queries, streams for event sourcing
- **Group an object's fields into one hash** rather than scattering them across `user:1001:name`, `user:1001:email` — one key, one round trip, one TTL, and a compact encoding for small objects
- **Name keys descriptively with colons** — `user:1001:profile`, `article:1001:tags` — and keep related keys under a shared prefix
- **Let the server do the set maths** with `sdiff` / `sinter` / `sunion` instead of pulling both collections to the client
- **Use `hincrby` and `zadd` for updates** — both are atomic, so concurrent writers cannot lose each other's changes
- **Set an expiration on everything temporary** — sessions, rate-limit counters, and caches should clean themselves up
- **Store a timestamp as a sorted-set score** when you need a time window — `zremrangebyscore key 0 <cutoff>` then expires the old entries in one command
- **Acknowledge stream messages with `xack`** and monitor the backlog with `xpending`
- **Cap streams at write time** with `xadd key maxlen ~ 10000 *`, or trim them with `xtrim`
- **Paginate large reads** — `lrange` a window, `zrange` a page, and `scan`-family commands for iteration
- **Pipeline bulk operations** so N commands cost one round trip

**❌ Avoid This**

- **Running `keys`, `smembers` or `hgetall` on huge collections in production** — each returns everything in one blocking reply; use the `scan` family instead
- **Relying on the order `smembers` returns** — a set is unordered, and the order can change between calls
- **Writing `geoadd key latitude longitude member`** — longitude comes **first**, and getting it backwards stores a wrong location silently
- **Expecting `lpop key value` to work** — the second argument is a count, not a value
- **Passing a member to `zremrangebyscore`** — the first argument is the key, then the score range
- **Reading a consumer-group message and never calling `xack`** — it sits in the Pending Entries List forever, neither re-delivered nor cleared
- **Letting a stream grow unbounded** — append-only means it never shrinks on its own
- **Treating `xread` like a queue pop** — it is non-destructive, so a consumer that forgets its last ID re-reads the same entries
- **Mixing types on one key** — `lpush` on a hash key fails with `WRONGTYPE`; Redis never converts
- **Storing values near the 512 MB ceiling** — split them, or the single command thread stalls on every access
- **Treating `MULTI`/`EXEC` as a distributed lock** — it prevents interleaving, nothing more, and it does not roll back

> Reference: [redis.io/docs/data-types](https://redis.io/docs/latest/develop/data-types/) · [redis.io/commands](https://redis.io/docs/latest/commands/)
