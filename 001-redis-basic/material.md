# 🔴 Redis Command Cheatsheet

A comprehensive guide to Redis commands and configurations.

---

## 📋 Table of Contents

- [Server Management](#-server-management)
- [Database Configuration](#-database-configuration)
- [Basic Key-Value Operations](#-basic-key-value-operations)
- [String Manipulation](#-string-manipulation)
- [Multiple Operations](#-multiple-operations)
- [Key Expiration](#-key-expiration)
- [Increment/Decrement Operations](#-incrementdecrement-operations)
- [Database Management](#-database-management)
- [Transactions](#-transactions)
- [Monitoring & Debugging](#-monitoring--debugging)
- [Client Management](#-client-management)
- [Security & Authentication](#-security--authentication)
- [Persistence](#-persistence)
- [Memory Management](#-memory-management)

---

## 🖥️ Server Management

- **Start Redis Server (default configuration)**
  
  ```
  redis-server
  ```

- **Connect to Redis CLI**
  
  ```
  redis-cli -h hostname -p port
  ```

- **Start Redis Server with a custom configuration file**
  
  ```
  redis-server path/redis.conf
  ```

---

## 🗄️ Database Configuration

- **Set number of Redis databases**
  
  Default: 16 databases (set in redis.conf)
  
  ```
  databases 16
  ```

- **Select a database index**
  
  ```
  select index
  ```
  
  Example:
  
  ```
  select 0
  select 1
  ```

---

## 🔑 Basic Key-Value Operations

- **Store key-value pair**
  
  Overwrites if key already exists
  
  ```
  set key value
  ```
  
  Example:
  
  ```
  set username "john_doe"
  set counter 100
  ```

- **Get the value of a key**
  
  ```
  get key
  ```
  
  Example:
  
  ```
  get username
  ```

- **Check if a key exists**
  
  Returns 1 if exists, 0 if does not exist
  
  ```
  exists key
  ```
  
  Example:
  
  ```
  exists username
  ```

- **Delete one or more keys**
  
  Returns 1 if deleted, 0 if not found
  
  ```
  del key [key ...]
  ```
  
  Example:
  
  ```
  del username
  del key1 key2 key3
  ```

- **Append a value to an existing key**
  
  Creates key if it does not exist
  
  ```
  append key value
  ```
  
  Example:
  
  ```
  append message "Hello"
  append message " World"
  ```

- **List keys matching a given pattern**
  
  ```
  keys pattern
  ```
  
  Example:
  
  ```
  keys *
  keys user:*
  keys *name*
  ```

---

## ✂️ String Manipulation

- **Overwrite part of a string**
  
  Starting at the given offset
  
  ```
  setrange key offset value
  ```
  
  Example:
  
  ```
  set greeting "Hello World"
  setrange greeting 6 "Redis"
  ```

- **Get a substring of the value**
  
  By start and end index
  
  ```
  getrange key start end
  ```
  
  Example:
  
  ```
  getrange greeting 0 4
  getrange greeting -5 -1
  ```

---

## 📦 Multiple Operations

- **Set multiple key-value pairs**
  
  ```
  mset key value [key value ...]
  ```
  
  Example:
  
  ```
  mset username "john" email "john@example.com" age 25
  ```

- **Get values of multiple keys**
  
  ```
  mget key [key ...]
  ```
  
  Example:
  
  ```
  mget username email age
  ```

---

## ⏰ Key Expiration

- **Set a key's expiration time**
  
  Time in seconds
  
  ```
  expire key seconds
  ```
  
  Example:
  
  ```
  expire session:abc123 3600
  ```

- **Set the value and expiration time for a key**
  
  Time in seconds
  
  ```
  setex key seconds value
  ```
  
  Example:
  
  ```
  setex token:xyz 7200 "secret_token_value"
  ```

- **Get the remaining time-to-live of a key**
  
  Time in seconds
  
  ```
  ttl key
  ```
  
  Example:
  
  ```
  ttl session:abc123
  ```
  
  **Return values:**
  - Positive number: seconds until expiration
  - `-1`: key exists but has no expiration
  - `-2`: key does not exist

---

## ➕ Increment/Decrement Operations

- **Increment the integer value of a key by one**
  
  ```
  incr key
  ```
  
  Example:
  
  ```
  incr page_views
  incr visitor_count
  ```

- **Decrement the integer value of a key by one**
  
  ```
  decr key
  ```
  
  Example:
  
  ```
  decr stock_count
  ```

- **Increment by the given amount**
  
  ```
  incrby key increment
  ```
  
  Example:
  
  ```
  incrby score 10
  incrby balance 500
  ```

- **Decrement by the given amount**
  
  ```
  decrby key increment
  ```
  
  Example:
  
  ```
  decrby stock 5
  decrby credits 100
  ```

---

## 🗑️ Database Management

- **Remove all keys from the current database**
  
  ```
  flushdb
  ```
  
  ⚠️ **Warning:** This action is irreversible!

- **Remove all keys from all databases**
  
  ```
  flushall
  ```
  
  ⚠️ **Warning:** This action is irreversible and affects all databases!

- **Import data into Redis using pipe mode**
  
  From file to specific database
  
  ```
  redis-cli -h hostname -p port -n database-number --pipe < filename
  ```
  
  Example:
  
  ```
  redis-cli -h localhost -p 6379 -n 0 --pipe < backup.txt
  ```

---

## 🔄 Transactions

- **Mark the start of a transaction block**
  
  ```
  multi
  ```
  
  All subsequent commands will be queued until EXEC is called.

- **Execute all commands issued after MULTI**
  
  ```
  exec
  ```
  
  Example transaction:
  
  ```
  multi
  set key1 "value1"
  set key2 "value2"
  incr counter
  exec
  ```

- **Discard all commands issued after MULTI**
  
  ```
  discard
  ```
  
  Cancels the transaction without executing any commands.

---

## 🔍 Monitoring & Debugging

- **Listen for all requests received by the server**
  
  In real time
  
  ```
  monitor
  ```
  
  Shows all commands being executed. Press Ctrl+C to stop monitoring.

- **Get information and statistics about the server**
  
  ```
  info
  ```
  
  Displays comprehensive server information including:
  - Server version
  - Memory usage
  - Connected clients
  - Persistence status
  - Replication info
  
  Filter by section:
  
  ```
  info server
  info memory
  info stats
  ```

- **Get the value of a configuration parameter**
  
  From redis.conf
  
  ```
  config get <key>
  ```
  
  Example:
  
  ```
  config get maxmemory
  config get databases
  config get *
  ```

---

## 👥 Client Management

- **Get the list of client connections**
  
  ```
  client list
  ```
  
  Shows all connected clients with details:
  - Client ID
  - IP address
  - Port
  - Connection age
  - Idle time

- **Get the client ID for the current connection**
  
  ```
  client id
  ```

- **Kill the connection of a client**
  
  ```
  client kill ip:port
  ```
  
  Example:
  
  ```
  client kill 192.168.1.100:52341
  ```

---

## 🔐 Security & Authentication

- **Bind Redis server to a specific hostname**
  
  Set in redis.conf
  
  ```
  bind hostname
  ```
  
  Example:
  
  ```
  bind 127.0.0.1
  bind 0.0.0.0
  bind 192.168.1.100
  ```

- **Enable default user with only connection permissions**
  
  Set in redis.conf
  
  ```
  user default on +@connection
  ```

- **Create/enable a user with full permissions**
  
  With password authentication (set in redis.conf)
  
  ```
  user username on +@all ~* >password
  ```
  
  Example:
  
  ```
  user admin on +@all ~* >StrongPassword123
  user dev on +@read ~* >DevPass456
  ```

- **Authenticate with Redis**
  
  Using username and password
  
  ```
  auth username password
  ```
  
  Example:
  
  ```
  auth admin StrongPassword123
  ```

---

## 💾 Persistence

- **Synchronously save the dataset to disk**
  
  ```
  save
  ```
  
  ⚠️ **Warning:** Blocks the server until save is complete. Use BGSAVE for production.

- **Asynchronously save the dataset to disk**
  
  ```
  bgsave
  ```
  
  Saves in the background without blocking the server.

- **Configure automatic snapshotting**
  
  Set in redis.conf
  
  ```
  save <seconds> <total-changes>
  ```
  
  Takes snapshot if at least `<total-changes>` keys changed within `<seconds>` seconds.
  
  Example configurations:
  
  ```
  save 900 1      # After 900 seconds if at least 1 key changed
  save 300 10     # After 300 seconds if at least 10 keys changed
  save 60 10000   # After 60 seconds if at least 10000 keys changed
  ```

---

## 🧠 Memory Management

- **Limit the maximum memory Redis can use**
  
  Set in redis.conf
  
  ```
  maxmemory <memory-size>
  ```
  
  Example:
  
  ```
  maxmemory 256mb
  maxmemory 2gb
  maxmemory 100mb
  ```

- **Define the eviction policy when maxmemory is reached**
  
  Set in redis.conf
  
  ```
  maxmemory-policy <policy>
  ```
  
  **Available policies:**
  
  - **noeviction** - Return error when memory limit is reached (default)
    
    Best for: Critical data that must not be lost
  
  - **allkeys-lru** - Evict least recently used keys among all keys
    
    Best for: General purpose caching
  
  - **volatile-lru** - Evict least recently used keys with expiration set
    
    Best for: Mixed workloads with both cache and persistent data
  
  - **allkeys-lfu** - Evict least frequently used keys among all keys
    
    Best for: Caching with frequency-based access patterns
  
  - **volatile-lfu** - Evict least frequently used keys with expiration set
    
    Best for: Mixed workloads with frequency-based eviction
  
  - **volatile-random** - Evict random keys with expiration set
    
    Best for: When access patterns are unpredictable
  
  - **allkeys-random** - Evict random keys among all keys
    
    Best for: When all keys have equal importance
  
  - **volatile-ttl** - Evict keys with the nearest expiration time (shortest TTL)
    
    Best for: Time-sensitive cache data
  
  Example:
  
  ```
  maxmemory-policy allkeys-lru
  maxmemory-policy volatile-ttl
  ```

---

## 💡 Tips & Best Practices

- **Key Naming Conventions**
  - Use descriptive names: `user:1001:email`, `session:abc123`
  - Use colons for hierarchy: `object:id:field`
  - Keep keys short but readable
  - Use consistent naming patterns

- **Performance Optimization**
  - Use pipelining for multiple commands
  - Avoid KEYS command in production (use SCAN instead)
  - Set appropriate expiration times
  - Monitor memory usage regularly
  - Use appropriate data structures

- **Security Best Practices**
  - Always set a password
  - Bind to specific interfaces
  - Use Redis ACL for user permissions
  - Keep Redis updated
  - Don't expose Redis to the internet directly

- **Data Persistence**
  - Use RDB for point-in-time snapshots
  - Use AOF for better durability
  - Combine both for maximum safety
  - Regular backups are essential
  - Test your backup restoration process

- **Memory Management**
  - Set maxmemory limits
  - Choose appropriate eviction policy
  - Monitor memory usage
  - Use expiration for temporary data
  - Clean up unused keys regularly

---

## 🆘 Troubleshooting

- **Redis won't start**
  - Check if port is already in use
  - Verify configuration file syntax
  - Check system resources (RAM)
  - Review error logs

- **High memory usage**
  - Check key count: `dbsize`
  - Identify large keys: `memory usage key`
  - Review expiration settings
  - Consider eviction policy

- **Slow performance**
  - Use SLOWLOG to identify slow queries
  - Avoid blocking commands (use SCAN instead of KEYS)
  - Check network latency
  - Monitor CPU usage

- **Connection issues**
  - Verify bind configuration
  - Check firewall rules
  - Test authentication
  - Review client timeout settings

---

**Note:** Commands marked with "set in redis.conf" should be configured in the Redis configuration file before starting the server.