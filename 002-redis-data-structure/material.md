# 🗂️ Redis Data Structures Guide

A comprehensive guide to Redis data structures and their operations with practical examples.

---

## 📋 Table of Contents

- [Lists](#-lists)
- [Sets](#-sets)
- [Hash](#-hash)
- [Sorted Set](#-sorted-set)
- [Geospatial](#-geospatial)
- [Streams](#-streams)
- [Practical Use Case Examples](#-practical-use-case-examples)

---

## 📜 Lists

Lists are a data structure in the form of a linked list that contain string data. Lists are similar to arrays, where each element in a list has an index number.

**Common use cases:** message queues, task pipelines, and logs.

### 🧭 Queue (FIFO - First In, First Out)

The first element added is the first one removed.

- **Push value to the left (head) of list**
  
  ```bash
  lpush queue value
  ```
  
  Example:
  
  ```bash
  lpush tasks "send email"
  # (integer) 1
  
  lpush tasks "process payment"
  # (integer) 2
  
  lpush tasks "generate report"
  # (integer) 3
  
  # Queue now: ["generate report", "process payment", "send email"]
  ```

- **Remove and return value from the right (tail)**
  
  ```bash
  rpop queue
  ```
  
  Example:
  
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

### 🧱 Stack (LIFO - Last In, First Out)

The last element added is the first one removed.

- **Push value to the stack (left side)**
  
  ```bash
  lpush stack value
  ```
  
  Example:
  
  ```bash
  lpush history "page1.html"
  # (integer) 1
  
  lpush history "page2.html"
  # (integer) 2
  
  lpush history "page3.html"
  # (integer) 3
  
  # Stack now: ["page3.html", "page2.html", "page1.html"]
  ```

- **Pop value from the stack (left side)**
  
  ```bash
  lpop stack value
  ```
  
  Example:
  
  ```bash
  lpop history
  # "page3.html"  # Last in, first out
  
  lpop history
  # "page2.html"
  
  lpop history
  # "page1.html"
  ```

- **Get elements from list within range**
  
  ```bash
  lrange queue/stack from until
  ```
  
  Example:
  
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

---

## 🔷 Sets

Sets are unordered collections of unique string elements. Duplicate elements are automatically ignored.

**Common use cases:** tags, unique IDs, removing duplicates.

- **Add one or more members to a set**
  
  ```bash
  sadd key ...value
  ```
  
  Example:
  
  ```bash
  sadd tags "javascript" "redis" "database"
  # (integer) 3
  
  sadd tags "javascript"
  # (integer) 0  # Already exists, not added
  
  sadd tags "python" "golang"
  # (integer) 2
  ```

- **Get the number of members in a set**
  
  ```bash
  scard key
  ```
  
  Example:
  
  ```bash
  scard tags
  # (integer) 5
  ```

- **Get all members in a set**
  
  ```bash
  smembers key
  ```
  
  Example:
  
  ```bash
  smembers tags
  # 1) "redis"
  # 2) "javascript"
  # 3) "database"
  # 4) "python"
  # 5) "golang"
  ```

- **Remove one or more members from a set**
  
  ```bash
  srem key ...value
  ```
  
  Example:
  
  ```bash
  srem tags "golang"
  # (integer) 1
  
  srem tags "rust" "cpp"
  # (integer) 0  # Neither exists
  ```

- **Get the members of the set resulting from the difference**
  
  Between the first set and all the following sets
  
  ```bash
  sdiff key1 key2
  ```
  
  Example:
  
  ```bash
  sadd skills:alice "python" "javascript" "redis" "docker"
  sadd skills:bob "javascript" "golang" "redis"
  
  sdiff skills:alice skills:bob
  # 1) "python"
  # 2) "docker"
  # Skills that Alice has but Bob doesn't
  ```

- **Get the members of the set resulting from the intersection**
  
  Of all the given sets
  
  ```bash
  sinter key1 key2
  ```
  
  Example:
  
  ```bash
  sinter skills:alice skills:bob
  # 1) "javascript"
  # 2) "redis"
  # Common skills between Alice and Bob
  ```

- **Get the members of the set resulting from the union**
  
  Of all the given sets
  
  ```bash
  sunion key1 key2
  ```
  
  Example:
  
  ```bash
  sunion skills:alice skills:bob
  # 1) "python"
  # 2) "javascript"
  # 3) "redis"
  # 4) "docker"
  # 5) "golang"
  # All skills combined
  ```

---

## 🧩 Hash

Hashes are data structures composed of field-value pairs (like a JSON object). Useful for representing objects with multiple properties.

**Common use cases:** user profiles, products, configuration objects.

- **Set field(s) in a hash**
  
  ```bash
  hset hashname ...key value
  ```
  
  Example:
  
  ```bash
  hset user:1001 name "John Doe" email "john@example.com" age 30
  # (integer) 3
  
  hset user:1001 city "Jakarta"
  # (integer) 1
  
  hset product:2001 name "Laptop" price 15000000 stock 25
  # (integer) 3
  ```

- **Get value of a field in a hash**
  
  ```bash
  hget hashname key
  ```
  
  Example:
  
  ```bash
  hget user:1001 name
  # "John Doe"
  
  hget user:1001 email
  # "john@example.com"
  
  hget user:1001 nonexistent
  # (nil)
  ```

- **Get all fields and values in a hash**
  
  ```bash
  hgetall hashname
  ```
  
  Example:
  
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

- **Increment or decrement a field value in a hash**
  
  ```bash
  hincrby hashname key increment-value/decrement-value
  ```
  
  Example:
  
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

---

## 🏅 Sorted Set

Sorted Sets are similar to Sets but each member has a numeric score. Members are ordered by their score (ascending order). If scores are equal, members are sorted lexicographically.

**Common use cases:** leaderboards, rankings, time-based queues.

- **Add a member with a specific score to a sorted set**
  
  ```bash
  zadd key score member
  ```
  
  Example:
  
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

- **Get the number of members in a sorted set**
  
  ```bash
  zcard key
  ```
  
  Example:
  
  ```bash
  zcard leaderboard
  # (integer) 5
  ```

- **Get all members and their scores in ascending order**
  
  ```bash
  zrange key 0 -1 withscores
  ```
  
  Example:
  
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

- **Get members by index range**
  
  From lowest to highest score
  
  ```bash
  zrange key start-index end-index
  ```
  
  Example:
  
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

- **Get members by score range**
  
  From minimum to maximum score
  
  ```bash
  zrange key minimum-score maximum-score byscore
  ```
  
  Example:
  
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

- **Remove one or more members from a sorted set**
  
  ```bash
  zrem key ...member
  ```
  
  Example:
  
  ```bash
  zrem leaderboard "player1"
  # (integer) 1
  
  zrem leaderboard "player2" "player5"
  # (integer) 2
  ```

- **Remove members within a specific score range**
  
  ```bash
  zremrangebyscore member minimum-score maximum-score
  ```
  
  Example:
  
  ```bash
  # Remove all players with scores below 100
  zremrangebyscore leaderboard 0 100
  # (integer) 1
  
  # Remove players with scores between 150 and 200
  zremrangebyscore leaderboard 150 200
  # (integer) 2
  ```

---

## 📍 Geospatial

Geospatial structures are used to store geographic locations as coordinates. Ideal for applications that need distance or radius queries.

**Common use cases:** finding nearby stores, users, delivery tracking.

- **Add location coordinates to a geospatial key**
  
  ```bash
  geoadd key longitude latitude member
  ```
  
  Example:
  
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

- **Get position (longitude and latitude) of a member**
  
  ```bash
  geopos key member
  ```
  
  Example:
  
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

- **Get the distance between two members**
  
  ```bash
  geodist key member1 member2 distance-unit
  ```
  
  Example:
  
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

- **Search for members within a given radius**
  
  ```bash
  geosearch key fromlonlat longitude latitude BYRADIUS distance distance-unit
  ```
  
  Example:
  
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

---

## 🌊 Streams

Streams are an append-only data structure that works like a log or event stream. Each entry in a stream is a collection of key-value pairs and has a unique ID.

**Common use cases:** event sourcing, message brokers, data pipelines, and activity feeds.

- **Add an entry to a stream**
  
  ```bash
  xadd stream-name * key value
  ```
  
  Adds a new entry with an auto-generated ID ("*" uses current timestamp).
  
  Example:
  
  ```bash
  xadd events * action "user_login" user_id "1001" timestamp "2025-11-02T10:30:00"
  # "1730548200000-0"
  
  xadd events * action "purchase" user_id "1002" amount "150000"
  # "1730548250000-0"
  
  xadd events * action "logout" user_id "1001"
  # "1730548300000-0"
  ```

- **Read entries from a stream**
  
  ```bash
  xread streams stream-name 0
  ```
  
  Reads all entries starting from the beginning (ID 0).
  
  Example:
  
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

- **Read entries with a limit**
  
  Number of messages to read
  
  ```bash
  xread count value streams stream-name 0
  ```
  
  Reads up to 'value' number of entries from the stream, starting from ID 0. Useful when you only want to fetch a limited number of messages at a time.
  
  Example:
  
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

- **Block and wait for new entries**
  
  ```bash
  xread block 0 streams stream-name last-seen-id
  ```
  
  Blocks the connection and waits indefinitely (0 = no timeout) for new entries. Useful for implementing real-time consumers or message listeners. When new data arrives, Redis immediately returns it to the client.
  
  Example:
  
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

- **Create a new consumer group for a stream**
  
  ```bash
  xgroup create stream-name group-name $ mkstream
  ```
  
  Creates a consumer group for the specified stream. '$' means the group will start reading only new entries added after the group is created. 'mkstream' creates the stream automatically if it doesn't exist.
  
  Example:
  
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

- **Create a new consumer within an existing consumer group**
  
  ```bash
  xgroup createconsumer stream-name group-name member-group-name
  ```
  
  Registers a new consumer (member) inside the given consumer group. Useful when adding multiple consumers that will share the stream workload.
  
  Example:
  
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

- **Read messages using a consumer group**
  
  ```bash
  xreadgroup group group-name member-group-name count 1 block 0 streams stream-name >
  ```
  
  Reads messages as part of the specified consumer group. 'count 1' limits the number of messages fetched at once. 'block 0' waits indefinitely until a new message arrives. '>' means read only messages that have never been delivered to any consumer.
  
  Example:
  
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

---

## 💡 Practical Use Case Examples

### Example 1: Building a Real-time Chat System

- **Add messages to chat stream**
  
  ```bash
  xadd chat:room1 * user "alice" message "Hello everyone!" timestamp "10:30:00"
  xadd chat:room1 * user "bob" message "Hi Alice!" timestamp "10:30:15"
  ```

- **Create consumer group for message delivery**
  
  ```bash
  xgroup create chat:room1 message-delivery $ mkstream
  ```

- **Each client reads messages**
  
  ```bash
  xreadgroup group message-delivery client-alice count 10 streams chat:room1 >
  ```

### Example 2: E-commerce Product Inventory

- **Store product information**
  
  ```bash
  hset product:1001 name "Gaming Laptop" price "15000000" stock "50"
  hset product:1002 name "Wireless Mouse" price "250000" stock "200"
  ```

- **Update stock after purchase**
  
  ```bash
  hincrby product:1001 stock -1
  hincrby product:1001 sold 1
  ```

### Example 3: Gaming Leaderboard

- **Add player scores**
  
  ```bash
  zadd game:leaderboard 1500 "player123"
  zadd game:leaderboard 2300 "player456"
  zadd game:leaderboard 1800 "player789"
  ```

- **Get top 10 players**
  
  ```bash
  zrange game:leaderboard 0 9 rev withscores
  ```

- **Get player rank**
  
  ```bash
  zrevrank game:leaderboard "player456"
  ```

### Example 4: Location-based Service

- **Add restaurant locations**
  
  ```bash
  geoadd restaurants 106.8456 -6.2088 "Restaurant A"
  geoadd restaurants 106.8270 -6.1751 "Restaurant B"
  ```

- **Find restaurants within 2km**
  
  ```bash
  geosearch restaurants fromlonlat 106.8456 -6.2088 byradius 2 km withdist
  ```

### Example 5: Task Queue System

- **Add tasks to queue**
  
  ```bash
  lpush task:queue "send-email:user123"
  lpush task:queue "process-payment:order456"
  lpush task:queue "generate-report:monthly"
  ```

- **Worker processes tasks**
  
  ```bash
  rpop task:queue
  # Process the task
  ```

### Example 6: Session Management

- **Store user session**
  
  ```bash
  setex session:abc123 3600 "user_id:1001"
  hset user:session:abc123 user_id "1001" login_time "10:30:00" ip "192.168.1.100"
  ```

- **Check session validity**
  
  ```bash
  ttl session:abc123
  hgetall user:session:abc123
  ```

### Example 7: Rate Limiting

- **Track API requests**
  
  ```bash
  incr rate:api:user123
  expire rate:api:user123 60
  ```

- **Check if limit exceeded**
  
  ```bash
  get rate:api:user123
  # If > 100, deny request
  ```

### Example 8: Tag System

- **Add tags to articles**
  
  ```bash
  sadd article:1001:tags "javascript" "redis" "tutorial"
  sadd article:1002:tags "python" "redis" "guide"
  ```

- **Find common tags**
  
  ```bash
  sinter article:1001:tags article:1002:tags
  # 1) "redis"
  ```

- **Find all unique tags**
  
  ```bash
  sunion article:1001:tags article:1002:tags
  ```

---

## 🎯 Best Practices

- **Choose the right data structure**
  
  - Use Lists for queues and stacks
  - Use Sets for unique collections
  - Use Hashes for objects with multiple fields
  - Use Sorted Sets for rankings and leaderboards
  - Use Geospatial for location-based queries
  - Use Streams for event sourcing and message queues

- **Key naming conventions**
  
  - Use descriptive names with colons: `user:1001:profile`
  - Group related keys: `product:*`, `session:*`
  - Keep keys short but meaningful

- **Memory optimization**
  
  - Set expiration times for temporary data
  - Use Hashes for small objects (more memory efficient)
  - Clean up unused keys regularly
  - Monitor memory usage

- **Performance tips**
  
  - Use pipelining for bulk operations
  - Avoid large list operations (use pagination)
  - Use Sorted Sets instead of Lists for ordered data
  - Implement proper error handling

---

## 🆘 Common Pitfalls

- **Avoid using KEYS in production**
  
  Use SCAN instead for iterating over keys without blocking the server.

- **Don't store large values**
  
  Keep values under 512MB. Split large data into smaller chunks.

- **Set appropriate TTLs**
  
  Always set expiration for temporary data to prevent memory leaks.

- **Handle empty results**
  
  Check for nil/empty responses before processing data.

- **Use transactions carefully**
  
  Remember that MULTI/EXEC is not a distributed lock.