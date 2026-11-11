# Merchant Pulse — Implementation Milestones & Roadmap

This document defines the complete sequential implementation roadmap for the Merchant Pulse synthetic data generator, based on `docs/IMPLEMENTATION.md` and the database contract under `reference/`.

---

## Milestone 1: Project Foundation and Configuration

- **Objective**: Establish the Python 3.12+ project structure, package management, centralized configuration, and UUID v7 utility.
- **Dependencies & Prerequisites**: None.
- **Scope & Expected Deliverables**:
  - Python configuration using `uv` (updating `pyproject.toml` and generating `uv.lock`).
  - Strict Python 3.12+ compatibility check.
  - Centralized configuration module (`config/` or `generator/config.py`) managing database connection string (`postgresql://pulse_user:local_password@127.0.0.1:5432/pulse_dev_db`), seed parameters, date windows (two-year range), and volume targets.
  - Reproducible random seed generator configuration.
  - RFC 9562-compliant UUID v7 utility using a maintained implementation (`uuid-utils`).
  - Verified `.gitignore` excluding `.venv`, caches, datasets, and local credentials.
  - Unit tests covering configuration loading, seed determinism, and UUID v7 structure.
- **Explicit Exclusions**:
  - No database connectivity or persistence logic.
  - No simulation, catalog generation, ML, or IoT logic.
- **Acceptance Criteria**:
  1. `uv.lock` is present and synchronized with `pyproject.toml`.
  2. Configuration loads default parameters successfully and allows override without modifying source files.
  3. Database URL is defined strictly in the centralized configuration module.
  4. UUID utility produces valid UUID v7 instances (version 7 bits verified, chronologically orderable).
  5. Unit tests for configuration and UUID utility pass via `uv run pytest`.
- **Verification Commands**:
  - `uv run pytest tests/test_config.py tests/test_uuid.py`
  - `python3 -c "import generator; print('Package import successful')"`
- **Expected Git Handoff**:
  - Milestone commit containing configuration, UUID utility, tests, and documentation update in `PROGRESS.md`.

---

## Milestone 2: Database Layer and Persistence Foundation

- **Objective**: Implement a direct, efficient PostgreSQL persistence layer adhering strictly to the existing schema and supporting provenance tracking via `DataSource` and `DataImport`.
- **Dependencies & Prerequisites**: Milestone 1.
- **Scope & Expected Deliverables**:
  - PostgreSQL connection handling using `psycopg` (binary) referencing the centralized configuration.
  - Transaction management and batch insertion helpers (`executemany` or PostgreSQL `COPY`).
  - Minimal schema-compatible provenance implementation:
    - Verification/upsert of single `DataSource` record (`type='SYNTHETIC'`, `name='Merchant Pulse Python Generator'`).
    - Helper to create a `DataImport` record (`status='IN_PROGRESS'`, format tracking, record counters, completion timestamping).
  - Test verifying connection, table existence, schema compatibility, transaction commit, rollback on error, and foreign key enforcement.
- **Explicit Exclusions**:
  - No schema modifications, DDL execution, table creation, or Prisma migrations.
  - No complex import orchestration or multi-tenant abstraction.
  - No large-scale data generation.
- **Acceptance Criteria**:
  1. Connection to `postgresql://pulse_user:local_password@127.0.0.1:5432/pulse_dev_db` succeeds.
  2. Verified query confirms existence of required tables (`categories`, `products`, `visitor_sessions`, `navigation_events`, `orders`, `order_items`, `data_sources`, `data_imports`, `iot_devices`, `iot_readings`).
  3. Safe insertion and retrieval of test records for `DataSource` and `DataImport`.
  4. Transaction rollback tested and verified without leaking orphan records.
  5. Read-only integrity of `reference/` verified.
- **Verification Commands**:
  - `uv run pytest tests/test_db.py`
- **Expected Git Handoff**:
  - Milestone commit containing database module, provenance helpers, tests, and updated `PROGRESS.md`.

---

## Milestone 3: Synthetic Catalog Generation

- **Objective**: Programmatically generate a realistic, reproducible electronics and accessories catalog adhering to the `Category` and `Product` models.
- **Dependencies & Prerequisites**: Milestone 1, Milestone 2.
- **Scope & Expected Deliverables**:
  - Programmatic catalog generator creating ~10 categories and ~100–150 products.
  - Meaningful electronics taxonomy (e.g., Audio, Computing, Wearables, Accessories, Mobile, Power & Cables).
  - Product attributes: unique SKU, plausible name, description, active status, baseline price (`Decimal(12, 2)`), and demand/popularity weight.
  - Persistence logic saving categories and products to PostgreSQL using UUID v7 primary keys.
  - Decoupled design allowing catalogs to be generated or loaded independently of session simulations.
  - Catalog export capability (CSV/JSON/DB).
  - Unit tests verifying SKU uniqueness, category relations, valid price boundaries, and seed reproducibility.
- **Explicit Exclusions**:
  - No external LLM API usage.
  - No inventory/stock management logic beyond optional initial `stockSnapshot`.
- **Acceptance Criteria**:
  1. Generates 8–12 categories and 100–150 products deterministically for a given seed.
  2. All SKUs are globally unique.
  3. Every product has a valid `categoryId` referencing an existing category.
  4. Product prices are strictly positive `Decimal` values with 2 decimal places.
  5. Catalog persists cleanly to PostgreSQL without violating schema constraints.
- **Verification Commands**:
  - `uv run pytest tests/test_catalog.py`
  - `uv run python -m generator.catalog --dry-run`
- **Expected Git Handoff**:
  - Milestone commit containing catalog generator, persistence logic, unit tests, and updated `PROGRESS.md`.

---

## Milestone 4: E-commerce Behavioral Simulation

- **Objective**: Simulate coherent e-commerce visitor sessions, navigation journeys, and order transactions across a two-year historical window.
- **Dependencies & Prerequisites**: Milestone 1, Milestone 2, Milestone 3.
- **Scope & Expected Deliverables**:
  - Behavioral simulation engine generating visitor sessions and chronological event streams over two years.
  - Funnel stage simulation (`SESSION_START` $\to$ `PRODUCT_VIEW` $\to$ `ADD_TO_CART` $\to$ `CHECKOUT_START` $\to$ `ORDER_COMPLETED`).
  - Pseudonymous visitor model: `visitor_hash` allowing returning visitors vs. new visitors.
  - Plausible variation: device types (mobile, desktop, tablet), traffic sources (organic, direct, paid, email), short/long session durations.
  - Temporal patterns: diurnal hourly curves, weekday vs. weekend shifts, seasonal trends, and holiday/promotional spikes.
  - Conversion logic: order generation emerging probabilistically from checkout sessions.
  - Strict financial invariant: `Order.total = Order.subtotal - Order.discount + Order.shippingAmount`.
  - Item pricing invariant: `OrderItem.unitPrice` set to historical price, `quantity >= 1`, line discounts respected.
  - Explicit five-minute landmark tracking for downstream ML compatibility.
  - Unit tests verifying timestamp chronology, event sequence validity, funnel consistency, and order arithmetic.
- **Explicit Exclusions**:
  - No customer personal data (PII), passwords, or real user accounts.
  - No payment gateway, credit card validation, or shipping fulfillment simulation.
  - No complex digital twin or multi-agent behavioral frameworks.
- **Acceptance Criteria**:
  1. Navigation events within a session have strictly non-decreasing timestamps (`occurredAt >= session.startedAt` and `occurredAt <= session.endedAt`).
  2. Every `ORDER_COMPLETED` event corresponds to an order record.
  3. Order totals strictly satisfy `total = subtotal - discount + shippingAmount`.
  4. Distinct session counts across funnel stages are monotonically non-increasing (Sessions $\ge$ Views $\ge$ Cart Adds $\ge$ Checkout $\ge$ Orders).
  5. Timestamps use UTC instants while business dates align with `America/Sao_Paulo`.
- **Verification Commands**:
  - `uv run pytest tests/test_simulation.py`
- **Expected Git Handoff**:
  - Milestone commit containing behavioral simulation engine, models, unit tests, and updated `PROGRESS.md`.

---

## Milestone 5: Batch Persistence and Data Exports

- **Objective**: Stream and persist large-scale historical synthetic datasets (~300,000 sessions, millions of events) into PostgreSQL and export to CSV/Parquet without excessive memory consumption.
- **Dependencies & Prerequisites**: Milestone 1, Milestone 2, Milestone 3, Milestone 4.
- **Scope & Expected Deliverables**:
  - Chunked generation pipeline processing data in manageable time or session blocks.
  - Fast PostgreSQL batch persistence (`psycopg` copy/batch insert) preserving foreign key integrity across tables.
  - Linking of generated sessions, navigation events, and orders to a `DataImport` record.
  - Optional export pipeline generating structured CSV and/or Parquet files in `output/`.
  - Configurable volume flags (e.g., small test scale vs. full 300k session scale).
- **Explicit Exclusions**:
  - In-memory buffering of entire multi-year dataset.
  - Destructive database operations (dropping tables, truncating without isolation).
- **Acceptance Criteria**:
  1. Small-scale generation (e.g., 1,000 sessions) and persistence run cleanly and verify foreign keys in PostgreSQL.
  2. Pipeline runs within bounded memory limits (streaming/chunking verified).
  3. `DataImport` records accurate counts of `records_processed`, timestamps, and status `COMPLETED`.
  4. CSV/Parquet export generates valid files matching the database schema structure.
- **Verification Commands**:
  - `uv run pytest tests/test_persistence.py`
  - `uv run python -m generator.pipeline --sessions 1000 --export csv`
- **Expected Git Handoff**:
  - Milestone commit containing streaming pipeline, batch persistence, export logic, tests, and updated `PROGRESS.md`.

---

## Milestone 6: Data Validation and Profiling

- **Objective**: Implement automated data integrity validation and summary profiling to ensure data meets analytical and ML requirements.
- **Dependencies & Prerequisites**: Milestone 1 through Milestone 5.
- **Scope & Expected Deliverables**:
  - Automated validation suite checking:
    - Referential integrity (no orphaned events, items, or orders).
    - Uniqueness constraints (`external_session_id`, `deduplication_key`, `external_order_id`, `sku`).
    - Event sequences and chronological validity.
    - Mathematical invariants (`total = subtotal - discount + shippingAmount`).
    - Non-negative quantities and monetary values.
    - Funnel stage conversion rates and distinct-session funnel metrics.
    - Temporal distribution and absence of missing date gaps.
    - Five-minute landmark eligibility and ML target distribution.
  - Profiling report generator outputting summary metrics (record counts, conversion rates, average order value, category revenue distribution).
  - Clear exception/failure reporting when critical invariants are violated.
- **Explicit Exclusions**:
  - Interactive dashboard or web UI.
- **Acceptance Criteria**:
  1. Validator passes on validly generated synthetic data and fails predictably on corrupted test data.
  2. Report accurately computes distinct-session funnel metrics rather than raw event counts.
  3. Validation summary report is generated in text or markdown format.
- **Verification Commands**:
  - `uv run pytest tests/test_validation.py`
  - `uv run python -m generator.validate`
- **Expected Git Handoff**:
  - Milestone commit containing validation engine, profiling reporter, tests, and updated `PROGRESS.md`.

---

## Milestone 7: Machine Learning Conversion Pipeline

- **Objective**: Implement the session conversion prediction experiment using a Decision Tree Classifier with a strict five-minute landmark.
- **Dependencies & Prerequisites**: Milestone 1, Milestone 4, Milestone 5, Milestone 6.
- **Scope & Expected Deliverables**:
  - Feature extraction at the five-minute landmark after session start:
    - Exclude sessions ending or converting at or before 5 minutes.
    - Target: Completed purchase between 5 minutes and session end.
    - Input features: Product views, cart additions, elapsed time up to landmark (5 minutes), device type, traffic source.
    - Zero data leakage from events occurring after 5 minutes.
  - Chronological train/test split (e.g., first 18–20 months for training, remaining months for testing).
  - Baseline comparison (majority class / dummy classifier).
  - Decision Tree Classifier training with constrained depth (`max_depth`) and minimum leaf size to prevent overfitting.
  - Comprehensive metric evaluation: Accuracy, Precision, Recall, F1-Score, ROC-AUC.
  - Feature importance extraction and tree visualization (text summary or exported image).
- **Explicit Exclusions**:
  - Deep learning, complex ensembles, or additional unsolicited ML algorithms.
  - Live real-time scoring web APIs.
- **Acceptance Criteria**:
  1. Feature dataset strictly excludes post-landmark event data.
  2. Training and testing datasets are split chronologically without temporal overlap or future leakage.
  3. Model evaluation reports Accuracy, Precision, Recall, F1, and ROC-AUC.
  4. Decision Tree outperforms the dummy baseline.
  5. Tree complexity is constrained and training vs. test metrics are compared to detect overfitting.
- **Verification Commands**:
  - `uv run pytest tests/test_ml.py`
  - `uv run python -m generator.ml --evaluate`
- **Expected Git Handoff**:
  - Milestone commit containing ML pipeline, evaluation module, test suite, and updated `PROGRESS.md`.

---

## Milestone 8: IoT Telemetry Simulator

- **Objective**: Implement a simple, continuous simulator for warehouse environmental monitoring devices persisting to `IoTDevice` and `IoTReading`.
- **Dependencies & Prerequisites**: Milestone 1, Milestone 2.
- **Scope & Expected Deliverables**:
  - Configuration for a small set of warehouse monitoring devices (~5–10 devices with locations).
  - Device initialization/registration in `IoTDevice`.
  - Periodic reading generator producing:
    - `temperatureCelsius` (`Decimal(5, 2)`).
    - `relativeHumidity` (`Decimal(5, 2)`).
    - `energyConsumptionKw` (`Decimal(8, 3)`).
  - Realistic diurnal fluctuations, sensor noise, and occasional anomalous readings.
  - Continuous execution loop running at configurable intervals with clean shutdown on `SIGINT`/`SIGTERM`.
  - Persistence to `iot_readings` table respecting the unique constraint `(deviceId, recordedAt)`.
- **Explicit Exclusions**:
  - MQTT, message brokers, Kafka, daemon managers, or hardware interfaces.
  - Complex multi-process streaming infrastructure.
- **Acceptance Criteria**:
  1. Registers devices in `iot_devices` if not already present.
  2. Generates plausible readings within defined bounds (e.g. humidity 0–100%).
  3. Persists readings with UTC timestamps matching `timestamptz(3)`.
  4. Handles `SIGINT` (Ctrl+C) gracefully, flushing pending writes and terminating without error.
- **Verification Commands**:
  - `uv run pytest tests/test_iot.py`
  - `uv run python -m generator.iot --interval 1 --max-readings 5`
- **Expected Git Handoff**:
  - Milestone commit containing IoT simulation loop, persistence, graceful shutdown test, and updated `PROGRESS.md`.

---

## Milestone 9: CLI, Documentation, and Integration

- **Objective**: Unify project capabilities under a clean, minimal CLI and complete end-to-end documentation and verification.
- **Dependencies & Prerequisites**: Milestones 1 through 8.
- **Scope & Expected Deliverables**:
  - Minimal CLI with four subcommands:
    - `generate`: Generates catalog and historical e-commerce sessions/orders.
    - `train`: Runs the ML feature extraction, training, and evaluation.
    - `iot`: Launches the continuous IoT telemetry generator.
    - `validate`: Runs data integrity validation on persisted records.
  - Comprehensive `README.md` covering:
    - Project objective and architecture.
    - Setup instructions using `uv`.
    - PostgreSQL configuration and connection string.
    - Execution guides for each CLI command.
    - Schema alignment and validation summary.
  - End-to-end integration test verifying the entire workflow from initial configuration to persistence and validation.
- **Explicit Exclusions**:
  - Elaborate CLI frameworks, interactive GUIs, or dashboard web services.
- **Acceptance Criteria**:
  1. CLI commands `generate`, `train`, `iot`, and `validate` are directly runnable via `uv run pulse-generator <command>` (or `python -m generator.cli <command>`).
  2. End-to-end test executes a complete scaled workflow without error.
  3. `README.md` enables a new developer to set up and run the system from scratch.
  4. Final repository status is clean, tested, and documented.
- **Verification Commands**:
  - `uv run pytest`
  - `uv run python -m generator.cli --help`
- **Expected Git Handoff**:
  - Milestone commit containing CLI module, `README.md`, integration tests, and finalized `PROGRESS.md`.

