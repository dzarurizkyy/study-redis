# 🔴 Redis Hands-On Practice — E-Commerce Platform

A complete hands-on scenario that covers all core Redis concepts: server setup, key-value operations, string manipulation, data structures (Lists, Sets, Hashes, Sorted Sets, Geospatial, Streams), key expiration, transactions, persistence, memory management, and security — all in one real-world e-commerce use case.

---

## 📖 Scenario

You are building the backend cache and data layer for **ShopFast** — a fast-growing e-commerce platform. You will manage:

- **Products** — catalog data, pricing, stock levels
- **Users** — sessions, profiles, authentication tokens
- **Orders** — task queues, order pipelines
- **Leaderboard** — top-selling products ranking
- **Stores** — location-based store lookup
- **Events** — real-time activity stream (purchases, logins, clicks)

By the end of this practice, you will have hands-on experience with every concept covered in the Redis guides.

---

## 🗂️ Table of Contents

1. [Setup & Verify](#1-setup--verify)
2. [Basic Key-Value Operations](#2-basic-key-value-operations)
3. [String Manipulation & Multiple Operations](#3-string-manipulation--multiple-operations)
4. [Key Expiration & Session Management](#4-key-expiration--session-management)
5. [Increment / Decrement — Rate Limiting & Counters](#5-increment--decrement--rate-limiting--counters)
6. [Lists — Order Queue & Browse History](#6-lists--order-queue--browse-history)
7. [Sets — Tags & Unique Visitors](#7-sets--unique-visitors--tag-system)
8. [Hashes — Product & User Profiles](#8-hashes--product--user-profiles)
9. [Sorted Sets — Leaderboard](#9-sorted-sets--leaderboard)
10. [Geospatial — Store Finder](#10-geospatial--store-finder)
11. [Streams — Real-Time Event Log](#11-streams--real-time-event-log)
12. [Transactions](#12-transactions)
13. [Persistence](#13-persistence)
14. [Memory Management](#14-memory-management)
15. [Security & Authentication](#15-security--authentication)
16. [Monitoring & Debugging](#16-monitoring--debugging)
17. [Challenge Tasks](#-challenge-tasks)

---

## 1. Setup & Verify

### 1a. Start Redis Server

```bash
redis-server
```

Or start with a custom configuration file:

```bash
redis-server /etc/redis/redis.conf
```

### 1b. Connect with Redis CLI

```bash
redis-cli -h localhost -p 6379
```

Verify the connection:

```bash
PING
# PONG

INFO server
```

### 1c. Select a Database

Redis provides 16 databases by default (index 0–15). Use database `0` for products, database `1` for sessions.

```bash
SELECT 0
# OK  (products database)

SELECT 1
# OK  (sessions database)

SELECT 0
# Switch back to database 0
```

---

## 2. Basic Key-Value Operations

### 2a. Store Product Names

```bash
SET product:1001:name "Wireless Bluetooth Headphones"
# OK

SET product:1002:name "Mechanical Keyboard"
# OK

SET product:1003:name "4K USB-C Monitor"
# OK
```

### 2b. Retrieve Product Names

```bash
GET product:1001:name
# "Wireless Bluetooth Headphones"

GET product:1002:name
# "Mechanical Keyboard"
```

### 2c. Check If a Key Exists

```bash
EXISTS product:1001:name
# (integer) 1

EXISTS product:9999:name
# (integer) 0
```

### 2d. Delete a Key

```bash
SET product:temp "temporary product"
# OK

DEL product:temp
# (integer) 1

GET product:temp
# (nil)
```

### 2e. List Keys by Pattern

```bash
KEYS product:*
# 1) "product:1001:name"
# 2) "product:1002:name"
# 3) "product:1003:name"

KEYS product:1001:*
# 1) "product:1001:name"
```

> ⚠️ **Note:** Avoid using `KEYS` in production — it blocks the server. Use `SCAN` instead for iterating keys in a live environment.

### 2f. Append to a Key

```bash
SET product:1001:description "Noise-cancelling."
# OK

APPEND product:1001:description " 40-hour battery life."
# (integer) 36

GET product:1001:description
# "Noise-cancelling. 40-hour battery life."
```

---

## 3. String Manipulation & Multiple Operations

### 3a. Overwrite Part of a String with SETRANGE

```bash
SET product:1002:status "DRAFT   "
# OK

SETRANGE product:1002:status 0 "ACTIVE"
# (integer) 8

GET product:1002:status
# "ACTIVE  "
```

### 3b. Extract a Substring with GETRANGE

```bash
SET product:1001:sku "SKU-WBH-2024-US"
# OK

GETRANGE product:1001:sku 4 6
# "WBH"

GETRANGE product:1001:sku -2 -1
# "US"
```

### 3c. Set Multiple Keys at Once

```bash
MSET product:1001:price "79.99" product:1002:price "129.99" product:1003:price "349.99"
# OK
```

### 3d. Get Multiple Keys at Once

```bash
MGET product:1001:price product:1002:price product:1003:price
# 1) "79.99"
# 2) "129.99"
# 3) "349.99"
```

---

## 4. Key Expiration & Session Management

### 4a. Create a User Session with SETEX

When a user logs in, create a session that expires after 2 hours (7200 seconds).

```bash
SETEX session:user:alice 7200 "user_id:501"
# OK

SETEX session:user:bob 7200 "user_id:502"
# OK
```

### 4b. Check Remaining Time with TTL

```bash
TTL session:user:alice
# (integer) 7198  (seconds remaining)
```

**TTL return values:**
- Positive number: seconds until expiration
- `-1`: key exists but has no expiration
- `-2`: key does not exist

### 4c. Extend Session with EXPIRE

```bash
EXPIRE session:user:alice 7200
# (integer) 1  (reset to another 2 hours)
```

### 4d. Create a One-Time Auth Token

```bash
SETEX token:reset:alice 900 "abc123xyz789"
# OK  (expires in 15 minutes)

TTL token:reset:alice
# (integer) 899
```

### 4e. Verify Token Then Delete It

```bash
GET token:reset:alice
# "abc123xyz789"

DEL token:reset:alice
# (integer) 1  (token consumed — cannot be reused)
```

---

## 5. Increment / Decrement — Rate Limiting & Counters

### 5a. Track Page Views

```bash
SET stats:product:1001:views 0
# OK

INCR stats:product:1001:views
# (integer) 1

INCR stats:product:1001:views
# (integer) 2

INCRBY stats:product:1001:views 50
# (integer) 52
```

### 5b. Track Stock Levels

```bash
SET product:1001:stock 200
# OK

DECR product:1001:stock
# (integer) 199  (one item purchased)

DECRBY product:1001:stock 5
# (integer) 194  (bulk purchase of 5)

INCRBY product:1001:stock 100
# (integer) 294  (restocked)
```

### 5c. Implement API Rate Limiting

Limit each user to 100 API calls per minute.

```bash
INCR rate:api:user:alice
# (integer) 1

EXPIRE rate:api:user:alice 60
# (integer) 1  (resets every 60 seconds)

GET rate:api:user:alice
# "1"

# Keep calling INCR — when the value exceeds 100, deny the request
```

---

## 6. Lists — Order Queue & Browse History

### 6a. Queue (FIFO) — Order Processing Pipeline

Orders are added to the queue and processed in the order they arrive.

```bash
# Customers place orders (push to left = newest at head)
LPUSH queue:orders "order:ORD-001:user:alice"
# (integer) 1

LPUSH queue:orders "order:ORD-002:user:bob"
# (integer) 2

LPUSH queue:orders "order:ORD-003:user:carol"
# (integer) 3

# Queue state: ["ORD-003", "ORD-002", "ORD-001"]
```

```bash
# Worker picks up the oldest order (pop from right)
RPOP queue:orders
# "order:ORD-001:user:alice"

RPOP queue:orders
# "order:ORD-002:user:bob"
```

### 6b. Stack (LIFO) — Recently Viewed Products

A user's browser history shows the most recently visited product first.

```bash
# User alice views products
LPUSH history:user:alice "product:1001"
# (integer) 1

LPUSH history:user:alice "product:1003"
# (integer) 2

LPUSH history:user:alice "product:1002"
# (integer) 3

# History: ["product:1002", "product:1003", "product:1001"]
```

```bash
# Get last viewed product (pop from left)
LPOP history:user:alice
# "product:1002"
```

### 6c. View Items in a List

```bash
# View all remaining history
LRANGE history:user:alice 0 -1
# 1) "product:1003"
# 2) "product:1001"

# View first 2 items
LRANGE history:user:alice 0 1
# 1) "product:1003"
# 2) "product:1001"
```

---

## 7. Sets — Unique Visitors & Tag System

### 7a. Track Unique Visitors Per Product

Sets automatically ignore duplicates — perfect for unique visitor counts.

```bash
SADD visitors:product:1001 "user:alice" "user:bob" "user:carol"
# (integer) 3

SADD visitors:product:1001 "user:alice"
# (integer) 0  (alice already exists, not re-added)

SADD visitors:product:1001 "user:dave"
# (integer) 1

SCARD visitors:product:1001
# (integer) 4  (unique visitors)

SMEMBERS visitors:product:1001
# 1) "user:alice"
# 2) "user:bob"
# 3) "user:carol"
# 4) "user:dave"
```

### 7b. Tag Products

```bash
SADD tags:product:1001 "electronics" "audio" "wireless" "sale"
# (integer) 4

SADD tags:product:1002 "electronics" "peripherals" "mechanical" "gaming"
# (integer) 4

SADD tags:product:1003 "electronics" "display" "4k" "usb-c"
# (integer) 4
```

### 7c. Find Common Tags (Intersection)

```bash
SINTER tags:product:1001 tags:product:1002
# 1) "electronics"
# Common tags between headphones and keyboard
```

### 7d. Find All Tags Combined (Union)

```bash
SUNION tags:product:1001 tags:product:1002
# 1) "electronics"
# 2) "audio"
# 3) "wireless"
# 4) "sale"
# 5) "peripherals"
# 6) "mechanical"
# 7) "gaming"
```

### 7e. Find Tags Unique to One Product (Difference)

```bash
SDIFF tags:product:1001 tags:product:1002
# 1) "audio"
# 2) "wireless"
# 3) "sale"
# Tags on product:1001 that are NOT on product:1002
```

### 7f. Remove a Tag

```bash
SREM tags:product:1001 "sale"
# (integer) 1

SMEMBERS tags:product:1001
# 1) "electronics"
# 2) "audio"
# 3) "wireless"
```

---

## 8. Hashes — Product & User Profiles

### 8a. Store Full Product Details

```bash
HSET product:1001 name "Wireless Bluetooth Headphones" price "79.99" stock 200 category "electronics" brand "SoundPro" rating "4.5"
# (integer) 6

HSET product:1002 name "Mechanical Keyboard" price "129.99" stock 85 category "peripherals" brand "TypeMaster" rating "4.7"
# (integer) 6

HSET product:1003 name "4K USB-C Monitor" price "349.99" stock 40 category "display" brand "VisionEdge" rating "4.8"
# (integer) 6
```

### 8b. Get a Specific Field

```bash
HGET product:1001 price
# "79.99"

HGET product:1001 brand
# "SoundPro"
```

### 8c. Get All Fields

```bash
HGETALL product:1001
# 1) "name"
# 2) "Wireless Bluetooth Headphones"
# 3) "price"
# 4) "79.99"
# 5) "stock"
# 6) "200"
# 7) "category"
# 8) "electronics"
# 9) "brand"
# 10) "SoundPro"
# 11) "rating"
# 12) "4.5"
```

### 8d. Update Stock After a Purchase

```bash
HINCRBY product:1001 stock -3
# (integer) 197  (3 units sold)

HINCRBY product:1001 stock 50
# (integer) 247  (50 units restocked)
```

### 8e. Store User Profile

```bash
HSET user:501 username "alice" email "alice@shopfast.com" full_name "Alice Johnson" city "New York" loyalty_points 1200 total_orders 15
# (integer) 6
```

### 8f. Award Loyalty Points After Purchase

```bash
HINCRBY user:501 loyalty_points 150
# (integer) 1350

HINCRBY user:501 total_orders 1
# (integer) 16
```

---

## 9. Sorted Sets — Leaderboard

### 9a. Add Products to Sales Leaderboard

```bash
ZADD leaderboard:sales 4520 "product:1001"
# (integer) 1

ZADD leaderboard:sales 8730 "product:1002"
# (integer) 1

ZADD leaderboard:sales 1260 "product:1003"
# (integer) 1

ZADD leaderboard:sales 6100 "product:1004" 3340 "product:1005"
# (integer) 2
```

### 9b. View Full Leaderboard (Ascending)

```bash
ZRANGE leaderboard:sales 0 -1 WITHSCORES
# 1) "product:1003"
# 2) "1260"
# 3) "product:1005"
# 4) "3340"
# 5) "product:1001"
# 6) "4520"
# 7) "product:1004"
# 8) "6100"
# 9) "product:1002"
# 10) "8730"
```

### 9c. Get Top 3 Best-Selling Products

```bash
ZRANGE leaderboard:sales -3 -1
# 1) "product:1001"
# 2) "product:1004"
# 3) "product:1002"
# (Highest 3 scores — reversed from bottom)
```

### 9d. Get Products Within a Sales Range

```bash
ZRANGE leaderboard:sales 3000 7000 BYSCORE
# 1) "product:1005"
# 2) "product:1001"
# 3) "product:1004"
```

### 9e. Count Members in Set

```bash
ZCARD leaderboard:sales
# (integer) 5
```

### 9f. Update a Score (New Sale Recorded)

```bash
# Add 200 more units sold to product:1003
ZADD leaderboard:sales 1460 "product:1003"
# (integer) 0  (updated, not added)
```

### 9g. Remove a Product From the Leaderboard

```bash
ZREM leaderboard:sales "product:1005"
# (integer) 1
```

### 9h. Remove Products With Low Sales

```bash
ZREMRANGEBYSCORE leaderboard:sales 0 1000
# (integer) 0  (none in that range after updates)
```

---

## 10. Geospatial — Store Finder

### 10a. Add Store Locations

```bash
GEOADD stores:shopfast -73.9857 40.7484 "ShopFast NYC Midtown"
# (integer) 1

GEOADD stores:shopfast -73.9442 40.6782 "ShopFast NYC Brooklyn"
# (integer) 1

GEOADD stores:shopfast -118.2437 34.0522 "ShopFast LA Downtown"
# (integer) 1

GEOADD stores:shopfast -87.6298 41.8781 "ShopFast Chicago"
# (integer) 1
```

### 10b. Get Store Coordinates

```bash
GEOPOS stores:shopfast "ShopFast NYC Midtown"
# 1) 1) "-73.98569971323013306"
#    2) "40.74840000534994929"
```

### 10c. Get Distance Between Stores

```bash
GEODIST stores:shopfast "ShopFast NYC Midtown" "ShopFast NYC Brooklyn" km
# "6.4832"

GEODIST stores:shopfast "ShopFast NYC Midtown" "ShopFast NYC Brooklyn" mi
# "4.0285"
```

### 10d. Find Stores Within 10km of a User's Location

Alice is at coordinates (-73.97, 40.76) — near Midtown Manhattan.

```bash
GEOSEARCH stores:shopfast FROMLONLAT -73.97 40.76 BYRADIUS 10 km WITHDIST
# 1) 1) "ShopFast NYC Midtown"
#    2) "1.3421"
# 2) 1) "ShopFast NYC Brooklyn"
#    2) "8.2647"
```

### 10e. Find Stores With Coordinates

```bash
GEOSEARCH stores:shopfast FROMLONLAT -73.97 40.76 BYRADIUS 10 km WITHCOORD WITHDIST
# 1) 1) "ShopFast NYC Midtown"
#    2) "1.3421"
#    3) 1) "-73.98569971323013306"
#       2) "40.74840000534994929"
```

---

## 11. Streams — Real-Time Event Log

### 11a. Add Events to the Stream

```bash
XADD events:shopfast * action "user_login" user_id "501" ip "192.168.1.10"
# "1730548200000-0"

XADD events:shopfast * action "product_view" user_id "501" product_id "1001"
# "1730548230000-0"

XADD events:shopfast * action "add_to_cart" user_id "501" product_id "1001" qty "2"
# "1730548260000-0"

XADD events:shopfast * action "purchase" user_id "501" order_id "ORD-001" total "159.98"
# "1730548300000-0"

XADD events:shopfast * action "user_login" user_id "502" ip "10.0.0.25"
# "1730548310000-0"
```

### 11b. Read All Events From the Beginning

```bash
XREAD STREAMS events:shopfast 0
# Returns all entries since ID 0
```

### 11c. Read Events With a Limit

```bash
XREAD COUNT 2 STREAMS events:shopfast 0
# Returns only the first 2 events
```

### 11d. Block and Wait for New Events

```bash
# Subscribe and wait for new events in real time
XREAD BLOCK 0 STREAMS events:shopfast $
# (blocks until a new event is added)

# In another terminal, add a new event:
# XADD events:shopfast * action "review_posted" user_id "503" product_id "1002" stars "5"
```

### 11e. Create Consumer Groups for Event Processing

```bash
# Notification service — sends email/SMS on purchases
XGROUP CREATE events:shopfast notification-service $ MKSTREAM
# OK

# Analytics service — records stats
XGROUP CREATE events:shopfast analytics-service $ MKSTREAM
# OK
```

### 11f. Add Consumers Within a Group

```bash
XGROUP CREATECONSUMER events:shopfast notification-service worker-notif-1
# (integer) 1

XGROUP CREATECONSUMER events:shopfast analytics-service worker-analytics-1
# (integer) 1

XGROUP CREATECONSUMER events:shopfast analytics-service worker-analytics-2
# (integer) 1
```

### 11g. Consume Events With Consumer Groups

```bash
# Notification worker reads a new event
XREADGROUP GROUP notification-service worker-notif-1 COUNT 1 BLOCK 0 STREAMS events:shopfast >
# Returns the next undelivered event

# Analytics worker-1 reads a different event from the same stream
XREADGROUP GROUP analytics-service worker-analytics-1 COUNT 1 BLOCK 0 STREAMS events:shopfast >

# Analytics worker-2 reads in parallel (gets a different message)
XREADGROUP GROUP analytics-service worker-analytics-2 COUNT 1 BLOCK 0 STREAMS events:shopfast >
```

> 💡 Each consumer group processes the stream independently. The notification service and analytics service each see all events. Within a group, messages are distributed across workers so no two workers process the same event.

---

## 12. Transactions

Transactions ensure that a set of operations run atomically — either all succeed or none do.

### 12a. Atomic Stock Deduction & Order Recording

```bash
MULTI
# OK

SET order:ORD-005:status "confirmed"
# QUEUED

HINCRBY product:1001 stock -2
# QUEUED

HINCRBY user:501 total_orders 1
# QUEUED

HINCRBY user:501 loyalty_points 80
# QUEUED

EXEC
# 1) OK
# 2) (integer) 245
# 3) (integer) 17
# 4) (integer) 1430
```

### 12b. Discard a Transaction

```bash
MULTI
# OK

SET product:1001:price "99.99"
# QUEUED

SET product:1002:price "149.99"
# QUEUED

DISCARD
# OK  (neither price was changed)

GET product:1001:price
# "79.99"  (original price unchanged)
```

---

## 13. Persistence

### 13a. Manual Snapshot

```bash
SAVE
# OK  (blocks server until done)
```

> ⚠️ Use `BGSAVE` in production — `SAVE` blocks the server.

### 13b. Background Snapshot

```bash
BGSAVE
# Background saving started
```

### 13c. Configure Automatic Snapshots

Add to `redis.conf`:

```bash
save 900 1       # Snapshot if at least 1 key changed in 15 minutes
save 300 10      # Snapshot if at least 10 keys changed in 5 minutes
save 60 10000    # Snapshot if at least 10,000 keys changed in 1 minute
```

---

## 14. Memory Management

### 14a. Check Configuration

```bash
CONFIG GET maxmemory
# 1) "maxmemory"
# 2) "0"  (0 = unlimited by default)

CONFIG GET maxmemory-policy
# 1) "maxmemory-policy"
# 2) "noeviction"
```

### 14b. Set Memory Limit (in redis.conf)

```bash
maxmemory 512mb
```

### 14c. Set Eviction Policy (in redis.conf)

For ShopFast's product catalog cache:

```bash
maxmemory-policy allkeys-lru
# Evict the least recently used keys when memory is full
# Best for general-purpose caching
```

**Policy reference:**

| Policy | Description | Best For |
|---|---|---|
| `noeviction` | Return error when memory is full | Critical data that must not be lost |
| `allkeys-lru` | Evict least recently used among all keys | General-purpose caching |
| `volatile-lru` | Evict least recently used keys that have a TTL | Mixed persistent + cache workloads |
| `allkeys-lfu` | Evict least frequently used among all keys | Frequency-based caching |
| `volatile-lfu` | Evict least frequently used keys with TTL | Mixed workloads with frequency patterns |
| `volatile-random` | Evict random keys with TTL | Unpredictable access patterns |
| `allkeys-random` | Evict random keys | All keys have equal importance |
| `volatile-ttl` | Evict keys with the shortest remaining TTL | Time-sensitive cache data |

---

## 15. Security & Authentication

### 15a. Bind to a Specific Interface (redis.conf)

```bash
bind 127.0.0.1
# Only accept connections from localhost — never expose Redis directly to the internet
```

### 15b. Create Users with ACL (redis.conf)

```bash
# Restrict default user to connection only
user default on +@connection

# Admin user — full access
user shopfast_admin on +@all ~* >AdminPass!2024

# Read-only reporting user — can only read
user shopfast_reader on +@read ~* >ReaderPass!2024

# App service user — read/write but cannot run admin commands
user shopfast_app on +@read +@write ~* >AppPass!2024
```

### 15c. Authenticate from Redis CLI

```bash
AUTH shopfast_admin AdminPass!2024
# OK
```

---

## 16. Monitoring & Debugging

### 16a. Monitor All Commands in Real Time

```bash
MONITOR
# (watching all incoming commands — press Ctrl+C to stop)
```

### 16b. Get Server Statistics

```bash
INFO
# Full server stats

INFO memory
# Memory usage only

INFO stats
# Command stats

INFO server
# Version, OS, uptime
```

### 16c. Get a Configuration Value

```bash
CONFIG GET databases
# 1) "databases"
# 2) "16"

CONFIG GET maxmemory
# 1) "maxmemory"
# 2) "536870912"
```

### 16d. Manage Client Connections

```bash
CLIENT LIST
# Lists all connected clients

CLIENT ID
# Returns ID of the current connection

CLIENT KILL 192.168.1.55:52345
# Disconnects a specific client
```

### 16e. Database Size

```bash
DBSIZE
# (integer) 42  (number of keys in current database)
```

---

## 🏆 Challenge Tasks

Once you have completed the practice above, try these on your own:

---

### Challenge 1 — Session & Expiration

Create a login session for user `carol` (user_id: 503) that expires in 30 minutes. Then simulate a session extension (add 30 more minutes) and verify the remaining TTL after each step.

---

### Challenge 2 — Rate Limiting

Simulate 5 consecutive API calls from user `bob`. After each call, print the current count and the TTL. Write the logic that would deny access once the count exceeds 3 calls per minute.

---

### Challenge 3 — Order Queue

Add 5 orders to `queue:orders` using `LPUSH`. Then simulate 3 workers each popping one order using `RPOP`. Verify the remaining queue with `LRANGE`.

---

### Challenge 4 — Tag Intersection

Add tags to 3 products:
- `product:2001` → `"electronics" "smart" "iot" "sale"`
- `product:2002` → `"electronics" "smart" "home" "sale"`
- `product:2003` → `"electronics" "home" "appliance"`

Find all tags shared by all three products using `SINTER`. Then find all unique tags across all three using `SUNION`.

---

### Challenge 5 — Leaderboard Query

Add 8 products to `leaderboard:weekly` with varied scores. Then:
1. Retrieve the top 3 products
2. Retrieve all products with scores between 2000 and 5000
3. Remove all products with scores below 1000

---

### Challenge 6 — Hash Inventory

Create product hashes for 3 new products with fields: `name`, `price`, `stock`, `category`. Then simulate a flash sale:
- Deduct 10 units from each product's stock using `HINCRBY`
- Update the price of one product using `HSET`
- Verify all changes with `HGETALL`

---

### Challenge 7 — Store Finder

Add 4 store locations (you can use real or fictional cities with approximate coordinates). Then search for all stores within 50km of a central coordinate using `GEOSEARCH`. Include distances in the result.

---

### Challenge 8 — Atomic Transaction

Using `MULTI` / `EXEC`, perform the following atomically in a single transaction:
- Deduct 1 unit of stock from `product:1002`
- Record a new order key `order:ORD-999:status` as `"paid"`
- Increment `user:501`'s `total_orders` by 1
- Award 200 loyalty points to `user:501`

Verify each value after `EXEC`.

---

### Challenge 9 — Stream Consumer Group

Create a stream called `events:payments` and add 5 payment events. Create a consumer group `payment-processors` with 2 workers (`worker-A` and `worker-B`). Have each worker read 2 messages and verify that no message is delivered to both workers.

---

### Challenge 10 — Memory Policy

Using `CONFIG GET`, check the current `maxmemory-policy`. Research what would happen to ShopFast's session keys (which all have TTLs) if the policy were set to `volatile-lru` vs `allkeys-lru`. Write your reasoning as a comment.

---

## ✅ Concepts Covered

| Concept | Where Practiced |
|---|---|
| Start Redis & connect | Step 1 |
| SELECT database | Step 1c |
| SET, GET, DEL, EXISTS | Step 2 |
| KEYS pattern | Step 2e |
| APPEND | Step 2f |
| SETRANGE, GETRANGE | Step 3a–3b |
| MSET, MGET | Step 3c–3d |
| SETEX, EXPIRE, TTL | Step 4 |
| INCR, DECR, INCRBY, DECRBY | Step 5 |
| LPUSH, RPOP (Queue / FIFO) | Step 6a |
| LPUSH, LPOP (Stack / LIFO) | Step 6b |
| LRANGE | Step 6c |
| SADD, SCARD, SMEMBERS, SREM | Step 7a–7b, 7f |
| SDIFF, SINTER, SUNION | Step 7c–7e |
| HSET, HGET, HGETALL, HINCRBY | Step 8 |
| ZADD, ZRANGE, ZCARD, ZREM | Step 9 |
| ZRANGE BYSCORE, ZREMRANGEBYSCORE | Step 9d, 9h |
| GEOADD, GEOPOS, GEODIST | Step 10a–10c |
| GEOSEARCH BYRADIUS | Step 10d–10e |
| XADD, XREAD, XREAD COUNT | Step 11a–11c |
| XREAD BLOCK | Step 11d |
| XGROUP CREATE, CREATECONSUMER | Step 11e–11f |
| XREADGROUP | Step 11g |
| MULTI, EXEC, DISCARD | Step 12 |
| SAVE, BGSAVE, save config | Step 13 |
| maxmemory, maxmemory-policy | Step 14 |
| bind, ACL user config, AUTH | Step 15 |
| MONITOR, INFO, CONFIG GET | Step 16a–16c |
| CLIENT LIST, CLIENT KILL | Step 16d |
| DBSIZE | Step 16e |
