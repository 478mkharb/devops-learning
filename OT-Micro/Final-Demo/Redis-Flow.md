# OT-Microservices — Redis Data Cache Flow 🧠

> **Presentation Cheat Sheet**
>
> **Remember:** `API → Redis → HIT / MISS → Database`
>
> Redis is a **cache**, not the source of truth.

---

# 1. Overall Redis Architecture

Your three APIs use Redis differently:

```text
                         FRONTEND
                            │
             ┌──────────────┼──────────────────┐
             │              │                  │
                      ▼                       ▼                              ▼
       Employee API   Attendance API       Salary API
             │              │                  │
        ┌────┴────┐    ┌────┴────┐        ┌────┴──────┐
        │         │    │         │        │           │
              ▼              ▼      ▼               ▼             ▼                  ▼
      Redis    Scylla Redis   PostgreSQL Redis     Scylla
      CACHE    SOURCE CACHE   SOURCE     CACHE     SOURCE
```

### Source of Truth

| API | Persistent Database | Redis Role |
|---|---|---|
| Employee API | ScyllaDB | Explicit cache |
| Attendance API | PostgreSQL | Flask-Caching |
| Salary API | ScyllaDB | Spring Cache |

---

# 2. Generic Cache Pattern

For a READ request:

```text
Frontend
   │
   ▼
API
   │
   ▼
Redis
   │
   ├── HIT ──────────────→ Return cached data
   │
   └── MISS
        │
             ▼
     Database
        │
             ▼
      API
        │
        ├──────────────→ Frontend
        │
             ▼
      Redis
     store data
```

### Key idea

> **Redis is checked first. The database is consulted only when the cache misses.**

---

# 3. Employee API — ScyllaDB + Redis

## File paths

```text
Employee_API-main/
├── client/
│   ├── redis.go
│   └── scylladb.go
└── api/
    └── api.go
```

Redis configuration is in the Employee API configuration.

---

## 3.1 Redis structure

The Employee API uses a Redis hash:

```text
Redis DB 0

Hash: employee

 ├── EMP001       → employee JSON
 ├── EMP002       → employee JSON
 ├── EMP003       → employee JSON
 ├── all_data     → all employee JSON
 ├── designation  → designation data
 └── location     → location data
```

---

## 3.2 Employee READ — Cache HIT

The code checks Redis using:

```go
redisData, redisError =
    client.CreateRedisClient().
    HGet(ctx, "employee", id).
    Result()
```

Flow:

```text
GET /employee/search?id=EMP001
              │
                        ▼
        Employee API
              │
                        ▼
      Redis HGET employee EMP001
              │
            HIT
              │
                        ▼
        Return JSON
```

**ScyllaDB is not queried.**

---

## 3.3 Employee READ — Cache MISS

If Redis does not contain the employee:

```text
GET /employee/search?id=EMP001
              │
                        ▼
        Employee API
              │
                        ▼
          Redis HGET
              │
             MISS
              │
                        ▼
          ScyllaDB
              │
                        ▼
       employee_info
              │
                        ▼
        Employee API
          │       │                
                ▼            ▼
    Frontend     Redis
                  HSET
```

The cache is populated using:

```go
writeinRedis(id, string(jsonData))
```

and:

```go
client.CreateRedisClient().
    HSet(ctx, "employee", cacheKey, cacheValue)
```

So the next request can be served directly from Redis.

---

# 4. Employee — All Employee Data

For:

```text
GET /employee/search/all
```

the API checks:

```go
HGet(requestCtx, "employee", "all_data")
```

If the key exists:

```text
Redis
  │
  └── employee → all_data
                 │
                             ▼
              Return
```

If it does not:

```text
Redis MISS
    │
       ▼
ScyllaDB
    │
       ▼
Read all employees
    │
      ▼
Employee API
    │
    ├──→ Frontend
    │
    └──→ Redis
          employee → all_data
```

---

# 5. Employee — Designation and Location Cache

The API also caches lookup information.

### Designation

Redis key:

```text
employee → designation
```

The API first performs:

```go
HGet(ctx, "employee", "designation")
```

On MISS it reads ScyllaDB, creates the lookup response, and stores it in Redis.

### Location

Redis key:

```text
employee → location
```

The API checks:

```go
HGet(ctx, "employee", "location")
```

On MISS it reads ScyllaDB and repopulates the cache.

---

# 6. ⭐ Employee Cache Invalidation

This is the most important invalidation mechanism in your Employee API.

When a new employee is created:

```text
POST /employee/create
       │
            ▼
Employee API
       │
            ▼
INSERT into ScyllaDB
       │
            ▼
     SUCCESS
       │
            ▼
Redis DEL "employee"
```

The code performs:

```go
client.CreateRedisClient().
    Del(ctx, "employee")
```

### Why delete the whole hash?

Because one employee change can affect multiple cached views:

```text
employee
 ├── EMP001
 ├── EMP002
 ├── all_data
 ├── designation
 └── location
```

Instead of trying to update every cached representation, the application invalidates the whole hash.

Next READ:

```text
Redis MISS
     │
        ▼
ScyllaDB
     │
        ▼
Fresh data
     │
        ▼
Redis repopulated
```

### Memory line

> **Write DB → Delete cache → Next read rebuilds cache**

---

# 7. Attendance API — PostgreSQL + Redis

## File paths

```text
Attendance_API-main/
├── app.py
├── routes/
│   └── attendance.py
└── client/
    └── postgres/
        └── postgres_conn.py
```

The Attendance API uses:

```python
from flask_caching import Cache
```

and:

```python
cache = Cache()
```

---

# 8. Attendance Redis Configuration

In `app.py`:

```python
cache.init_app(
    app,
    get_caching_data(),
)
```

The cache configuration uses:

```python
"CACHE_TYPE": "redis"
```

and Redis settings:

```text
REDIS_HOST
REDIS_PORT
REDIS_PASSWORD
```

---

# 9. Attendance READ Cache

The search endpoint is decorated with:

```python
@route.route("/attendance/search", methods=["GET"])
@cache.cached(timeout=20)
def read_record():
```

Therefore the response is automatically cached in Redis for:

```text
20 seconds
```

Flow:

```text
GET /attendance/search
          │
                 ▼
     Redis Cache
      │        │
     HIT     MISS
      │        │
          ▼             ▼
  Response  PostgreSQL
              │
                        ▼
           records
              │
                        ▼
          Response
              │
                        ▼
            Redis
```

---

# 10. Attendance PostgreSQL Query

On a cache MISS, the application queries PostgreSQL:

```sql
SELECT id, name, status, date
FROM records
WHERE id = %s
```

PostgreSQL remains the persistent source of truth.

---

# 11. Attendance — Search All

The endpoint:

```python
@route.route("/attendance/search/all", methods=["GET"])
@cache.cached(timeout=20)
def read_all_record():
```

is also cached automatically.

Flow:

```text
GET /attendance/search/all
          │
                ▼
        Redis
      │        │          
     HIT     MISS
      │        │
          ▼             ▼
  Response  PostgreSQL
              │
                        ▼
       SELECT all records
              │
                        ▼
            Redis
              │
                        ▼
          Frontend
```

---

# 12. Attendance Cache Expiration

The cache decorator says:

```python
@cache.cached(timeout=20)
```

Therefore:

```text
Cache created
     │
     ├── 0 sec → available
     ├── 10 sec → available
     └── 20 sec → expires
```

After expiry:

```text
Next GET
   │
     ▼
Redis MISS
   │
     ▼
PostgreSQL
   │
     ▼
Fresh response
   │
     ▼
Redis
```

This is **TTL-based invalidation**.

---

# 13. Attendance WRITE

For an attendance create/update request:

```text
POST /attendance/create
        │
             ▼
 Attendance API
        │
             ▼
 PostgreSQL
        │
             ▼
 records table
```

The application does not explicitly populate Redis with the new attendance record.

The GET cache is populated when a GET request occurs.

So remember:

> **Attendance uses automatic response caching, not manual Redis data synchronization.**

---

# 14. Salary API — ScyllaDB + Redis

## File paths

```text
Salary_API-main/
└── src/main/java/com/opstree/microservice/salary/
    ├── SalaryApplication.java
    ├── controllers/
    │   └── SpringDataController.java
    └── services/
        └── SpringDataSalaryService.java
```

Redis is configured through Spring Data Redis.

---

# 15. Salary Redis Enablement

In:

```text
SalaryApplication.java
```

the application enables caching:

```java
@EnableCaching
```

and creates a Redis-backed:

```java
RedisCacheManager
```

The cache TTL is:

```java
.entryTtl(Duration.ofSeconds(1))
```

### Therefore:

> **Salary Redis cache TTL = 1 second**

---

# 16. Salary — Cached Method

In:

```text
SpringDataSalaryService.java
```

the salary lookup method has:

```java
@Cacheable("salary-search-id")
public Employee getEmployeeSalary(String id)
```

This means Spring automatically checks Redis before executing the method.

---

# 17. Salary READ — Cache MISS

First request:

```text
GET /salary/search?id=EMP001
              │
                        ▼
          Salary API
              │
                        ▼
        Redis Cache
              │
             MISS
              │
                        ▼
           ScyllaDB
              │
                        ▼
        employee_salary
              │
                        ▼
         Salary API
              │
              ├──────────→ Frontend
              │
                        ▼
            Redis
```

---

# 18. Salary READ — Cache HIT

If the same request arrives within the 1-second TTL:

```text
GET /salary/search?id=EMP001
              │
                        ▼
          Salary API
              │
                        ▼
            Redis
              │
             HIT
              │
                        ▼
        Return salary
```

ScyllaDB is not queried.

After 1 second:

```text
Redis entry expires
        │
             ▼
Next request → ScyllaDB
```

---

# 19. Salary WRITE

Salary creation:

```java
public Employee saveSalary(Employee employee) {
    employeeRepository.save(employee);
    return employee;
}
```

The repository writes to ScyllaDB.

```text
Frontend
   │
     ▼
Salary API
   │
     ▼
ScyllaDB
   │
     ▼
employee_salary
```

There is no explicit Redis `SET` here and no explicit `@CacheEvict` on this method.

So do **not** say:

> "Salary creation writes the salary into Redis."

Instead say:

> **"Salary is persisted in ScyllaDB; Redis caches salary lookup results."**

---

# 20. 🔥 Three Types of Cache Invalidation in Your Project

This is a very good interview topic.

## A. Employee — Explicit Invalidation

After successful database write:

```go
Del(ctx, "employee")
```

Pattern:

```text
WRITE DB
   ↓
DELETE CACHE
   ↓
NEXT READ
   ↓
REBUILD CACHE
```

---

## B. Attendance — TTL Invalidation

Using:

```python
@cache.cached(timeout=20)
```

Pattern:

```text
CACHE
  ↓
20 seconds
  ↓
EXPIRE
  ↓
NEXT READ → PostgreSQL
```

---

## C. Salary — TTL Invalidation

Using:

```java
.entryTtl(Duration.ofSeconds(1))
```

Pattern:

```text
CACHE
  ↓
1 second
  ↓
EXPIRE
  ↓
NEXT READ → ScyllaDB
```

---

# 21. ⭐ Critical Difference

| API | Cache Mechanism | Invalidation |
|---|---|---|
| Employee | Manual Redis hash | `DEL employee` after write |
| Attendance | Flask-Caching | TTL = 20 sec |
| Salary | Spring Cache | TTL = 1 sec |

---

# 22. What Happens if Redis Goes Down?

Redis is **not the source of truth**.

### Employee

```text
Redis unavailable
      ↓
Employee API cannot use cache
      ↓
ScyllaDB remains authoritative
```

### Attendance

```text
Redis unavailable
      ↓
Cache unavailable
      ↓
PostgreSQL remains authoritative
```

### Salary

```text
Redis unavailable
      ↓
Cache unavailable
      ↓
ScyllaDB remains authoritative
```

The key architectural principle is:

> **Losing Redis should mean losing cached performance, not losing persistent business data.**

---

# 23. 🎯 Interview Answer — "Explain Redis in your project"

> "We use Redis as a caching layer in all three APIs, but the implementation differs. The Employee API uses go-redis directly and stores employee data in a Redis hash. On a cache miss it reads ScyllaDB and populates Redis; after an employee write it invalidates the entire employee hash using DEL to avoid stale data. The Attendance API uses Flask-Caching with Redis, where GET responses are cached for 20 seconds and expire automatically. The Salary API uses Spring Cache backed by Redis, with `@Cacheable` on the salary lookup and a one-second TTL. In every case, the underlying database remains the source of truth."

---

# 🧠 Final Memory Diagram

```text
                         FRONTEND
                            │
            ┌───────────────┼───────────────┐
            │               │               │
                    ▼                         ▼                         ▼
       EMPLOYEE         ATTENDANCE        SALARY
          API              API              API
            │               │               │
                    ▼                         ▼                         ▼
          REDIS            REDIS           REDIS
       HGet/HSet/Del    Flask-Caching   Spring Cache
            │               │               │
            │ MISS          │ MISS          │ MISS
                    ▼                         ▼                         ▼
         SCYLLA          POSTGRESQL       SCYLLA
            │               │               │
            │               │               │
            └───────────────┴───────────────┘

Employee:
WRITE → ScyllaDB → DEL Redis → rebuild on next GET

Attendance:
GET → Redis → MISS → PostgreSQL → cache for 20 sec

Salary:
GET → Redis → MISS → ScyllaDB → cache for 1 sec
```

## 🚨 Three lines to memorize

> **Employee:** `ScyllaDB → Redis HSET`, and write causes `DEL employee`.

> **Attendance:** `PostgreSQL → Flask-Caching → Redis`, TTL **20 seconds**.

> **Salary:** `ScyllaDB → @Cacheable → Redis`, TTL **1 second**.

> **Overall:** **Database = source of truth | Redis = cache | API = bridge between them.**
