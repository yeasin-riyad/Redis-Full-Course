# 🚀 Redis Course Backend

A hands-on **Redis + Node.js backend project** built to understand Redis from the fundamentals to real-world backend use cases.

This repository focuses on learning how Redis works as a **fast in-memory data store** and how it can be integrated into a Node.js backend for **caching, rate limiting, and Pub/Sub communication**.

---

## 📚 What You'll Learn

This project covers Redis concepts step-by-step, starting from the basics and progressing toward practical backend implementations.

### 🔥 Redis Fundamentals

* What Redis is and why we need it
* Why Redis is used as a fast in-memory layer
* Redis setup with Docker
* Installing the Redis client package in Node.js
* Creating a reusable Redis connection file
* Testing Redis connection with `PING`

### 🧱 Redis Data Types

Understanding Redis' core data structures:

* Redis keys and values
* Strings
* Hashes
* Lists
* Sets
* Sorted Sets

### ⏱️ Expiry & TTL

* Redis key expiration
* TTL (Time To Live)
* Why expiration is useful for temporary data
* Understanding automatic cache expiration

---

# ⚡ Redis Caching

One of the main goals of this project is understanding how Redis can dramatically improve API performance by caching frequently requested data.

### 🛒 Product API Caching

Caching has been implemented for:

```text
GET /products
GET /products/:id
```

The project demonstrates:

* Cache-aside pattern
* Cache hit
* Cache miss
* `JSON.stringify()`
* `JSON.parse()`
* Clean Redis key naming
* Cache invalidation
* Stale cache problem
* Why TTL alone is not enough
* Delete-based cache invalidation

### 🔄 Cache Flow

```text
Client
  │
  │ GET /products
  ↓
Backend API
  │
  ├── Check Redis
  │
  ├── Cache HIT ───────→ Return cached data
  │
  └── Cache MISS
          │
          ↓
      PostgreSQL
          │
          ↓
      Store in Redis
          │
          ↓
      Return response
```

### 💡 Cache Hit

When the requested data already exists in Redis:

```text
Request
   ↓
Redis
   ↓
Cache HIT ✅
   ↓
Return cached data
```

The application doesn't need to query PostgreSQL.

### ❌ Cache Miss

When the data doesn't exist in Redis:

```text
Request
   ↓
Redis
   ↓
Cache MISS ❌
   ↓
PostgreSQL
   ↓
Save result to Redis
   ↓
Return response
```

---

# 🧹 Cache Invalidation

Caching creates an important problem:

> What happens when the database changes but the cached data is still old?

This project explores the **stale cache problem** and different ways to handle it.

### Product Creation

After creating a product:

```text
POST /products
      ↓
Create product in PostgreSQL
      ↓
Invalidate product cache
```

### Product Update

After updating a product:

```text
PUT /products/:id
      ↓
Update PostgreSQL
      ↓
Invalidate related Redis cache
```

### Why TTL Alone Is Not Enough

TTL eventually removes stale data, but the cache can still contain outdated information until the TTL expires.

Therefore, this project demonstrates **delete-based cache invalidation** so that cached data can be removed immediately when the underlying data changes.

---

# 🚦 Redis Rate Limiting

This project also implements **Redis-based API rate limiting**.

The rate limiter tracks the number of requests made by each IP address.

### Concepts Covered

* Tracking request count per IP address
* Redis `INCR` command
* Expiry-based fixed time window
* Returning `429 Too Many Requests`
* Redis-based distributed rate limiting
* Why Redis rate limiting is better than simple in-memory rate limiting

### Rate Limiting Flow

```text
Incoming Request
       │
       ↓
Get Client IP
       │
       ↓
Redis INCR
       │
       ├── Within limit ───→ Continue Request
       │
       └── Limit exceeded
                    │
                    ↓
             429 Too Many Requests
```

### Example

Suppose an API allows:

```text
100 requests / minute / IP
```

Redis can maintain a counter:

```text
rate_limit:192.168.1.10 → 73
```

Every request increments the counter:

```text
INCR rate_limit:192.168.1.10
```

When the limit is exceeded:

```http
429 Too Many Requests
```

is returned.

---

# 📡 Redis Pub/Sub

The project also explores Redis **Publish/Subscribe (Pub/Sub)** for event-driven communication.

### Concepts Covered

* Redis Publisher
* Redis Subscriber
* Redis Channels
* Publishing notification events
* Listening for notification events
* Separate subscriber process
* Publisher → Channel → Subscriber architecture

### Notification API

A notification endpoint is implemented:

```http
POST /notifications
```

When a notification is created, the backend publishes an event to a Redis channel.

```text
                 Redis Channel
                      │
                      │
             "notifications"
                      │
             ┌────────┴────────┐
             ↓                 ↓
         Publisher          Subscriber
             │                 │
             ↓                 ↓
       POST /notifications   Event Listener
```

### Publisher

The publisher sends an event:

```text
Publisher
    │
    │ publish()
    ↓
Redis Channel
```

### Subscriber

The subscriber listens to the channel:

```text
Redis Channel
    │
    │ message
    ↓
Subscriber
    │
    ↓
Handle notification event
```

This demonstrates how Redis can be used not only for caching, but also for **communication between different processes/services**.

---

# 🛠️ Tech Stack

| Technology    | Purpose                        |
| ------------- | ------------------------------ |
| 🟢 Node.js    | Backend runtime                |
| 🟦 TypeScript | Type-safe development          |
| 🚂 Express.js | REST API framework             |
| 🐘 PostgreSQL | Primary database               |
| 🔴 Redis      | Cache, rate limiting & Pub/Sub |
| 🐳 Docker     | Local infrastructure           |
| 📦 `pg`       | PostgreSQL client              |
| 📦 `redis`    | Node.js Redis client           |

---

# 🏗️ Architecture

The project uses PostgreSQL as the primary persistent database and Redis as a fast in-memory layer.

```text
                    ┌──────────────┐
                    │    Client    │
                    └──────┬───────┘
                           │
                           ↓
                    ┌──────────────┐
                    │  Express API │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ↓            ↓            ↓
           Redis      PostgreSQL     Redis Pub/Sub
              │            │            │
              │            │       ┌────┴────┐
              │            │       ↓         ↓
           Caching       Storage  Publisher Subscriber
              │
              ↓
        Rate Limiting
```

---

# 🐳 Docker Setup

Redis and PostgreSQL can be run locally using Docker.

Example architecture:

```text
Docker
├── PostgreSQL
│   └── Port: 5432
│
└── Redis
    └── Port: 6379
```

This keeps the development environment consistent and makes it easy to start the required infrastructure.

---

# ⚙️ Environment Variables

Create a `.env` file in the project root:

```env
PORT=5000

DATABASE_HOST=localhost
DATABASE_PORT=5439
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
DATABASE_NAME=redis_course_db
DATABASE_URL=postgresql://postgres:postgres@localhost:5439/redis_course_db

REDIS_URL=redis://localhost:6380
```

> ⚠️ Never commit your real `.env` file or production credentials to GitHub.

---

# 🚀 Getting Started

## 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <your-project-directory>
```

## 2. Install Dependencies

```bash
npm install
```

## 3. Start Docker Services

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

## 4. Configure Environment Variables

Create:

```text
.env
```

and add the required PostgreSQL and Redis configuration.

## 5. Start Development Server

```bash
npm run dev
```

---

# 🧪 Redis Connection Test

The application tests the Redis connection using:

```text
PING
```

A successful connection returns:

```text
PONG
```

Example:

```text
Redis client connected
Redis client ready
redis ping response PONG
```

---

# 🔑 Redis Key Naming

A consistent Redis key naming strategy makes cache management easier.

Examples:

```text
products:all
product:1
product:2
product:3
rate-limit:192.168.1.10
```

Using predictable key names makes it easier to:

* Find cached data
* Delete specific cache entries
* Debug Redis
* Avoid key collisions
* Manage invalidation

---

# 🧠 Important Redis Concepts

By completing this project, you will understand:

### Redis as an In-Memory Layer

```text
PostgreSQL
    ↓
Persistent storage
    ↓
Slower than memory

Redis
    ↓
RAM / In-memory
    ↓
Very fast access
```

### Cache-Aside Pattern

```text
Application
     │
     ↓
 Check Redis
   /     \
 HIT     MISS
  │        │
  ↓        ↓
Return   Database
data        │
            ↓
       Save to Redis
            │
            ↓
       Return data
```

### Cache Invalidation

```text
Database changes
       ↓
Delete stale Redis cache
       ↓
Next request → Cache MISS
       ↓
Fetch fresh database data
       ↓
Store fresh data in Redis
```

### Rate Limiting

```text
IP Address
    ↓
Redis INCR
    ↓
Request Counter
    ↓
Check Limit
   /      \
Allowed   Exceeded
  ↓          ↓
Continue   HTTP 429
```

### Pub/Sub

```text
Publisher
    │
    ↓
Redis Channel
    │
    ↓
Subscriber
```

---

# 📖 Learning Progress

* [x] Redis fundamentals
* [x] Redis setup with Docker
* [x] Redis client integration
* [x] Redis connection management
* [x] Redis `PING`
* [x] Redis data types
* [x] Redis keys and values
* [x] Strings
* [x] Hashes
* [x] Lists
* [x] Sets
* [x] Sorted Sets
* [x] Expiry & TTL
* [x] Redis caching
* [x] Cache-aside pattern
* [x] Cache hit & cache miss
* [x] Product API caching
* [x] Cache invalidation
* [x] Stale cache problem
* [x] Delete-based invalidation
* [x] Redis rate limiting
* [x] `INCR`
* [x] Fixed time window
* [x] HTTP `429`
* [x] Redis Pub/Sub
* [x] Publisher
* [x] Subscriber
* [x] Redis channels
* [x] Notification events

---

# 🎯 Why This Project?

Redis is much more than a simple key-value store.

This project focuses on understanding **why Redis is useful in real backend systems**, including:

* ⚡ Improving API response time
* 🗄️ Reducing database load
* 🚦 Protecting APIs from excessive requests
* 📡 Communicating between processes
* 🔄 Building event-driven backend features
* 🧠 Understanding real-world caching strategies

The goal is not just to memorize Redis commands, but to understand **where, why, and how Redis should be used in backend architecture**.

---

# 🌱 What's Next?

Possible improvements and advanced topics:

* [ ] Redis transactions
* [ ] Redis pipelines
* [ ] Redis Streams
* [ ] Distributed locking
* [ ] Advanced caching strategies
* [ ] Sliding-window rate limiting
* [ ] Redis Cluster
* [ ] Redis Sentinel
* [ ] Docker production setup
* [ ] Redis monitoring
* [ ] Background job processing with Redis
* [ ] BullMQ integration
* [ ] Distributed systems with Redis

---

# 👨‍💻 Author

**Md. Yeasin Mazumder**

Backend Developer | Full-Stack Enthusiast | Problem Solver

* 💼 LinkedIn: [Md. Yeasin Mazumder](https://www.linkedin.com/in/mdyeasinmazumder)
* 🌐 Portfolio: [Portfolio](https://riyad-dev-i3aj.vercel.app/)
* 📧 Email: [mazumderyeasin98@gmail.com](mailto:mazumderyeasin98@gmail.com)

---

## ⭐ Support

If this project helped you understand Redis or backend development, consider giving the repository a ⭐.

Happy coding! 🚀
