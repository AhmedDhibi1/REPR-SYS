# REPR-SYS

> **Ensemble Registration Manager** — A Spring Boot application for scheduling and managing remote mob programming (REPR-SYS) learning sessions.

[![Build Status](https://img.shields.io/github/actions/workflow/status/AhmedDhibi1/REPR-SYS/maven.yml?branch=main)](https://github.com/AhmedDhibi1/REPR-SYS/actions)
[![Java](https://img.shields.io/badge/Java-25-blue.svg)](https://openjdk.org/projects/jdk/25/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue.svg)](https://www.postgresql.org/)

---

## Table of Contents

- [Description](#description)
- [Business Problem](#business-problem)
- [Features](#features)
- [Architecture](#architecture)
- [Technologies](#technologies)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Environment Variables](#environment-variables)
- [Running Locally](#running-locally)
- [Running with Docker](#running-with-docker)
- [Database](#database)
- [API Overview](#api-overview)
- [Testing](#testing)
- [Build](#build)
- [Development Workflow](#development-workflow)
- [Future Improvements](#future-improvements)
- [License](#license)

---

## Description

**Ensembler** is a web-based platform designed to streamline the organization of remote ensemble (mob) programming sessions. It provides a complete workflow from scheduling sessions, managing participant registrations, automatically creating video conference meetings, running in-session rotation timers, to sending notifications and distributing session recordings.

The application follows **Domain-Driven Design (DDD)** principles and **Hexagonal Architecture** to ensure a clean separation between business logic and infrastructure concerns.

---

## Business Problem

Organizing remote ensemble programming sessions involves significant coordination overhead:

- **Scheduling**: Finding suitable times across time zones
- **Registration**: Tracking who is participating, spectating, or declining
- **Video Conferencing**: Manually creating Zoom meetings for each session
- **Communication**: Notifying participants of schedule changes and providing calendar invites
- **Session Management**: Running rotation timers and managing driver/navigator roles during sessions
- **Recordings**: Distributing session recordings to participants

**Ensembler automates all of these tasks**, allowing organizers to focus on the content of the sessions rather than the logistics.

---

## Features

### For Administrators
- 📅 **Schedule Ensembles** — Create sessions with name, date/time, and duration
- 🎥 **Auto-create Zoom Meetings** — Automatic video conference generation via Zoom API
- 👥 **Manage Participants** — Manual registration, view participant details
- ✅ **Complete Sessions** — Mark sessions as complete and attach recording links
- ❌ **Cancel Sessions** — Cancel sessions with automatic cleanup
- 📧 **Send Notifications** — Email notifications for new sessions and registrations
- 🔗 **Generate Invites** — Invite-based onboarding for new members
- ⏱️ **Session Timer** — Built-in rotation timer with WebSocket real-time updates

### For Members
- 🔐 **GitHub OAuth2 Login** — Secure authentication via GitHub
- 📋 **View Upcoming Sessions** — See all upcoming ensembles with registration status
- ✋ **Register/Deregister** — Accept, decline, or join as spectator
- 📹 **Access Recordings** — View session recordings after completion
- 🌍 **Time Zone Support** — Personal time zone preferences
- 📅 **Google Calendar** — One-click calendar invite generation
- 👤 **Profile Management** — Update name, email, and time zone

### Real-Time Features
- 🔄 **WebSocket Timer** — Live countdown timer with role rotation
- 🎵 **Audio Cues** — Sound notifications for timer events
- 📡 **External Display** — Sync with Conductor external timer display
- 🎯 **HTMX Integration** — Dynamic UI updates without full page reloads

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        ADAPTERS (IN)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │  Web / HTTP  │  │   OAuth2     │  │    WebSocket        │ │
│  │  Thymeleaf   │  │   GitHub     │  │    Timer UI         │ │
│  │  HTMX        │  │   Login      │  │    Real-time        │ │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬──────────┘ │
└─────────┼─────────────────┼─────────────────────┼────────────┘
          │                 │                     │
          ▼                 ▼                     ▼
┌──────────────────────────────────────────────────────────────┐
│                     APPLICATION LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │EnsembleService│ │MemberService │  │EnsembleTimerHolder  │ │
│  │  Scheduling  │  │  Management  │  │  Timer Lifecycle    │ │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬──────────┘ │
└─────────┼─────────────────┼─────────────────────┼────────────┘
          │                 │                     │
          ▼                 ▼                     ▼
┌──────────────────────────────────────────────────────────────┐
│                        DOMAIN LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │   Ensemble   │  │    Member    │  │   EnsembleTimer     │ │
│  │  (Aggregate) │  │  (Aggregate) │  │   CountdownTimer    │ │
│  │  State Mach. │  │  Identity    │  │   Rotation          │ │
│  └──────────────┘  └──────────────┘  └─────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
          ▲                 ▲                     ▲
          │                 │                     │
┌─────────┼─────────────────┼─────────────────────┼──────────┐
│         │              ADAPTERS (OUT)                      │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌────────┴──────────┐ │
│  │  PostgreSQL  │  │    Email     │  │   Zoom API        │ │
│  │  Spring Data │  │    Brevo     │  │   Conductor WS    │ │
│  │  JDBC        │  │    Pushover  │  │   GitHub API      │ │
│  └──────────────┘  └──────────────┘  └───────────────────┘ │
└────────────────────────────────────────────────────────────┘
```

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Hexagonal Architecture** | Clear separation of domain, application, and infrastructure concerns |
| **Spring Data JDBC** (not JPA) | Explicit aggregate design, better alignment with DDD |
| **Server-Side Rendering** | Thymeleaf + HTMX for progressive enhancement without SPA complexity |
| **WebSocket for Timer** | Real-time updates for all connected clients during sessions |
| **Flyway Migrations** | Version-controlled, reproducible database schema evolution |
| **Testcontainers** | Integration tests with real PostgreSQL for confidence |

---

## Technologies

### Core Framework
- **Java 25** — Latest LTS with virtual threads support
- **Spring Boot 3.5** — Application framework
- **Spring Security 6** — OAuth2 + role-based authorization
- **Spring Data JDBC** — Data persistence
- **Spring WebSocket** — Real-time communication

### Frontend
- **Thymeleaf** — Server-side templating
- **Tailwind CSS** — Utility-first CSS framework
- **HTMX** — Dynamic HTML updates
- **Font Awesome** — Icons

### Database & Migrations
- **PostgreSQL 15** — Primary database
- **Flyway** — Schema migrations (13 versions)

### External Integrations
- **GitHub OAuth2** — Authentication
- **Zoom API** — Video conference scheduling
- **Brevo (Sendinblue)** — Transactional email
- **Pushover** — Push notifications
- **Conductor** — External timer display sync

### DevOps & Quality
- **Docker** + **Docker Compose** — Containerization
- **GitHub Actions** — CI/CD pipeline
- **Dependabot** — Automated dependency updates
- **ArchUnit** — Architecture compliance testing
- **Qodana** — Static code analysis
- **Micrometer + Prometheus** — Observability

---

## Project Structure

```
ensembler/
├── .github/
│   ├── workflows/maven.yml      # CI/CD pipeline
│   └── dependabot.yml           # Dependency automation
├── docs/                         # Architecture documentation
│   ├── UbiquitousLanguage.md
│   ├── API for Conductor.txt
│   ├── MobTimeWebSocket.md
│   ├── invite-sign-up-flow.puml
│   ├── login-flow.puml
│   ├── project.md
│   └── tasks.md
├── src/
│   ├── main/java/com/jitterted/mobreg/
│   │   ├── domain/              # Domain layer (Aggregates, VOs, Events)
│   │   ├── application/           # Application layer (Services, Ports)
│   │   │   └── port/            # Port interfaces
│   │   ├── adapter/
│   │   │   ├── in/web/          # Incoming adapters (Controllers, Forms)
│   │   │   │   ├── admin/       # Admin controllers & views
│   │   │   │   └── member/      # Member controllers & views
│   │   │   └── out/             # Outgoing adapters (Repos, APIs)
│   │   │       ├── jdbc/        # Database adapters
│   │   │       ├── email/       # Email notification adapters
│   │   │       ├── zoom/        # Zoom API adapter
│   │   │       ├── conductor/   # Conductor sync adapter
│   │   │       ├── websocket/   # WebSocket broadcaster
│   │   │       ├── membership/  # GitHub membership adapter
│   │   │       └── notification/pushover/  # Push notifications
│   │   ├── EnsemblerApplication.java
│   │   ├── EnsemblerConfiguration.java
│   │   ├── WebSecurityConfig.java
│   │   ├── WebSocketConfiguration.java
│   │   └── VersionInfoContributor.java
│   ├── main/resources/
│   │   ├── application*.properties  # Environment configs
│   │   ├── db/migration/        # Flyway migrations (V1-V13)
│   │   ├── templates/           # Thymeleaf HTML templates
│   │   └── static/              # CSS, JS, audio, favicon
│   └── test/java/...            # Comprehensive test suite
├── Dockerfile                    # Multi-stage Docker build
├── compose.yml                   # Docker Compose (PostgreSQL)
├── pom.xml                       # Maven configuration
├── qodana.yaml                   # Static analysis config
└── README.md                     # This file
```

---

## Prerequisites

- **JDK 25** or later
- **Maven 3.9+** (or use `./mvnw` wrapper)
- **Docker & Docker Compose** (for local PostgreSQL)
- **GitHub OAuth App** (for authentication)
- **Zoom App** (for video conference integration — optional)
- **Brevo API Key** (for email notifications — optional)

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/AhmedDhibi1/REPR-SYS.git
cd REPR-SYS
```

### 2. Set Up Local Environment

Create a `application-local.properties` override (or use environment variables):

```bash
# GitHub OAuth2 (create at https://github.com/settings/developers)
export GITHUB_OAUTH2_LOCAL_CLIENTID="your-client-id"
export GITHUB_OAUTH2_LOCAL_CLIENTSECRET="your-client-secret"

# Optional: Zoom integration
export ZOOM_CLIENT_ID="your-zoom-client-id"
export ZOOM_CLIENT_SECRET="your-zoom-client-secret"
export ZOOM_ACCOUNT_ID="your-zoom-account-id"

# Optional: Brevo email
export BREVO_API_KEY="your-brevo-api-key"
```

### 3. Start PostgreSQL

```bash
docker compose up -d
```

This starts a PostgreSQL 15 container with:
- Database: `postgres`
- User: `postgres`
- Password: `password`
- Port: `5432`

---

## Configuration

### Profile-Based Configuration

| Profile | Purpose | File |
|---------|---------|------|
| `local` | Local development with Docker PostgreSQL | `application-local.properties` |
| `postgresql` | Generic PostgreSQL config | `application-postgresql.properties` |
| `railway` | Production deployment on Railway | `application-railway.properties` |

### Key Properties

```properties
# Active profile
spring.profiles.active=local

# GitHub OAuth2
spring.security.oauth2.client.registration.github.clientId=${github.oauth2.local.clientId}
spring.security.oauth2.client.registration.github.clientSecret=${github.oauth2.local.clientSecret}

# Database
spring.datasource.url=jdbc:postgresql:postgres
spring.datasource.username=postgres
spring.datasource.password=password

# Feature toggles
ensembler.repository.inmemory=false  # Use JDBC repositories

# Management endpoints
management.endpoints.web.exposure.include=health,info,metrics,prometheus
```

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_OAUTH2_CLIENTID` | ✅ | GitHub OAuth2 Client ID |
| `GITHUB_OAUTH2_CLIENTSECRET` | ✅ | GitHub OAuth2 Client Secret |
| `PGHOST` | ⚠️ (Railway) | PostgreSQL host |
| `PGPORT` | ⚠️ (Railway) | PostgreSQL port |
| `PGDATABASE` | ⚠️ (Railway) | PostgreSQL database |
| `PGUSER` | ⚠️ (Railway) | PostgreSQL username |
| `PGPASSWORD` | ⚠️ (Railway) | PostgreSQL password |
| `ZOOM_CLIENT_ID` | ❌ | Zoom OAuth Client ID |
| `ZOOM_CLIENT_SECRET` | ❌ | Zoom OAuth Client Secret |
| `ZOOM_ACCOUNT_ID` | ❌ | Zoom Account ID |
| `BREVO_API_KEY` | ❌ | Brevo API Key for email |
| `GITHUB_OAUTH` | ❌ | GitHub Personal Access Token |

---

## Running Locally

### With Maven Wrapper

```bash
# Start PostgreSQL first
docker compose up -d

# Run the application
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

The application will be available at: **http://localhost:8080**

### As a JAR

```bash
./mvnw clean package -DskipTests
java -jar target/ensembler-*.jar --spring.profiles.active=local
```

### With In-Memory Repositories (No DB)

```bash
./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-Densembler.repository.inmemory=true"
```

---

## Running with Docker

### Build and Run

```bash
# Build the image
docker build -t ensembler .

# Run with linked PostgreSQL
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=local \
  -e GITHUB_OAUTH2_CLIENTID=... \
  -e GITHUB_OAUTH2_CLIENTSECRET=... \
  --network host \
  ensembler
```

### Docker Compose (Full Stack)

```bash
docker compose up --build
```

---

## Database

### Schema Evolution

The database schema is managed by **Flyway** with 13 migrations:

| Version | Description |
|---------|-------------|
| V1 | Initial `huddle_entity`, `member_entity`, `registered_members` |
| V2 | Add `zoom_meeting_link` |
| V3 | Add `completed` flag and `recording_link` |
| V4 | Add `email` to members |
| V5 | Rename `registered_members` → `accepted_member` |
| V6 | Add `declined_member` table |
| V7 | Rename `huddle` → `ensemble` |
| V8 | Add `time_zone` to members |
| V9 | Rename tables to plural (`ensembles`, `members`) |
| V10 | Convert `is_completed` boolean → `state` enum |
| V11 | Full conference details (meeting ID, start URL) |
| V12 | Invite table + unique GitHub username constraint |
| V13 | Spectator member table |

### Entity Relationships

```
ensembles 1--* accepted_member *--1 members
ensembles 1--* declined_member *--1 members
ensembles 1--* spectator_member *--1 members
members 1--* roles (set)
invites (standalone)
```

---

## API Overview

### Public Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Landing page |
| GET | `/login` | OAuth2 login |

### Member Endpoints (ROLE_MEMBER)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/member/register` | View upcoming/past ensembles |
| POST | `/member/accept` | Register as participant |
| POST | `/member/decline` | Decline participation |
| POST | `/member/join-as-spectator` | Join as spectator |
| GET | `/member/profile` | View/edit profile |
| GET | `/member/timer-view/{id}` | View ensemble timer |
| POST | `/member/start-timer/{id}` | Start timer |
| POST | `/member/pause-timer/{id}` | Pause timer |
| POST | `/member/resume-timer/{id}` | Resume timer |
| POST | `/member/reset-timer/{id}` | Reset timer |
| POST | `/member/rotate-timer/{id}` | Rotate roles |

### Admin Endpoints (ROLE_ADMIN)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/admin/dashboard` | Admin dashboard |
| GET | `/admin/ensemble/{id}` | Ensemble detail |
| POST | `/admin/schedule` | Schedule new ensemble |
| POST | `/admin/ensemble/{id}` | Edit ensemble |
| POST | `/admin/ensemble/{id}/complete` | Complete ensemble |
| POST | `/admin/ensemble/{id}/cancel` | Cancel ensemble |
| POST | `/admin/register` | Manual participant registration |
| POST | `/admin/notify/{id}` | Trigger notifications |
| POST | `/admin/create-timer/{id}` | Create timer |
| POST | `/admin/delete-timer/{id}` | Delete timer |
| GET | `/admin/members` | Member management |
| GET | `/admin/invites` | Invite management |

### WebSocket
| Endpoint | Description |
|----------|-------------|
| `/member/ws/timer` | Real-time timer updates |

### Actuator
| Endpoint | Description |
|----------|-------------|
| `/actuator/health` | Health check |
| `/actuator/info` | Build info |
| `/actuator/metrics` | Application metrics |
| `/actuator/prometheus` | Prometheus scrape endpoint |

---

## Testing

### Run All Tests

```bash
./mvnw verify
```

### Run Unit Tests Only

```bash
./mvnw test
```

### Run Integration Tests

Integration tests use **Testcontainers** to spin up a real PostgreSQL instance:

```bash
./mvnw verify -P integration-tests
```

### Run Manual/External API Tests

Tests tagged with `@Tag("manual")` require real API credentials:

```bash
./mvnw test -Dtest="*ManualTest,*ApiTest"
```

### Architecture Tests

```bash
./mvnw test -Dtest=HexagonalArchitectureTest
```

### Test Coverage

| Layer | Test Strategy |
|-------|---------------|
| Domain | Pure unit tests, no Spring context |
| Application | Service tests with in-memory repositories |
| Adapters | Integration tests with Testcontainers |
| Web | Spring MVC Mock tests |
| Architecture | ArchUnit dependency rules |

---

## Build

### Development Build

```bash
./mvnw clean package
```

### Production Build

```bash
./mvnw clean package -DskipTests
```

### Native Image (GraalVM)

```bash
./mvnw native:compile -Pnative
```

---

## Development Workflow

### Git Workflow

1. Create a feature branch from `main`
2. Make changes with conventional commits
3. Ensure tests pass: `./mvnw verify`
4. Submit a pull request
5. Merge after CI passes

### Conventional Commits

```
feat:     new feature
docs:     documentation only
style:    formatting, missing semicolons, etc.
refactor: code change that neither fixes a bug nor adds a feature
test:     adding or refactoring tests
chore:    build process or auxiliary tool changes
perf:     performance improvements
ci:       CI/CD configuration
```

### Database Migrations

When modifying the schema:

1. Create a new Flyway migration: `V{next}__descriptive_name.sql`
2. Test with `./mvnw flyway:migrate`
3. Never modify existing migration files

---

## Future Improvements

- [ ] **Group Management** — Ensembles organized into groups with tiered membership
- [ ] **Discord Integration** — Direct message notifications via Discord bot
- [ ] **Calendar Integration** — Native Google/Outlook calendar API integration
- [ ] **Self-Service Sign-Up** — Bot-assisted onboarding workflow
- [ ] **MobTi.me Integration** — WebSocket sync with mobti.me timer
- [ ] **Analytics Dashboard** — Participation metrics and reporting
- [ ] **Mobile Responsiveness** — Enhanced mobile UI
- [ ] **Dark Mode** — Theme switching support

"""
