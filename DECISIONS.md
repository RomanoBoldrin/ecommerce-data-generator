# Merchant Pulse Data Generator — Technical Decision Log

This log records established architectural and technical decisions for the Merchant Pulse synthetic data generator project, distinguishing accepted decisions from proposed ones.

---

## Accepted Decisions

### DEC-001: Standalone Python Project Architecture
- **Status**: Accepted
- **Context**: Merchant Pulse requires a synthetic data generator for demonstration, analytical dashboard verification, and machine learning experimentation.
- **Decision**: The generator is developed as an independent, standalone Python project, completely decoupled from the Next.js application codebase.
- **Rationale**: Keeps generation logic self-contained, lightweight, and easily testable without requiring node/frontend toolchains.

### DEC-002: Dependency and Virtual Environment Management via `uv`
- **Status**: Accepted
- **Context**: Fast, reliable, and reproducible Python packaging is required.
- **Decision**: Use `uv` for Python virtual environment management, dependency resolution, lockfile maintenance (`uv.lock`), and execution.
- **Rationale**: Ensures deterministic environments, fast resolution, and strict Python 3.12+ support per project specification.

### DEC-003: Python 3.12+ Minimum Runtime
- **Status**: Accepted
- **Context**: The project brief specifies modern Python standards and dependencies.
- **Decision**: Define `requires-python = ">=3.12"` in `pyproject.toml`.
- **Rationale**: Standardizes modern typing features, performance improvements, and ecosystem compatibility.

### DEC-004: Direct PostgreSQL Persistence using `psycopg`
- **Status**: Accepted
- **Context**: Synthetic data must be loaded into the existing PostgreSQL database (`pulse_dev_db`).
- **Decision**: Connect directly to PostgreSQL using `psycopg` (binary) and execute efficient batch inserts without introducing an ORM or Prisma Client for Python.
- **Rationale**: Avoids ORM overhead, minimizes dependencies, enables fast bulk-loading (`COPY` / `executemany`), and respects the database schema as the single source of truth.

### DEC-005: Read-Only Database Schema Contract
- **Status**: Accepted
- **Context**: Schema and migrations are maintained under `reference/` (`reference/schema.prisma` and `reference/migrations/`).
- **Decision**: Treat `reference/` as strictly read-only. The Python generator does not alter tables, create new tables, execute DDL, or run migrations.
- **Rationale**: Preserves schema stability across the entire Merchant Pulse system.

### DEC-006: Centralized Configuration Strategy
- **Status**: Accepted
- **Context**: Database connection details and simulation parameters must not be scattered throughout the codebase.
- **Decision**: Consolidate database URLs (defaulting to `postgresql://pulse_user:local_password@127.0.0.1:5432/pulse_dev_db`), random seeds, date ranges, and volume parameters into a single configuration module (`generator/config.py`).
- **Rationale**: Complies with the implementation brief requirement for a single centralized configuration location.

### DEC-007: UUID v7 Primary Key Generation
- **Status**: Accepted
- **Context**: The Prisma schema defines `@default(uuid(7))` for primary keys across all models.
- **Decision**: Generate RFC 9562-compliant UUID v7 identifiers in Python using a maintained library (`uuid-utils`).
- **Rationale**: Generates time-ordered UUIDs matching Prisma's UUID v7 convention, improving B-tree indexing performance in PostgreSQL.

### DEC-008: Deterministic Programmatic Generation (No LLM APIs)
- **Status**: Accepted
- **Context**: Synthetic catalog items and behavioral events must be plausible, consistent, and reproducible.
- **Decision**: Generate catalogs, sessions, and events using deterministic rule-based probabilistic algorithms with configurable random seeds, without calling external LLM APIs.
- **Rationale**: Guarantees zero runtime cost, offline reproducibility, fast generation times, and compliance with the implementation brief.

### DEC-009: Strict Five-Minute ML Landmark Formulation
- **Status**: Accepted
- **Context**: Session conversion prediction must be realistic and free from data leakage.
- **Decision**: Formulate ML features using only session events occurring within the first 5 minutes of session duration. Sessions that convert or end at or before 5 minutes are excluded from prediction examples.
- **Rationale**: Strictly prevents future leakage and models realistic early-session purchase intent.

### DEC-010: Simple Continuous IoT Simulator Architecture
- **Status**: Accepted
- **Context**: Warehouse environmental monitoring must be demonstrated via the `IoTDevice` and `IoTReading` tables.
- **Decision**: Implement a lightweight, single-process continuous loop that periodically writes temperature, humidity, and energy readings with realistic fluctuations and anomalies, shutting down gracefully on `SIGINT`/`SIGTERM`.
- **Rationale**: Fulfills dashboard demonstration requirements without requiring MQTT brokers, daemon managers, or complex streaming infrastructure.

---

## Proposed Decisions

*(No pending proposals at this time.)*

