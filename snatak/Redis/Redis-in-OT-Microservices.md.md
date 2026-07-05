# Redis in OT-Microservices

## Redis Service Verification

### Check Redis Status

```bash
sudo systemctl status redis-server
```

### Start Redis

```bash
sudo systemctl start redis-server
```

### Restart Redis

```bash
sudo systemctl restart redis-server
```

### Verify Redis Connectivity

```bash
redis-cli ping
```

Expected Output:

```text
PONG
```

---

# Step-by-Step Redis Verification Procedure

---

# Step 1 — Start Redis Server

```bash
sudo systemctl start redis-server
```

---

# Step 2 — Verify Redis Server

```bash
redis-cli ping
```

Expected Output:

```text
PONG
```

---

# Step 3 — Open Redis Monitoring Terminal

Open a NEW terminal window.

Run:

```bash
redis-cli MONITOR
```

This terminal will now display all live Redis operations.

Keep this terminal running.

---

# Step 4 — Open Another Terminal

Open another terminal window.

Use this terminal for API testing.

---

# Step 5 — Verify Employee API Redis Operations

Run:

```bash
curl http://localhost:8080/api/v1/employee/search?id=1
```

Now check MONITOR terminal.

Expected Output:

```text
"hget" "employee" "1"
"hset" "employee" "1" "{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}"
```

---

# Understanding HGET and HSET

## HGET

```redis
HGET employee 1
```

Purpose:

* Reads employee data from Redis HASH
* Checks whether cache already exists
* Happens before database query

Meaning:

| Parameter | Value             |
| --------- | ----------------- |
| employee  | Redis HASH key    |
| 1         | Employee ID field |

---

## HSET

```redis
HSET employee 1 "{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}"
```

Purpose:

* Stores employee data in Redis
* Happens after database query
* Executed during cache miss

Meaning:

| Parameter                                                                          | Value                |
| ---------------------------------------------------------------------------------- | -------------------- |
| employee                                                                           | Redis HASH key       |
| 1                                                                                  | Employee ID field    |
| {"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"} | Cached employee data |

---

# Employee API Redis Flow

```text
Client Request
      ↓
Employee API
      ↓
HGET employee 1
      ↓
Cache MISS
      ↓
ScyllaDB Query
      ↓
HSET employee 1
      ↓
Response Returned
```

---

# Cache HIT Flow

```text
Client Request
      ↓
Employee API
      ↓
HGET employee 1
      ↓
Cache HIT
      ↓
Response Returned Directly From Redis
```

No database query occurs during cache hit.

---

# Step 6 — Verify All Employee Data Cache

Run:

```bash
curl http://localhost:8080/api/v1/employee/search/all
```

Expected MONITOR Output:

```text
"hget" "employee" "all_data"
"hset" "employee" "all_data" "[{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}]"
```

---

# Step 7 — Verify Attendance API Redis Operations

Run:

```bash
curl http://localhost:8081/api/v1/attendance/search?id=1
```

OR

```bash
curl http://localhost:8081/api/v1/attendance/search/all
```

Expected MONITOR Output:

```text
"GET" "flask_cache_view//api/v1/attendance/search"
"SETEX" "flask_cache_view//api/v1/attendance/search" "20"
```

---

# Attendance API Redis Flow

```text
Client Request
      ↓
Attendance API
      ↓
GET Cache Key
      ↓
Cache MISS
      ↓
PostgreSQL Query
      ↓
SETEX Cache Key 20
      ↓
Response Returned
```

---

# Understanding SETEX

```redis
SETEX key 20 value
```

Purpose:

* Stores cache data
* Automatically expires cache after 20 seconds

Meaning:

| Parameter | Value           |
| --------- | --------------- |
| key       | Redis cache key |
| 20        | TTL in seconds  |
| value     | Cached response |

---

# Step 8 — Verify Salary API Redis Operations

Run:

```bash
curl http://localhost:8082/api/v1/salary/search?id=1
```

Expected MONITOR Output:

```text
"GET" "com.opstree.microservice.salary.salary-search-id::1"
"SET" "com.opstree.microservice.salary.salary-search-id::1" "<java-object>" "PX" "1000"
```

---

# Salary API Redis Flow

```text
Client Request
      ↓
Salary API
      ↓
GET Cache Key
      ↓
Cache MISS
      ↓
Database Query
      ↓
SET Cache PX 1000
      ↓
Response Returned
```

---

# Understanding PX

```redis
SET key value PX 1000
```

Purpose:

* Stores cached data
* Automatically expires cache

Meaning:

| Parameter | Value                      |
| --------- | -------------------------- |
| PX        | Expiration in milliseconds |
| 1000      | 1 second TTL               |

---

# Redis Monitoring

## Monitor Live Redis Operations

```bash
redis-cli MONITOR
```

This command shows:

* GET requests
* SET requests
* HGET requests
* HSET requests
* Cache activity from all APIs

---

# Employee API Redis Verification

## Verify Employee API

```bash
curl http://localhost:8080/api/v1/employee/search?id=1
```

## Expected Redis Operations

```text
"hget" "employee" "1"
"hset" "employee" "1" "{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}"
```

## Verify All Employee Cache

```bash
curl http://localhost:8080/api/v1/employee/search/all
```

Expected Redis Output:

```text
"hget" "employee" "all_data"
"hset" "employee" "all_data" "[{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}]"
```

---

# Employee API Redis Commands

## Read Redis HASH Data

```redis
HGET employee 1
```

## Read Complete HASH

```redis
HGETALL employee
```

## Store HASH Data

```redis
HSET employee 1 "{"id":"1","name":"Mukesh Kharb","designation":"DevOps","department":"Engineering"}"
```

---

# Attendance API Redis Verification

## Verify Attendance API

```bash
curl http://localhost:8081/api/v1/attendance/search?id=1
```

## Verify All Attendance Data

```bash
curl http://localhost:8081/api/v1/attendance/search/all
```

## Expected Redis Operations

```text
"GET" "flask_cache_view//api/v1/attendance/search"
"SETEX" "flask_cache_view//api/v1/attendance/search" "20"
```

---

# Attendance API Redis Commands

## Check Cache TTL

```redis
TTL flask_cache_view//api/v1/attendance/search
```

## Verify Attendance Cache Keys

```redis
KEYS *attendance*
```

---

# Salary API Redis Verification

## Verify Salary API

```bash
curl http://localhost:8082/api/v1/salary/search?id=1
```

## Expected Redis Operations

```text
"GET" "com.opstree.microservice.salary.salary-search-id::1"
"SET" "com.opstree.microservice.salary.salary-search-id::1" "<java-object>" "PX" "1000"
```

---

# Salary API Redis Commands

## Verify Salary Cache Keys

```redis
KEYS *salary*
```

## Check Salary Cache TTL

```redis
TTL com.opstree.microservice.salary.salary-search-id::1
```

---

# Common Redis Commands

## Show All Keys

```redis
KEYS *
```

## Delete Specific Key

```redis
DEL employee
```

## Flush Complete Redis Cache

```redis
FLUSHALL
```

## Check Redis Memory Usage

```redis
INFO memory
```

## Check Redis Server Information

```redis
INFO server
```

## Check Connected Clients

```redis
CLIENT LIST
```

---

# Redis Cache Behavior

## Cache Miss

When data is not available in Redis:

```text
HGET → MISS
      ↓
Database Query
      ↓
HSET / SETEX / SET
```

---

## Cache Hit

When data already exists in Redis:

```text
GET / HGET
      ↓
Redis returns cached response
      ↓
No database query
```

---

# Redis Persistence Files

## Redis Configuration File

```bash
/etc/redis/redis.conf
```

## AOF File

```bash
/var/lib/redis/appendonly.aof
```

## Enable AOF

```conf
appendonly yes
```

---

# Important Notes

* Employee API uses HGET and HSET
* Attendance API uses GET and SETEX
* Salary API uses GET and SET PX
* Attendance API cache expires after 20 seconds
* Salary API cache expires after 1000 milliseconds
* Employee API cache does not expire automatically
