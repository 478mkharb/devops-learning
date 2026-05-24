# Mock Questions & Answers – Attendance API (OT-Microservices)

---

## 1. Explain the complete architecture flow of the Attendance API.

### Answer

The Attendance API is a Python-based microservice developed using Flask. It is responsible for handling attendance-related operations such as marking attendance, retrieving records, and validating attendance data.

The request flow is:

```text
Frontend / Client
        ↓
NGINX / Reverse Proxy
        ↓
Gunicorn Server
        ↓
Flask Attendance API
        ↓
Controllers & Services
        ↓
Redis Cache Lookup
   ↙                 ↘
Cache Hit         Cache Miss
   ↓                  ↓
Fast Response    PostgreSQL Query
                      ↓
                 Attendance Records
                      ↓
                 JSON Response
```

PostgreSQL is used as the primary relational database for permanent attendance storage, while Redis is used as a caching layer to improve API response time and reduce database load.

---

## 2. In which programming language is the Attendance API written and what are its main dependencies?

### Answer

The Attendance API is developed using Python and uses the Flask framework for REST API development.

Python is preferred because it provides:

* Faster backend development
* Clean and readable syntax
* Strong library ecosystem
* Excellent support for microservices
* Easy database and cache integration

The main dependencies used in the Attendance API are:

| Dependency             | Purpose                                     |
| ---------------------- | ------------------------------------------- |
| Flask                  | REST API framework                          |
| Gunicorn               | Production WSGI server                      |
| Poetry                 | Dependency & virtual environment management |
| psycopg2               | PostgreSQL connectivity                     |
| Redis client           | Redis cache integration                     |
| Liquibase              | Database schema migrations                  |
| PostgreSQL JDBC Driver | Liquibase database communication            |

Dependencies are managed using:

```text
pyproject.toml
poetry.lock
```

---

## 3. Explain how the Attendance API executes internally.

### Answer

The Attendance API executes using Gunicorn as the WSGI application server.

Execution lifecycle:

1. Gunicorn starts the Flask application
2. Flask initializes API routes and controllers
3. `config.yaml` is loaded
4. PostgreSQL and Redis connections are initialized
5. Gunicorn workers start listening on port 8081
6. HTTP requests are received from the frontend
7. Controllers process requests
8. Services interact with Redis and PostgreSQL
9. JSON responses are returned to clients

The application is started using:

```bash
nohup poetry run gunicorn --bind 0.0.0.0:8081 app:app
```

Gunicorn manages worker processes and handles concurrent request execution efficiently.

---

## 4. Why is a virtual environment required for the Attendance API?

### Answer

A virtual environment is an isolated Python environment used to manage project-specific dependencies separately from system-wide Python packages.

Virtual environments are important because:

* Different projects may require different package versions
* Global package installation creates dependency conflicts
* It prevents accidental modification of system Python packages
* It improves deployment consistency
* CI/CD builds become reproducible and stable

Poetry automatically manages virtual environments.

Example command:

```bash
poetry env use /usr/bin/python3.11
```

This configures Python 3.11 specifically for the Attendance API environment.

---

## 5. Explain the purpose of `pyproject.toml` in the Attendance API.

### Answer

`pyproject.toml` is the main configuration and dependency definition file used in modern Python projects.

It stores:

* Project metadata
* Dependency definitions
* Python version requirements
* Build configuration
* Poetry configuration

Poetry reads this file to install dependencies and manage the project environment.

Example:

```text
pyproject.toml
```

The related file:

```text
poetry.lock
```

stores exact dependency versions to ensure reproducible builds across environments.

---

## 6. Explain why Gunicorn is used to run the Attendance API.

### Answer

Gunicorn is used because Flask’s built-in development server is not production-ready.

Gunicorn provides:

* Multiple worker processes
* Better concurrency handling
* Improved production stability
* Efficient HTTP request processing
* WSGI-based application execution

The API is started using:

```bash
nohup poetry run gunicorn --bind 0.0.0.0:8081 app:app
```

Explanation:

| Part         | Purpose                                         |
| ------------ | ----------------------------------------------- |
| nohup        | Keeps application running after terminal closes |
| poetry run   | Executes inside Poetry environment              |
| gunicorn     | Production WSGI server                          |
| --bind       | Defines IP and port                             |
| 0.0.0.0:8081 | Exposes API on port 8081                        |
| app:app      | Flask application entry point                   |

---

## 7. Explain the role of Redis in the Attendance API.

### Answer

Redis is used as a caching layer in the Attendance API.

Its responsibilities include:

* Caching frequently accessed attendance records
* Reducing direct PostgreSQL queries
* Improving API response time
* Temporary in-memory storage
* Improving scalability under high traffic

Redis follows:

```text
Cache Hit → Fast Response
Cache Miss → Query PostgreSQL
```

If Redis becomes unavailable, the API still works using PostgreSQL directly, but response times may increase.

---

## 8. Explain why PostgreSQL is preferred for attendance records.

### Answer

PostgreSQL is preferred because attendance data is highly structured and requires strong consistency, reliability, transactional safety, and permanent storage.

Attendance systems manage critical enterprise data such as:

* Employee attendance entries
* Login and logout timestamps
* Working hours
* Leave records
* Historical attendance reports
* Salary-related attendance calculations

This type of data cannot tolerate inconsistency or accidental data corruption.

PostgreSQL supports ACID properties.

### Meaning of ACID in Attendance API

| Property    | Explanation                                                                |
| ----------- | -------------------------------------------------------------------------- |
| Atomicity   | Attendance operations complete fully or fail completely                    |
| Consistency | Database always remains in a valid state                                   |
| Isolation   | Multiple users updating attendance simultaneously do not corrupt data      |
| Durability  | Once attendance is saved, it remains permanently stored even after crashes |

These properties are extremely important because attendance data directly impacts:

* Salary generation
* Employee reports
* Payroll processing
* Compliance tracking
* HR auditing

### Why PostgreSQL Is Preferred Over Other Databases

#### Compared to MongoDB

MongoDB is schema-flexible and document-oriented.

However attendance systems require:

* Strong relational consistency
* Structured tabular data
* Transaction-heavy operations
* Reliable joins and reporting

PostgreSQL handles these scenarios more effectively than document databases.

---

#### Compared to MySQL

Both MySQL and PostgreSQL are relational databases.

However PostgreSQL is preferred because it provides:

* Better standards compliance
* Stronger transactional reliability
* Advanced indexing capabilities
* Better concurrency handling
* More powerful query optimization
* Better support for complex analytical queries

PostgreSQL is generally considered more suitable for enterprise-grade transactional systems.

---

### Additional Advantages of PostgreSQL

* Relational schema management
* Foreign key constraints
* Backup and recovery support
* Complex query execution
* Indexing for faster searches
* High reliability in production systems
* Long-term durable storage

Because attendance data is mission-critical and highly relational, PostgreSQL becomes a more reliable and production-ready choice compared to cache databases or schema-less document databases.

---

## 9. Explain the role of Makefile in the Attendance API.

### Answer

The Makefile is used to standardize project operations and simplify repetitive development tasks.

Example:

```bash
make run-migrations
```

Benefits of Makefile:

* Simplifies complex commands
* Reduces manual operational errors
* Standardizes workflows across teams
* Improves automation consistency
* Simplifies CI/CD execution

Developers can execute predefined commands without remembering lengthy operational scripts.

---

## 10. How can the port of the Attendance API be changed permanently?

### Answer

The Attendance API port is controlled mainly through the Gunicorn startup command.

Current example:

```bash
nohup poetry run gunicorn --bind 0.0.0.0:8081 app:app
```

To permanently change the port:

```bash
nohup poetry run gunicorn --bind 0.0.0.0:9090 app:app
```

The new port replaces:

```text
8081
```

In production environments, this command is usually configured inside:

* Systemd service files
* Jenkins deployment scripts
* Docker containers
* Kubernetes manifests
* Shell startup scripts

If NGINX or a load balancer is used, upstream configurations must also be updated to forward traffic to the new backend port.

Firewall rules and security groups may also require updates for the modified port.
