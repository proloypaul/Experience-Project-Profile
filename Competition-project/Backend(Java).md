<div align="center">

<img src="https://img.shields.io/badge/Spring%20Boot-3.5.0-6DB33F?style=for-the-badge&logo=springboot&logoColor=white" />
<img src="https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" />
<img src="https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white" />
<img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" />
<img src="https://img.shields.io/badge/AWS-SQS%20%7C%20S3%20%7C%20SES-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" />
<img src="https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white" />

# Competition Backend

**A high-performance, cloud-native backend for the Competition platform — designed to handle high-volume concurrent quiz submissions.**

</div>

---

## Table of Contents

- [Project Overview](#project-overview)
- [Core Features](#core-features)
- [Technology Stack](#technology-stack)
- [Architecture Design](#architecture-design)
- [High-Scale Submission Handling](#high-scale-submission-handling)
- [Caching Strategy](#caching-strategy)
- [Database & Schema Management](#database--schema-management)
- [Security](#security)
- [File Storage & Media](#file-storage--media)
- [CI/CD Pipeline](#cicd-pipeline)
- [File Structure](#file-structure)

---

## Project Overview

Production-grade competition platform backend — engineering case study

A portfolio case study documenting the architecture, engineering decisions, scalability techniques, and cloud technologies used to build a high-throughput backend for an academic competition platform.

The system was designed to support student registration, OTP verification, competition management, quiz delivery, automated grading, asynchronous submission processing, administrative dashboards, and cloud-based file storage.

The production implementation is proprietary and therefore not included in this repository. This documentation presents the technical concepts and architectural patterns used in the project without exposing company-confidential source code or infrastructure details.

---

## Core Features

### 🎓 Student Management
- Complete student lifecycle management: registration, profile updates, verification, and soft-deletion
- Mobile OTP-based identity verification using a configurable OTP system with rate limiting, cooldowns, and per-hour caps
- Secure password reset flow via OTP token with expiry enforcement
- Student data archiving with full audit trail

### 🏫 Venue & Registration
- Hierarchical venue management (districts, sub-districts, individual centers)
- Student competition registration linked to venue assignment
- Registration status tracking and bulk management tools for administrators
- Soft-delete support across all venue and registration records

### 📋 Notice & Announcements
- Admin-controlled notice board for broadcasting competition announcements to all students
- Full CRUD operations with active/inactive status toggling

### 📊 Admin Dashboard
- Centralized dashboard with real-time statistics on student registrations, quiz participation, and submission counts
- Role-protected admin endpoints with phone/password authentication

### 🧠 Quiz Engine
A multi-format quiz engine supporting six distinct question types:

| Question Type | Description |
|---|---|
| `MCQ` | Standard multiple-choice with one correct option |
| `SINGLE_TEXT` | Free-text answer (case-insensitive match) |
| `SINGLE_LETTER` | Single character answer (e.g., fill-in-the-blank) |
| `TRUE_FALSE_CONDITIONAL` | True/False with mandatory correction word when answer is False |
| `PARAGRAPH_CORRECTION` | Students identify and correct multiple errors embedded in a paragraph |
| `IMAGE_IDENTIFICATION` | Students type the answer to an image-based question |

Each quiz is composed of **sections**, where every section:
- Enforces a **single question type** for the entire section
- Has its own **independent countdown timer** (`duration_seconds`)
- Configures **marks per question** and **negative marking** independently
- Defines a custom **instruction text** displayed to the student

### 🏆 Dual Quiz Mode
- **Practice Mode** : Visible on the student dashboard; students can attempt freely
- **Live Competition Mode** : Hidden from students until activated; enforces strict start/end time windows and one-time-only submission

### 📬 Exam Submission & Auto-Grading
- Submissions are asynchronously queued via **AWS SQS** for decoupled, durable processing
- Automatic grading engine handles all six question types including **partial scoring** for paragraph correction
- Duplicate submission protection via database-level uniqueness constraint
- Submission results are available immediately after SQS worker processing completes

---

## Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Runtime** | Java 21 (Virtual Threads) | Lightweight concurrency without thread pool bottlenecks |
| **Framework** | Spring Boot | Core web, JPA, security, scheduling, actuator |
| **Database** | PostgreSQL | Primary relational data store with dedicated schema |
| **Cache** | Redis | Quiz bundle caching, student auth caching, OTP storage |
| **Message Queue** | AWS SQS | Asynchronous exam submission pipeline |
| **File Storage** | AWS S3 + S3 Transfer Manager | Question images, student answer JSON files |
| **Email** | AWS SES | OTP delivery and system notifications |
| **SMS** | ReVe SMS (configurable) | OTP delivery via SMS gateway |
| **Security** | Spring Security + JWT (JJWT) | Stateless auth with HTTP-only cookie tokens |
| **DB Migrations** | Flyway | Versioned, auditable schema evolution |
| **API Docs** | SpringDoc OpenAPI (Swagger UI) | Auto-generated, interactive API documentation |
| **ORM** | Hibernate + Spring Data JPA | Entity management with batch insert support |
| **Build** | Maven | Dependency management and packaging |
| **CI/CD** | Jenkins + SSH deploy | Automated build, provision, deploy, and health-check pipeline |
| **Monitoring** | Spring Actuator | Health checks and metrics endpoints |

---

## Architecture Design

The backend follows a **Domain-Driven, Layered Architecture** organized within four top-level domains. Each domain is fully self-contained with its own controllers, services, DTOs, entities, repositories, mappers, and specifications.

```
- API / Controller Layer
- Application / Service Layer
- Domain Layer
- Persistence Layer
- Infrastructure Integrations
- Security
- Configuration                       
```

## High-Scale Submission Handling

This is the most performance-critical aspect of the system. When a competition ends, thousands of students submit simultaneously. The system is designed specifically to handle this spike gracefully.

### The Three-Layer Submission Pipeline

```
Student Browser
      │
      │  POST 
      │  (Validation only — no DB write)
      ▼
  ┌─────────────────────────────────────────────────────────┐
  │  1. Fast Validation (in-cache)                           │
  │     • Quiz is active?                                    │
  │     • Within time window?                                │
  │     • Already submitted? (guard from cache)              │
  └───────────────────────────┬─────────────────────────────┘
                              │ Pass
                              ▼
  ┌─────────────────────────────────────────────────────────┐
  │  2. SQS Dispatch (< 50ms)                                │
  │     • Serialize payload → JSON                           │
  │     • Push to AWS SQS queue                              │
  │     • Return HTTP 200 immediately                        │
  └───────────────────────────┬─────────────────────────────┘
                              │ (async)
                              ▼
  ┌─────────────────────────────────────────────────────────┐
  │  3. SQS Worker (background — runs every 500ms)           │
  │     • Long-poll: pulls up to 10 messages at once         │
  │     • processBatchSubmission() → writes to PostgreSQL    │
  │     • Delete message on success                          │
  │     • Failed messages → retry or DLQ (Dead Letter Queue) │
  └─────────────────────────────────────────────────────────┘
```

### Why This Works at Scale

1. **Instant HTTP response**: The API never blocks on database I/O during submission. Students receive a success response the moment the payload hits SQS — typically under 50ms.

2. **Decoupled persistence**: The SQS worker runs as an independent scheduled task, processing up to 10 messages per poll cycle every 500ms. It can absorb bursts without degrading the API response time.

3. **Auto-scaling durability**: SQS retains unprocessed messages. If the server restarts mid-exam, no submission is lost — the worker resumes from where it left off.

4. **Fallback executor**: If SQS is unreachable, a `CompletableFuture` with a dedicated `ThreadPoolTaskExecutor` immediately takes over and processes the submission directly to the database asynchronously.

5. **N+1 prevention on batch insert**: The method pre-fetches all question IDs and option IDs from the submission payload in **two bulk queries** before the insert loop, completely eliminating N+1 query patterns even for large submissions.

6. **Java 21 Virtual Threads**: Enabled platform-wide via `spring.threads.virtual.enabled: true`, allowing thousands of concurrent HTTP connections to be served without the overhead of traditional OS threads.

7. **HikariCP connection pool**: Tuned with `maximum-pool-size` and a `connection-timeout` — sized deliberately to avoid overwhelming PostgreSQL while sustaining high throughput.

8. **Hibernate batch inserts**: `jdbc.batch_size` is configured at the JPA level, allowing bulk answer inserts to be sent in batches rather than individual SQL statements.

---

## Caching Strategy

The system uses a **Resilient Redis Cache Service** — a custom wrapper around Spring Data Redis that implements circuit-breaker semantics to prevent Redis failures from cascading into API failures.

### How the Resilient Cache Works

```
Request arrives
      │
      ▼
  Circuit Open?  ──YES──▶  Skip Redis, serve from DB
      │
      NO
      ▼
  Redis GET (with retry, max 2 attempts, 200ms delay)
      │
  ┌───┴────────────┐
  │                │
  HIT             MISS
  │                │
  Return         Fetch from DB
  cached         Cache result
  value          Return value
      │
  Failure after 2 retries?
      │
      YES ──▶ Open circuit for 30s, fallback to DB
```

### What Gets Cached

| Cached Data | Invalidated On |
|---| ---|
| Full quiz bundle (sections + questions) | Quiz/section update |
Currently active live quiz metadata | Quiz toggle |
Student credentials for login acceleration | Password change |
 OTP values with attempt counters | Verification or expiry |

### Cache-Aside for Exam Bundle

The quiz bundle endpoint (`GET /bundle`) is the single most frequently hit endpoint during an exam — every student loads it once. The bundle (which includes all sections and questions) is fetched from the database on the **first request only**, then cached in Redis for subsequent loads. This transforms what could be thousands of complex JPA queries into a single Redis `GET` per request, dramatically reducing database load during peak hours.

The **dynamic S3 presigned URL** is intentionally **not** cached — it is generated fresh for each student on each bundle load, ensuring per-student upload isolation.

---

## API Design

All APIs follow a consistent RESTful design wrapped in a unified `ApiResponse<T>` envelope:

```json
{
  "success": true,
  "message": "....",
  "data": { ... }
}
```

### API Base URL
```
http://<host>:9090/api/v1
```

### Interactive Docs
Swagger UI is available in development at:
```
http://<host>:9090/swagger-ui.html
```

## Database & Schema Management

### Schema
All tables live under the dedicated PostgreSQL schema

This schema isolation ensures the application data is logically separated from any other services using the same PostgreSQL instance.

### Flyway Migrations
Schema changes are managed exclusively through **Flyway** versioned migrations, ensuring the database state is fully reproducible and auditable across all environments.

### Key Performance Indexes

The schema includes carefully placed composite indexes targeting the most frequent query patterns:

## Security

### JWT Authentication
- **Access Token**: Short-lived (configurable, default days in dev), stored in HTTP-only cookies
- **Refresh Token**: Longer-lived, stored in HTTP-only cookies
- **Stateless**: No server-side session storage; all claims are embedded in the JWT
- Token validation is handled by a custom `JwtAuthenticationFilter` that runs before every secured request

### Role-Based Access Control
| Role | Access Scope |
|---|---|
| `ADMIN` | All `/api/v1/admin/**` endpoints, submission review |
| `STUDENT` | `/api/v1/student/**`, profile management, quiz access |

### OTP Security
- OTP values are stored in **Redis** (not the database) with a 5-minute TTL
- Rate limiting: max **5 OTP attempts** before lockout; **120-second** resend cooldown; max **5 OTPs per hour** per phone number
- OTPs are HMAC-signed with a configurable secret for tamper resistance

### Cookie Security
- `HttpOnly`: Prevents JavaScript access to tokens
- `SameSite`: Configurable (`Lax` in dev, `Strict` in prod)
- `Secure`: Enforced in production (HTTPS-only transmission)

### CORS
Allowed origins are environment-specific and fully configurable — tight in production, permissive in local development.

---

## File Storage & Media

### S3 Architecture: Client-Side Upload Pattern

Instead of routing file uploads through the application server (which would add unnecessary latency and memory pressure), the system uses a **presigned URL** workflow:

```
1. Client → API:   POST /presign-image-url (metadata only)
2. API → S3:       Generate presigned PUT URL (expires in 2h)
3. API → Client:   Return { presignedUrl, permanentFileUrl }
4. Client → S3:    PUT binary directly to S3 (bypasses backend)
5. Client → API:   Store permanentFileUrl in question entity
```

This means the backend **never handles raw binary data** for question images, keeping it lightweight and horizontally scalable.

### Exam Answer JSON Storage
For live exam submissions, each student's full answer payload is also uploaded to S3 as a JSON file at `/competition-{id}/student-{id}.json` — providing a durable audit record independent of the database, useful for dispute resolution.

---

## CI/CD Pipeline

The project uses a **Jenkins declarative pipeline** with SSH-based deployment to a production VPS.


## File Structure

```
competition/
├── Jenkinsfile                          # CI/CD pipeline definition
├── pom.xml                              # Maven dependencies and build config
└── src/
    └── main/
        ├── java/com/projectName/
        │   ├── Application.java      # Spring Boot entry point
        │   ├── config/
        │   │   └── AsyncConfig.java     # Fallback executor thread pool config
        │   ├── common/
        │   │   ├── BaseEntity.java      # Shared audit fields
        │   │   ├── config/              # Redis, S3, CORS, OpenAPI configurations
        │   │   ├── constants/
        │   │   │   └── RedisCacheKeys.java   # Centralized Redis key definitions
        │   │   ├── exception/           # Global exception handler, custom exceptions
        │   │   ├── response/
        │   │   │   └── ApiResponse.java # Unified API response envelope
        │   │   ├── service/
        │   │   │   ├── ResilientRedisCacheService.java   # Circuit-breaker Redis wrapper
        │   │   │   ├── FileStorageService.java            # S3 upload + presign abstraction
        │   │   │   ├── FileStorageServiceImpl.java
        │   │   │   ├── EmailService.java                  # AWS SES email sending
        │   │   │   ├── AwsSesEmailServiceImpl.java
        │   │   │   ├── SmsService.java                    # SMS OTP gateway
        │   │   │   └── SmsServiceImpl.java
        │   │   └── util/
        │   │       └── JwtTokenProvider.java              # JWT generation, validation, cookie helpers
        │   ├── security/
        │   │   ├── JwtAuthenticationFilter.java           # Per-request JWT validation filter
        │   │   └── SecurityConfig.java                    # Spring Security filter chain config
        │   └── domain/
        │       └── moduleName/
        │           └── controller/
        │           └── service/
        │           └── entity/
        │           └── dto/
        │           └── enums/
        │           └── exception/
        │           └── mapper/
        │           └── repository/
        │           └── specification/
        │     
        └── resources/
            ├── application.yaml         # Profile selector
            ├── application-dev.yaml     # Development config (local Redis, PostgreSQL, S3)
            ├── application-prod.yaml    # Production config (VPS-specific secrets via env)
            └── db/
                └── migration/           # Flyway SQL scripts (V1–V12)
```

---

## 👨‍💻 My Role & Contribution

This project was developed as part of a **professional employment** engagement at a technology company. My contributions included end-to-end development of all features and core architectural components described in this documentation, with the sole exception of the CI/CD pipeline and DevOps infrastructure, which were handled by a dedicated DevOps team. Key responsibilities included:

- 🔨 Designing and implementing the Domain-Driven Layered Architecture.
- 🧠 Engineering the high-scale asynchronous exam submission pipeline using AWS SQS.
- 🔐 Developing the security, authentication, and caching strategies.
- 📦 Building the multi-format quiz engine and automated grading logic.
- 📊 Integrating cloud services (AWS S3, SES, SQS) and structuring the PostgreSQL database.

---

## 📄 Confidentiality Notice

> 🔒 **This project is proprietary and confidential.**
>
> The source code, client identity, database schemas, API keys, environment configurations, and all internal business logic are protected under a company Non-Disclosure Agreement (NDA). No source code or sensitive data is disclosed in this document.
>
> This README is intended **solely as a professional portfolio reference** to demonstrate implementation skills and technical experience to potential employers.

---

<div align="center">

**Built with ❤️ for the Competition platform*

*Engineered for scale · Designed for reliability · Built for students*

</div>
