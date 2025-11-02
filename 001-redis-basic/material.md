# Redis Command Cheatsheet

A comprehensive guide to Redis commands and configurations.

## Server Management

### Start Redis Server (default configuration)
```
redis-server
```

### Connect to Redis CLI
```
redis-cli -h hostname -p port
```

### Start Redis Server with a custom configuration file
```
redis-server path/redis.conf
```

## Database Configuration

### Set number of Redis databases (default: 16 in redis.conf)
```
databases 16
```

### Select a database index
```
select index
```

## Basic Key-Value Operations

### Store key-value pair (overwrites if key already exists)
```
set key value
```

### Get the value of a key
```
get key
```

### Check if a key exists (1 = exists, 0 = does not exist)
```
exists key
```

### Delete one or more keys (1 = deleted, 0 = not found)
```
del key [key ...]
```

### Append a value to an existing key (creates key if it does not exist)
```
append key value
```

### List keys matching a given pattern (e.g., keys *)
```
keys pattern
```

## String Manipulation

### Overwrite part of a string starting at the given offset
```
setrange key offset value
```

### Get a substring of the value (by start and end index)
```
getrange key start end
```

## Multiple Operations

### Set multiple key-value pairs
```
mset key value [key value ...]
```

### Get values of multiple keys
```
mget key [key ...]
```

## Key Expiration

### Set a key's expiration time (in seconds)
```
expire key seconds
```

### Set the value and expiration time (in seconds) for a key
```
setex key seconds value
```

### Get the remaining time-to-live (TTL) of a key (in seconds)
```
ttl key
```

## Increment/Decrement Operations

### Increment the integer value of a key by one
```
incr key
```

### Decrement the integer value of a key by one
```
decr key
```

### Increment the integer value of a key by the given amount
```
incrby key increment
```

### Decrement the integer value of a key by the given amount
```
decrby key increment
```

## Database Management

### Remove all keys from the current database
```
flushdb
```

### Remove all keys from all databases
```
flushall
```

### Import data into redis using pipe mode (from file to specific database)
```
redis-cli -h hostname -p port -n database-number --pipe < filename
```

## Transactions

### Mark the start of a transaction block
```
multi
```

### Executes all commands issued after MULTI
```
exec
```

### Discard all commands issued after MULTI
```
discard
```

## Monitoring & Debugging

### Listen for all requests received by the server in real time
```
monitor
```

### Get information and statistics about the server
```
info
```

### Get the value of a configuration parameter from redis.conf
```
config get <key>
```

## Client Management

### Get the list of client connections
```
client list
```

### Returns the client ID for the current connection
```
client id
```

### Kill the connection of a client
```
client kill ip:port
```

## Security & Authentication

### Bind Redis server to a specific hostname (set in redis.conf)
```
bind hostname
```

### Enable default user with only connection permissions (set in redis.conf)
```
user default on +@connection
```

### Create/enable a user with full permissions and password authentication (set in redis.conf)
```
user username on +@all ~* >password
```

### Authenticate with Redis using username and password
```
auth username password
```

## Persistence

### Synchronously save the dataset to disk
```
save
```

### Asynchronously save the dataset to disk
```
bgsave
```

### Configure automatic snapshotting (set in redis.conf)
```
save <seconds> <total-changes>
```
**Note:** Takes snapshot if at least `<total-changes>` keys changed within `<seconds>` seconds.

## Memory Management

### Limit the maximum memory Redis can use (set in redis.conf)
```
maxmemory <memory-size>
```

### Define the eviction policy when maxmemory is reached (set in redis.conf)
```
maxmemory-policy <policy>
```

**Available policies:**
- `noeviction` - Return error when memory limit is reached (default)
- `allkeys-lru` - Evict least recently used keys among all keys
- `volatile-lru` - Evict least recently used keys with expiration set
- `allkeys-lfu` - Evict least frequently used keys among all keys
- `volatile-lfu` - Evict least frequently used keys with expiration set
- `volatile-random` - Evict random keys with expiration set
- `allkeys-random` - Evict random keys among all keys
- `volatile-ttl` - Evict keys with the nearest expiration time (shortest TTL)

---

**Note:** Commands marked with "set in redis.conf" should be configured in the Redis configuration file.