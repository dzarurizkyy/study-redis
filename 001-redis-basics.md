# 🔴 Redis Basics

A practical reference guide for learning Redis from scratch — covering the server and CLI, databases and key–value operations, string manipulation, expiration, counters, transactions, monitoring, client and user management, persistence, and memory management, worked through hands-on in `redis-cli`.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is Redis?](#what-is-redis)
  - [The Data Model](#the-data-model)
- [Server Management](#-server-management)
  - [Starting the Server](#starting-the-server)
  - [Connecting with the CLI](#connecting-with-the-cli)
  - [Using a Custom Configuration](#using-a-custom-configuration)
- [Database Configuration](#-database-configuration)
  - [Number of Databases](#number-of-databases)
  - [Selecting a Database](#selecting-a-database)
- [Basic Key-Value Operations](#-basic-key-value-operations)
  - [Core Commands](#core-commands)
  - [Storing & Reading Values](#storing--reading-values)
  - [Checking & Deleting Keys](#checking--deleting-keys)
  - [Appending to a Value](#appending-to-a-value)
  - [Listing Keys by Pattern](#listing-keys-by-pattern)
- [String Manipulation](#-string-manipulation)
  - [Overwriting Part of a String](#overwriting-part-of-a-string)
  - [Reading a Substring](#reading-a-substring)
- [Multiple Operations](#-multiple-operations)
- [Key Expiration](#-key-expiration)
  - [Expiration Commands](#expiration-commands)
  - [Setting an Expiration](#setting-an-expiration)
  - [Reading the Remaining TTL](#reading-the-remaining-ttl)
- [Increment & Decrement](#-increment--decrement)
  - [Counter Commands](#counter-commands)
  - [Counting by One](#counting-by-one)
  - [Counting by an Amount](#counting-by-an-amount)
- [Database Management](#-database-management)
  - [Flushing Data](#flushing-data)
  - [Mass Insertion with Pipe Mode](#mass-insertion-with-pipe-mode)
- [Transactions](#-transactions)
  - [Transaction Commands](#transaction-commands)
  - [Running a Transaction](#running-a-transaction)
  - [Discarding a Transaction](#discarding-a-transaction)
- [Monitoring & Debugging](#-monitoring--debugging)
  - [Watching Live Traffic](#watching-live-traffic)
  - [Server Information](#server-information)
  - [Reading the Configuration](#reading-the-configuration)
- [Client Management](#-client-management)
  - [Client Commands](#client-commands)
  - [Inspecting Connections](#inspecting-connections)
  - [Killing a Connection](#killing-a-connection)
- [Security & Authentication](#-security--authentication)
  - [Binding the Server](#binding-the-server)
  - [Users & ACL](#users--acl)
  - [Authenticating](#authenticating)
- [Persistence](#-persistence)
  - [Saving on Demand](#saving-on-demand)
  - [Automatic Snapshotting](#automatic-snapshotting)
  - [RDB vs AOF](#rdb-vs-aof)
- [Memory Management](#-memory-management)
  - [Limiting Memory](#limiting-memory)
  - [Eviction Policies](#eviction-policies)
- [Troubleshooting](#-troubleshooting)
- [Command Execution Flow](#-command-execution-flow)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is Redis?

- **Redis** (REmote DIctionary Server) is a free and open-source, **in-memory** data store
- Every read and write happens in RAM, which is why a single command typically completes in microseconds
- It is used as a **cache**, a **session store**, a **message broker**, and a **real-time counter** — often all at once
- Commands are executed by a **single thread**, so operations are atomic by nature and nothing interleaves
- Data survives a restart only through [persistence](#-persistence) — RDB snapshots, AOF, or both

> Reference: [redis.io](https://redis.io/) · [redis.io/docs](https://redis.io/docs/latest/)

### The Data Model

There are no tables, no rows, and no schema — only a flat keyspace:

| Concept | In Redis |
| --- | --- |
| Database | A numbered index (`0`–`15` by default), selected with `select` |
| Key | Always a **string**, e.g. `user:1001:email` |
| Value | Typed — string, list, set, sorted set, hash, stream, … |
| Namespace | A **naming convention**, `object:id:field`, not a real container |
| Expiry | A per-key TTL, checked by the server and removed automatically |

> **Key Insight:** Redis has no folders and no schema — `user:1001:email` is one flat key that merely *looks* nested. The colon convention is what turns a flat keyspace into something a human can navigate, which is why naming discipline matters more here than in a relational database.

---

## 🖥️ Server Management

### Starting the Server

Running `redis-server` with no arguments starts it on the default port **`6379`** with the built-in configuration:

```bash
redis-server
```

### Connecting with the CLI

```bash
redis-cli -h hostname -p port
```

Omit both flags and `redis-cli` connects to `127.0.0.1:6379`.

### Using a Custom Configuration

```bash
redis-server path/redis.conf
```

> **Note:** most of the settings in this guide — `databases`, `bind`, `user`, `save`, `maxmemory` — live in `redis.conf` and are read **at startup**. Editing the file after the server is running changes nothing until it restarts.

---

## 🗄️ Database Configuration

### Number of Databases

Redis ships with **16** logical databases, numbered `0` to `15`. The count is set in `redis.conf`:

```bash
databases 16
```

### Selecting a Database

```bash
select index
```

```bash
select 0
select 1
```

> **Key Insight:** these are numbered namespaces sharing one server, one thread, and one memory limit — not separate databases in the SQL sense. `flushall`, `maxmemory`, and every performance characteristic apply across all of them at once.

---

## 🔑 Basic Key-Value Operations

### Core Commands

| Command | Description |
| --- | --- |
| `set key value` | Store a key-value pair (overwrites if the key exists) |
| `get key` | Get the value of a key |
| `exists key` | Check if a key exists — returns `1` if it does, `0` if not |
| `del key [key ...]` | Delete one or more keys — returns the number deleted |
| `append key value` | Append to an existing value (creates the key if missing) |
| `keys pattern` | List every key matching a pattern |

### Storing & Reading Values

```bash
set username "john_doe"
set counter 100

get username
```

> **Gotcha:** a plain `set` on an existing key **clears its TTL**. A key with 3600 seconds left becomes permanent the moment it is overwritten — add the `KEEPTTL` option if the expiry should survive.

### Checking & Deleting Keys

```bash
exists username

del username
del key1 key2 key3
```

### Appending to a Value

```bash
append message "Hello"
append message " World"
```

### Listing Keys by Pattern

```bash
keys *
keys user:*
keys *name*
```

> ⚠️ **Warning:** `keys` scans the **entire keyspace** and blocks the single command thread until it finishes. On a production database with millions of keys, every other client waits. Use `scan` — a cursor-based iteration — instead.

---

## ✂️ String Manipulation

### Overwriting Part of a String

Writes `value` into the string starting at `offset`:

```bash
setrange key offset value
```

```bash
set greeting "Hello World"
setrange greeting 6 "Redis"
```

### Reading a Substring

Returns the characters between `start` and `end`, both inclusive:

```bash
getrange key start end
```

```bash
getrange greeting 0 4
getrange greeting -5 -1
```

> **Tip:** negative indexes count from the end — `-1` is the last character. `getrange key 0 -1` is therefore the whole string, which is handy when you want a substring expression that also covers the full value.

---

## 📦 Multiple Operations

Setting and reading many keys in one command means **one** network round trip instead of N.

| Command | Description |
| --- | --- |
| `mset key value [key value ...]` | Set multiple key-value pairs |
| `mget key [key ...]` | Get the values of multiple keys |

```bash
mset username "john" email "john@example.com" age 25

mget username email age
```

> **Note:** `mget` always returns one entry per key in the order requested, with `nil` for keys that don't exist — so the reply lines up positionally with your input even when some keys are missing.

---

## ⏰ Key Expiration

### Expiration Commands

| Command | Description |
| --- | --- |
| `expire key seconds` | Set a key's time-to-live, in seconds |
| `setex key seconds value` | Set the value and its expiration in one command |
| `ttl key` | Get the remaining time-to-live, in seconds |

### Setting an Expiration

```bash
# Add a TTL to an existing key
expire session:abc123 3600

# Set value and TTL together
setex token:xyz 7200 "secret_token_value"
```

> **Key Insight:** `setex` is atomic where `set` followed by `expire` is two commands — and if the connection drops between them, you are left with a key that never expires. For anything session- or token-shaped, always set the value and the TTL in a single command.

### Reading the Remaining TTL

```bash
ttl session:abc123
```

| Return value | Meaning |
| --- | --- |
| A positive number | Seconds until expiration |
| `-1` | The key exists but has no expiration |
| `-2` | The key does not exist |

---

## ➕ Increment & Decrement

### Counter Commands

| Command | Description |
| --- | --- |
| `incr key` | Increment the integer value by one |
| `decr key` | Decrement the integer value by one |
| `incrby key increment` | Increment by the given amount |
| `decrby key increment` | Decrement by the given amount |

### Counting by One

```bash
incr page_views
incr visitor_count

decr stock_count
```

### Counting by an Amount

```bash
incrby score 10
incrby balance 500

decrby stock 5
decrby credits 100
```

> **Key Insight:** these commands run **on the server**, so they are atomic. Ten clients calling `incr page_views` at the same instant produce ten increments — reading the value, adding one in application code, and writing it back would lose most of them.

> **Note:** a missing key is treated as `0`, so `incr` on a brand-new key returns `1`. A key holding a non-numeric string returns an error instead.

---

## 🗑️ Database Management

### Flushing Data

| Command | Description |
| --- | --- |
| `flushdb` | Remove all keys from the **current** database |
| `flushall` | Remove all keys from **every** database |

```bash
flushdb

flushall
```

> ⚠️ **Warning:** both are irreversible and take no confirmation. `flushall` ignores which database you selected and empties all 16 at once.

### Mass Insertion with Pipe Mode

Pipe mode streams a file of commands straight into the server — the fastest way to load bulk data.

```bash
redis-cli -h hostname -p port -n database-number --pipe < filename
```

```bash
redis-cli -h localhost -p 6379 -n 0 --pipe < backup.txt
```

> **Note:** `-n` selects the target database index, so the same dump file can be loaded into a scratch database first and verified before it touches database `0`.

---

## 🔄 Transactions

### Transaction Commands

| Command | Description |
| --- | --- |
| `multi` | Mark the start of a transaction block |
| `exec` | Execute every command queued since `multi` |
| `discard` | Throw away the queued commands without executing them |

### Running a Transaction

After `multi`, commands are **queued** rather than executed. `exec` runs the whole block:

```bash
multi
set key1 "value1"
set key2 "value2"
incr counter
exec
```

> **Gotcha:** a Redis transaction is not a SQL transaction — there is **no rollback**. `exec` guarantees the block runs as one uninterrupted unit, not that every command inside it succeeds. If `incr` fails on a non-numeric key, the two `set` commands still take effect.

### Discarding a Transaction

```bash
discard
```

Cancels the transaction and clears the queue without executing anything.

---

## 🔍 Monitoring & Debugging

### Watching Live Traffic

`monitor` streams every command the server receives, in real time. Press `Ctrl + C` to stop.

```bash
monitor
```

> ⚠️ **Warning:** `monitor` makes the server echo every single command to your connection, which measurably slows down production traffic. Use it to debug, never leave it running.

### Server Information

```bash
info
```

The report covers, among much else:

- Server version
- Memory usage
- Connected clients
- Persistence status
- Replication info

Ask for one section instead of the whole report:

```bash
info server
info memory
info stats
```

### Reading the Configuration

Reads the effective value of a `redis.conf` parameter:

```bash
config get <key>
```

```bash
config get maxmemory
config get databases
config get *
```

> **Tip:** `config set` changes most parameters on a **running** server, which is how you tune `maxmemory` without a restart. The change lives in memory only — `config rewrite` writes it back into `redis.conf` so it survives the next start.

---

## 👥 Client Management

### Client Commands

| Command | Description |
| --- | --- |
| `client list` | List every client connection |
| `client id` | Get the ID of the current connection |
| `client kill ip:port` | Close another client's connection |

### Inspecting Connections

```bash
client list
```

Each row describes one connection:

- Client ID
- IP address
- Port
- Connection age
- Idle time

```bash
client id
```

### Killing a Connection

```bash
client kill 192.168.1.100:52341
```

> **Tip:** `client list` is the fastest way to find the culprit behind a stuck server — sort by idle time and look for the connection that has been running `monitor` or a blocking command for hours.

---

## 🔐 Security & Authentication

### Binding the Server

`bind` decides which network interfaces Redis listens on. Set it in `redis.conf`:

```bash
bind hostname
```

```bash
bind 127.0.0.1      # localhost only — the safe default
bind 0.0.0.0        # every interface — reachable from anywhere
bind 192.168.1.100  # one specific interface
```

> ⚠️ **Warning:** `bind 0.0.0.0` on a public server exposes Redis to the internet. Combined with no password, that is a database anyone can read, write, and flush.

### Users & ACL

Redis 6 introduced **ACL** — named users, each with an explicit set of allowed command categories and key patterns. Users are declared in `redis.conf`:

```bash
# Default user, allowed only to connect
user default on +@connection

# A named user with a password and full access
user username on +@all ~* >password
```

```bash
user admin on +@all ~* >StrongPassword123
user dev on +@read ~* >DevPass456
```

The rule is read left to right:

| Token | Meaning |
| --- | --- |
| `on` | The user is enabled |
| `+@all` | Allow every command category (`+@read` allows read commands only) |
| `~*` | Allow access to every key pattern |
| `>password` | Set this user's password |

> **Key Insight:** locking `default` down to `+@connection` is the important half. Creating an admin user means nothing while the default user — the one every unauthenticated client lands on — can still run every command.

### Authenticating

```bash
auth username password
```

```bash
auth admin StrongPassword123
```

---

## 💾 Persistence

### Saving on Demand

| Command | Description |
| --- | --- |
| `save` | Write the dataset to disk **synchronously** |
| `bgsave` | Write the dataset to disk **in the background** |

```bash
save

bgsave
```

> ⚠️ **Warning:** `save` blocks the server until the snapshot is finished — with a large dataset, every client stalls. Use `bgsave` in production: it forks a child process and the main thread keeps serving commands.

### Automatic Snapshotting

Configure snapshots in `redis.conf` — take one if at least `<total-changes>` keys changed within `<seconds>` seconds:

```bash
save <seconds> <total-changes>
```

```bash
save 900 1      # After 900 seconds if at least 1 key changed
save 300 10     # After 300 seconds if at least 10 keys changed
save 60 10000   # After 60 seconds if at least 10000 keys changed
```

> **Note:** the three lines are cumulative, not alternatives — whichever condition is met first triggers the snapshot. Frequent writes hit the `60 10000` rule; a nearly idle server still gets a snapshot every 15 minutes.

### RDB vs AOF

| | **RDB** (snapshot) | **AOF** (append-only file) |
| --- | --- | --- |
| What it stores | The dataset at a point in time | Every write command, in order |
| Restart speed | Fast — one file is loaded | Slower — the log is replayed |
| Worst-case data loss | Everything since the last snapshot | Typically the last second of writes |
| Best for | Backups and fast restarts | Durability |

> **Key Insight:** RDB and AOF are not a choice between two options — enabling both is the usual production answer. AOF keeps the loss window down to a second, RDB gives you a compact file to actually copy off the machine.

---

## 🧠 Memory Management

### Limiting Memory

Cap the memory Redis may use, in `redis.conf`:

```bash
maxmemory <memory-size>
```

```bash
maxmemory 256mb
maxmemory 2gb
maxmemory 100mb
```

### Eviction Policies

When the limit is reached, `maxmemory-policy` decides what happens next:

```bash
maxmemory-policy <policy>
```

| Policy | Behaviour | Best for |
| --- | --- | --- |
| `noeviction` | Return an error instead of evicting (default) | Critical data that must not be lost |
| `allkeys-lru` | Evict the least recently used key, from all keys | General purpose caching |
| `volatile-lru` | Evict the least recently used key **with a TTL** | Mixed cache + persistent data |
| `allkeys-lfu` | Evict the least frequently used key, from all keys | Frequency-based access patterns |
| `volatile-lfu` | Evict the least frequently used key **with a TTL** | Mixed workloads, frequency-based |
| `volatile-random` | Evict a random key **with a TTL** | Unpredictable access patterns |
| `allkeys-random` | Evict a random key, from all keys | All keys equally important |
| `volatile-ttl` | Evict the key with the nearest expiration | Time-sensitive cache data |

```bash
maxmemory-policy allkeys-lru
maxmemory-policy volatile-ttl
```

> **Gotcha:** every `volatile-*` policy only considers keys that **have a TTL**. Set one of them on a database where nothing expires and Redis finds nothing to evict — it starts returning out-of-memory errors exactly like `noeviction`.

---

## 🆘 Troubleshooting

**Redis won't start**

- Check whether the port is already in use
- Verify the configuration file syntax
- Check system resources, RAM in particular
- Review the error log

**High memory usage**

- Count the keys with `dbsize`
- Find the large ones with `memory usage key`
- Review the expiration settings — are TTLs actually being set?
- Reconsider the eviction policy

**Slow performance**

- Use `slowlog` to identify slow commands
- Avoid blocking commands — `scan` instead of `keys`, `bgsave` instead of `save`
- Check network latency
- Monitor CPU usage

**Connection issues**

- Verify the `bind` configuration
- Check the firewall rules
- Test authentication with `auth`
- Review the client timeout settings

---

## 🔁 Command Execution Flow

Knowing the order in which Redis handles a command is what makes the rest of this guide click. For every command sent:

```
Client (redis-cli / driver)
  ↓
Connection & AUTH             username + password → the user's ACL rules
  ↓
Command Queue                 one thread — one command at a time, in arrival order
  ↓
MULTI … EXEC                  ← queued here, then run as one uninterrupted block
  ↓
Execution (in memory)         the data is already in RAM; there is no disk read
  ↓
Keyspace Maintenance          TTL expiration, and eviction once maxmemory is hit
  ↓
Persistence                   AOF append and/or RDB snapshot — after the reply
  ↓
Reply → Client
```

This explains the design decisions throughout this guide:

| Question | Answer |
| --- | --- |
| Why does one slow command freeze everything? | Commands share a single thread — `keys`, `save` and `monitor` make every other client wait their turn |
| Why is `incr` safe without a lock? | Nothing interleaves on a single thread, so read-modify-write happens entirely inside the command |
| Why does a transaction not roll back? | `exec` guarantees the block runs uninterrupted, not that each command succeeds — there is no undo step in the flow |
| Why can data be lost even with persistence on? | Persistence happens *after* the reply — RDB on an interval, AOF on an fsync policy |
| Why did a key disappear before its TTL? | Eviction runs before the key expires when `maxmemory` is reached |
| Why does `bgsave` not block but `save` does? | `bgsave` forks a child process; `save` writes on the same thread that serves commands |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Server** | Start Redis, default port `6379` | `redis-server [path/redis.conf]` |
| **CLI** | Connect a client | `redis-cli -h hostname -p port` |
| **Database** | Pick one of the 16 numbered databases | `select 0` |
| **Set / Get** | Store and read a value | `set key value`, `get key` |
| **Exists / Delete** | Test for and remove keys | `exists key`, `del key [key ...]` |
| **Append** | Add to an existing string | `append message " World"` |
| **Pattern search** | Find keys by pattern | `keys user:*` (prefer `scan` in production) |
| **Substring** | Read or overwrite part of a string | `getrange key 0 4`, `setrange key 6 "Redis"` |
| **Multi-key** | One round trip for many keys | `mset k v k v`, `mget k k` |
| **Expiration** | Give a key a lifetime | `expire key 3600`, `setex key 7200 value` |
| **TTL** | Read the remaining lifetime | `ttl key` → seconds, `-1`, or `-2` |
| **Counter** | Atomic arithmetic on the server | `incr`, `decr`, `incrby`, `decrby` |
| **Flush** | Empty a database | `flushdb` (current), `flushall` (every) |
| **Mass insert** | Load a file of commands | `redis-cli -n 0 --pipe < backup.txt` |
| **Transaction** | Run commands as one block | `multi` … `exec` / `discard` |
| **Monitoring** | Inspect traffic and state | `monitor`, `info memory`, `slowlog` |
| **Configuration** | Read and tune parameters | `config get maxmemory`, `config set`, `config rewrite` |
| **Clients** | Inspect and close connections | `client list`, `client id`, `client kill ip:port` |
| **Network** | Choose the listening interface | `bind 127.0.0.1` |
| **ACL** | Define users and permissions | `user admin on +@all ~* >password` |
| **Auth** | Log in as a user | `auth username password` |
| **Persistence** | Write the dataset to disk | `bgsave`, `save 900 1`, AOF |
| **Memory** | Cap usage and choose eviction | `maxmemory 256mb`, `maxmemory-policy allkeys-lru` |

---

## 💡 Best Practices

**✅ Do This**

- **Name keys hierarchically** — `user:1001:email`, `session:abc123`, following a consistent `object:id:field` pattern that stays short but readable
- **Set an expiration on everything temporary** — sessions, tokens and cache entries should clean themselves up rather than wait for eviction
- **Use `setex` instead of `set` + `expire`** so the value and its TTL are written atomically
- **Let the server do the arithmetic** with `incr` / `incrby` rather than read-modify-write in application code
- **Batch with `mset`, `mget` and pipelining** — one round trip beats N round trips by far
- **Prefer `scan` over `keys`** for any iteration over a production keyspace
- **Prefer `bgsave` over `save`**, and combine RDB with AOF when durability matters
- **Set `maxmemory` and an eviction policy that matches the workload** — `allkeys-lru` for a pure cache, `volatile-*` only when keys actually carry TTLs
- **Lock down the default user** (`user default on +@connection`) and give each application its own ACL user and password
- **Bind to a specific interface** and keep Redis behind a firewall, never directly on the internet
- **Watch the numbers regularly** — `info memory`, `dbsize`, `slowlog` and `client list` tell you what is going wrong before users do
- **Test your restore, not just your backup** — an RDB file nobody has ever loaded is not a backup

**❌ Avoid This**

- **Running `keys *` on production** — it walks the entire keyspace on the one thread every client shares
- **Running `save` on a large dataset** — it blocks the server for the whole snapshot; `bgsave` forks instead
- **Leaving `monitor` running** — it echoes every command to your connection and slows the server measurably
- **Expecting a transaction to roll back** — `exec` runs the block uninterrupted, but a failing command does not undo the others
- **Overwriting a key with plain `set` and expecting its TTL to survive** — it is cleared unless you pass `KEEPTTL`
- **Choosing a `volatile-*` eviction policy when no key has a TTL** — Redis finds nothing to evict and starts erroring out
- **Running `flushall` when you meant `flushdb`** — it empties all 16 databases, irreversibly and without a prompt
- **Exposing Redis with `bind 0.0.0.0` and no password** — that is an open database, not a cache
- **Treating Redis as the source of truth** — it is in-memory first, and persistence is a safety net rather than a guarantee
- **Editing `redis.conf` on a running server and expecting it to apply** — it is read at startup; use `config set` for the live value

> Reference: [redis.io/docs](https://redis.io/docs/latest/) · [redis.io/commands](https://redis.io/docs/latest/commands/)
