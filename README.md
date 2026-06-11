This URL shortener is a high-performance backend service designed to convert long URLs into compact, shareable short links while ensuring fast redirection, scalability, and operational reliability.

## Built With
- __FastAPI & asyncio__: High-performance asynchronous API gateway ecosystem maximizing concurrent request throughput
- __Python__: Core execution language utilizing type validation and secure cryptographic standard libraries
- __Redis__: High-speed in-memory data store running decoupled, multi-instance transaction pools
- __MySQL__ – Relational database engine configured with connection recycling and pre-ping validation for high-availability persistent storage
- __SQLAlchemy__ – Database abstraction layer driving atomic transactions and optimized session lifecycles
- __Docker__ – Containerization platform ensuring isolated microservice environments and consistent production deployments
- __GitHub Actions__ – Managed automation engine coordinating continuous background database hygiene workflows

## System Architecture
- __Asynchronous API Gateway & Persisted Storage__: At its core, the system accepts RFC-compliant long URLs through a validated Pydantic API gateway and persists them in a MySQL database. The MySQL engine is further configured with connection recycling and pre-ping validation to eliminate pessimistic disconnect states.
- __Deterministic O(1) Short Code Encoding__: Each URL is assigned a unique numeric identifier utilizing an unsigned BIGINT key. Using an atomic session-flushing strategy, this key is then converted into a short, human-friendly string using a custom Base62 encoding algorithm. This approach ensures that generated short codes are compact, URL-safe, and efficiently derived with a deterministic O(1) complexity without requiring additional lookup tables or collision handling mechanisms.
- __Write-Through Caching & Cache-Aside Fallback__: Once a short URL is created, the system executes a write-through caching pattern, immediately seeding the mapping between the short code and the original long URL in Redis. This caching layer significantly reduces database load and enables ultra-fast read performance during redirection. When a user accesses a short URL, the service first checks Redis for a cached entry. If found, it returns the original URL instantly; otherwise, it falls back to querying the indexed database table. This cache-aside fallback architecture is entirely encapsulated in fault-tolerant execution blocks, ensuring both speed and core system resilience.
- __Atomic Sliding Window Rate Limiter__: The application includes a rate-limiting mechanism implemented using Redis Sorted Sets and transactional pipelines. Each incoming request is tracked per client IP within a sliding window log, allowing the system to enforce strict request limits while minimizing network round-trip overhead. If a client exceeds the allowed number of requests, the service responds with an appropriate HTTP error, maintaining system stability under load. Crucially, the Redis caching layer and the rate-limiting middleware run on isolated connection pools to eliminate database performance bottleneck cross-contamination.
- __Defensive Cryptography & Administrative Isolation__: Security and administrative control are handled through HTTP Basic Authentication for protected endpoints, hardened with constant-time cryptographic string comparisons to immunize the gateway against side-channel timing attacks. These endpoints allow only Authorized Admins to verify the individual health diagnostics of the database and Redis instances, ensuring that all critical infrastructure components are operational.
- __Automated Continuous Data Hygiene__: To maintain data hygiene and control storage growth, the service enforces a time-based expiration policy. URLs older than 15 days are automatically removed from the database through a scheduled GitHub Actions workflow that executes a direct cleanup query daily. This ensures that stale data does not accumulate and that the system remains efficient over time.
- __High-Precision Telemetry Middleware__: Additionally, the service includes a custom middleware layer that profiles performance on high-precision decimal counters, injecting latency benchmarks directly into client payloads within custom X-Response-Time-Ms network headers. With an internal redirection latency of under 220 milliseconds, the system is optimized for fast user experiences.

Overall, this URL shortener is a well-structured, production-oriented backend system that combines efficient encoding, database persistence, caching, rate limiting, and automated maintenance to deliver a reliable and scalable link-shortening solution.

# Local Setup
## Prerequisites
- Python 3.9+
- MySQL database
- Dockerized Redis

## Clone the Repository
git clone 
<br>
cd url-shortener

## Create a Virtual Environment
python -m venv <venv_name>
- __Linux / Mac__: <venv_name>/bin/activate
- __Windows__: <venv_name>\Scripts\activate

## Install Dependencies
pip install -r requirements.txt

## Environment Configuration
Create a `.env` file in the root directory:
- __DATABASE_URL__=mysql+pymysql://[user]:[password]@localhost:3306/<db_name>
- __REDIS_CACHE_URL__=redis://localhost:6379/<db_number_0-15>
- __REDIS_RATE_LIMITER_URL__=redis://localhost:6379/<db_number_0-15>
- __ADMIN__=[username]
- __PASSWORD__=<admin_password>
- __BASE_URL__=http://localhost:8000

__Note__: Remove `SSL/CA` certificate arguments from the `create_engine` call in `database.py` file.

## Run the application
uvicorn app.main:app --reload
<br>
<br>
API will be available at: `http://localhost:8000`
<br>
Access the interactive Swagger documentation at: `http://localhost:8000/docs`
