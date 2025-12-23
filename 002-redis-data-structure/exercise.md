# Redis CLI - Hands-on Practice

A practical guide to Redis data structures with Lists, Sets, Hashes, Sorted Sets, Geospatial, and Streams.

## 🛠️ Prerequisites

Ensure Redis server is running and connect to CLI:

```bash
redis-cli
```

## 🏃 Exercise 1: List (Order Queue Management)

**Scenario:** You are managing an order queue at a restaurant.

- **Add 3 New Orders to Queue (LPUSH)**

  ```bash
  LPUSH queue:orders "Order-001" "Order-002" "Order-003"
  ```

- **View All Current Orders**

  ```bash
  LRANGE queue:orders 0 -1
  ```

- **Get Oldest Order to Process (RPOP - FIFO)**

  ```bash
  RPOP queue:orders
  ```
  
  (Should return "Order-001")

## 🔷 Exercise 2: Set (Friend Recommendation System)

**Scenario:** Finding common hobbies between two users.

- **Add Alice's and Bob's Hobbies**

  ```bash
  SADD hobbies:alice "coding" "gaming" "hiking"
  SADD hobbies:bob "gaming" "cooking" "traveling"
  ```

- **Find Common Hobbies (Intersection)**

  ```bash
  SINTER hobbies:alice hobbies:bob
  ```

- **Find Hobbies Alice Has but Bob Doesn't (Difference)**

  ```bash
  SDIFF hobbies:alice hobbies:bob
  ```

- **Find All Unique Hobbies (Union)**

  ```bash
  SUNION hobbies:alice hobbies:bob
  ```

## 🧩 Exercise 3: Hash (User Profile)

**Scenario:** Storing user data in a single object.

- **Create User Profile**

  ```bash
  HSET user:id:99 name "Budi" email "budi@mail.com" login_count 0
  ```

- **Simulate Login (Increment)**

  ```bash
  HINCRBY user:id:99 login_count 1
  ```

- **Get All Profile Data**

  ```bash
  HGETALL user:id:99
  ```

## 🏅 Exercise 4: Sorted Set (Game Leaderboard)

**Scenario:** Displaying top player scores.

- **Add Player Scores**

  ```bash
  ZADD game:score 1200 "Slayer" 2500 "Master" 1800 "Noob"
  ```

- **Update "Slayer" Score (Add 500 Points)**

  ```bash
  ZINCRBY game:score 500 "Slayer"
  ```

- **Display Top 3 Players (Highest Score First)**

  ```bash
  ZREVRANGE game:score 0 2 WITHSCORES
  ```

## 📍 Exercise 5: Geospatial (Nearest Store Feature)

**Scenario:** Finding store branches in Jakarta.

- **Add Coordinates (Bundaran HI and Monas)**

  ```bash
  GEOADD jakarta:branch 106.8230 -6.1950 "BunderanHI"
  GEOADD jakarta:branch 106.8272 -6.1751 "Monas"
  ```

- **Calculate Distance Between Them**

  ```bash
  GEODIST jakarta:branch "BunderanHI" "Monas" km
  ```

- **Find Locations Within 2km Radius from Specific Coordinates**

  ```bash
  GEOSEARCH jakarta:branch FROMLONLAT 106.8230 -6.1950 BYRADIUS 2 km
  ```

## 🌊 Exercise 6: Streams (System Activity Log)

**Scenario:** Recording login activity in append-only mode.

- **Record New Activities**

  ```bash
  XADD log:activity * user_id "101" action "login"
  XADD log:activity * user_id "102" action "upload_photo"
  ```

- **Read Last 5 Messages from the Beginning**

  ```bash
  XREAD COUNT 5 STREAMS log:activity 0
  ```

## 🧼 Exercise 7: Cleanup

After completing the exercises, clean up your database:

⚠️ **Warning:** Use this only in practice environments!

```bash
FLUSHDB
```
