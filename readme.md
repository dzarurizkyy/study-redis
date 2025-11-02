# Study Redis 🔴

This repository contains comprehensive examples of Redis commands, data structures, and best practices. It includes hands-on exercises covering basic operations, advanced data structures, persistence, security, and practical scenarios to strengthen your Redis skills.

## Installation 🔧

### Option 1: Using Package Manager

**macOS (Homebrew):**
```
brew install redis
```

**Ubuntu/Debian:**
```
sudo apt update
sudo apt install redis-server
```

**Windows:**
- Download [Redis for Windows](https://github.com/microsoftarchive/redis/releases)
- Or use [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install) with Linux installation

### Option 2: Using Docker
```
docker pull redis
docker run -d -p 6379:6379 --name my-redis redis
```

### Verify Installation
```
redis-server --version
redis-cli ping
# Expected output: PONG
```

## List of Material 📚

### 📘 **Redis Basics**
Contains fundamental Redis commands including server management, basic operations, and configuration:

```
# Start Redis Server
redis-server

# Connect to Redis CLI
redis-cli -h hostname -p port

# Basic key-value operations
set user:1 "John Doe"
get user:1
exists user:1
del user:1

# Key expiration
expire session:123 3600
ttl session:123

# Increment/Decrement
incr page_views
decr stock:item1
```

**Topics covered:**
- Server management and configuration
- Key-value operations (SET, GET, DEL, EXISTS)
- String manipulation (APPEND, SETRANGE, GETRANGE)
- Multiple operations (MSET, MGET)
- Key expiration and TTL
- Increment/Decrement operations
- Transactions (MULTI, EXEC, DISCARD)
- Persistence (SAVE, BGSAVE)
- Security and authentication
- Memory management and eviction policies

### 📗 **Redis Data Structures**
Demonstrates how to work with Redis advanced data structures and their practical applications:

```
# Lists - Queue (FIFO)
lpush tasks "task1"
lpush tasks "task2"
rpop tasks

# Sets - Unique collections
sadd tags "redis" "database" "cache"
smembers tags
sinter tags1 tags2

# Hashes - Object storage
hset user:1 name "Alice" age 25 email "alice@example.com"
hget user:1 name
hgetall user:1

# Sorted Sets - Leaderboard
zadd leaderboard 100 "player1"
zadd leaderboard 200 "player2"
zrange leaderboard 0 -1 withscores

# Geospatial - Location tracking
geoadd stores -6.175 106.827 "Store A"
geoadd stores -6.200 106.850 "Store B"
geodist stores "Store A" "Store B" km

# Streams - Event log
xadd events * action "login" user "john" timestamp "2024-01-01"
xread count 10 streams events 0
```

**Topics covered:**
- **Lists**: Queue (FIFO) and Stack (LIFO) implementations
- **Sets**: Unique collections with set operations (union, intersection, difference)
- **Hashes**: Object representation with field-value pairs
- **Sorted Sets**: Ordered collections with scores for rankings
- **Geospatial**: Location-based queries and distance calculations
- **Streams**: Event sourcing and message streaming with consumer groups


## 📍 References
* [Udemy](https://www.udemy.com/course/belajar-redis)


## 👨‍💻 Contributors
* [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)