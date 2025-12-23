# Redis CLI - Hands-on Practice

A practical guide to Redis CLI with basic operations, data manipulation, and monitoring.

## 🛠️ Prerequisites: Connect to Redis CLI

Open your terminal and connect to your Redis server:

```bash
redis-cli
```

## 🏃 Exercise 1: Basic Operations & String Manipulation

**Scenario:** You want to store a user's simple profile.

- **Store Username**

  ```bash
  SET user:1 "Budi Santoso"
  ```

- **Add Title (Append)**

  ```bash
  APPEND user:1 ", S.Kom"
  ```

- **Get Substring**
  
  Extract only the word "Budi":
  
  ```bash
  GETRANGE user:1 0 3
  ```

- **Replace Last Name (SetRange)**
  
  Change "Budi" to "Andi":
  
  ```bash
  SETRANGE user:1 0 "Andi"
  ```

## ⏱️ Exercise 2: Expiration & Automation

**Scenario:** Create an OTP (One-Time Password) code that expires in 30 seconds.

- **Store OTP with Expiry**

  ```bash
  SETEX otp:user:1 30 "123456"
  ```

- **Check Remaining Time**

  ```bash
  TTL otp:user:1
  ```
  
  (Repeat this command multiple times to see the countdown).

- **Check Status After Expiration**
  
  After 30 seconds, run:
  
  ```bash
  EXISTS otp:user:1
  ```

## 📈 Exercise 3: Counters & Statistics (Incr/Decr)

**Scenario:** Count website visitors and product stock.

- **Increment Visitors (One by One)**

  ```bash
  INCR web:views
  INCR web:views
  ```

- **Add Points (In Bulk)**

  ```bash
  INCRBY user:1:points 50
  ```

- **Decrease Product Stock**

  ```bash
  SET produk:laptop:stok 10
  DECR produk:laptop:stok
  DECRBY produk:laptop:stok 3
  ```

## 📦 Exercise 4: Batch Operations & Pattern Matching

**Scenario:** Insert multiple data at once and search for them.

- **Bulk Input**

  ```bash
  MSET session:a "active" session:b "idle" config:theme "dark"
  ```

- **Search Keys by Pattern**
  
  Find all keys starting with "session":
  
  ```bash
  KEYS session:*
  ```

- **Get Specific Data**

  ```bash
  MGET session:a config:theme
  ```

## 🔄 Exercise 5: Transactions (Atomic Operations)

**Scenario:** Transfer points between users safely (no interruption in the middle).

- **Start Transaction**

  ```bash
  MULTI
  ```

- **Queue Commands**

  ```bash
  DECRBY user:1:points 20
  INCRBY user:2:points 20
  ```

- **Execute**

  ```bash
  EXEC
  ```
  
  (If you want to cancel before EXEC, type `DISCARD`).

## 🔍 Exercise 6: Monitoring & Debugging

**Scenario:** See what happens "behind the scenes" in Redis.

- **Open a New Terminal Tab, then run:**

  ```bash
  redis-cli MONITOR
  ```

- **Return to the First Terminal**
  
  Type any command (example: `SET test 1`).

- **Check the Monitor Terminal**
  
  You will see the command log in real-time.

## 🧼 Exercise 7: Cleanup

⚠️ **Warning:** Use this only in practice environments!

- **Delete Current Database**

  ```bash
  FLUSHDB
  ```

- **Verify**

  ```bash
  DBSIZE
  ```
  
  (Should return 0).
