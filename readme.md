# Study Redis 🔴

A comprehensive Redis reference guide covering server management, core commands, all six data structures, transactions, persistence, security, and memory management.

## Installation 🔧

1. **Install Redis** (choose one):

   `macOS (Homebrew)`

   ```bash
   brew install redis
   ```

   `Ubuntu / Debian`

   ```bash
   sudo apt update
   sudo apt install redis-server
   ```

   `Windows`
   - Download [Redis for Windows](https://github.com/microsoftarchive/redis/releases)
   - Or use [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install) with the Linux installation

   `Docker`

   ```bash
   docker pull redis
   docker run -d -p 6379:6379 --name my-redis redis
   ```

2. **Run the Redis Server**:

   ```bash
   # Start the server
   redis-server

   # Connect with the CLI (default: 127.0.0.1:6379)
   redis-cli -h hostname -p port
   ```

3. **Verify the Installation**:

   ```bash
   redis-server --version

   redis-cli ping
   # Expected output: PONG
   ```

## List of Material 📚

- 🔴 **[Redis Basics](001-redis-basics.md)**

  Hands-on `redis-cli` guide covering server and database management, key-value operations, string manipulation, expiration, counters, transactions, monitoring, ACL security, persistence, and eviction policies.

  ```bash
  # Basic key-value operations
  set user:1 "John Doe"
  get user:1
  exists user:1
  del user:1

  # Key expiration
  setex session:123 3600 "user_id:1001"
  ttl session:123

  # Atomic counters
  incr page_views
  decrby stock:item1 5
  ```

  Configure the server in `redis.conf`:

  ```bash
  # Bind to a specific interface and lock down the default user
  bind 127.0.0.1
  user default on +@connection
  user admin on +@all ~* >StrongPassword123

  # Snapshot and cap memory
  save 900 1
  maxmemory 256mb
  maxmemory-policy allkeys-lru
  ```

- 🗂️ **[Redis Data Structures](002-redis-data-structure.md)**

  Lists, sets, hashes, sorted sets, geospatial indexes, and streams — the commands for each one, what they return, and the use cases they are built for, from task queues to leaderboards and consumer groups.

  ```bash
  # Lists — queue (FIFO)
  lpush tasks "send email"
  rpop tasks

  # Sets — unique collections
  sadd tags "redis" "database" "cache"
  sinter article:1001:tags article:1002:tags

  # Hashes — object storage
  hset user:1 name "Alice" age 25 email "alice@example.com"
  hincrby user:1 login_count 1

  # Sorted Sets — leaderboard
  zadd leaderboard 250 "player2"
  zrange leaderboard 0 9 rev withscores

  # Geospatial — longitude first, then latitude
  geoadd stores 106.8270 -6.1751 "Store A"
  geosearch stores fromlonlat 106.8456 -6.2088 byradius 5 km withdist

  # Streams — event log with consumer groups
  xadd events * action "login" user "john"
  xreadgroup group order-processors worker-1 count 1 block 0 streams orders >
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/belajar-redis)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
