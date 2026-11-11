# Merchant Pulse — Synthetic Data Generator

## Implementation Brief for Coding Agent

### 1. Objective

Develop a standalone Python project that automatically generates synthetic data for Merchant Pulse, a simulated e-commerce analytics application.

The project starts as a new, blank repository. It has no existing Python implementation or real-world e-commerce dataset. All historical e-commerce data must therefore be generated programmatically.

The generator must produce plausible, internally consistent data over a two-year period, supporting:

- The analytical metrics and visualizations planned for the Merchant Pulse dashboard.
- At least one meaningful machine-learning application.
- A simulated IoT telemetry ingestion demonstration.

Prioritize implementation simplicity, reproducibility, and analytical usefulness over reproducing every aspect of a production e-commerce platform.

### 2. Project environment

Use **uv** for Python version management, dependency management, virtual environments, and lockfile generation.

Project requirements:

- Python 3.12 or newer.
- Define the supported Python version in `pyproject.toml`.
- Maintain dependencies in `pyproject.toml`.
- Commit the generated `uv.lock`.
- Use a project-local virtual environment managed by uv.
- Provide clear setup and execution instructions.

Create the project structure, modules, configuration, tests, and documentation from scratch.

### 3. Database schema and reference materials

The project will be provided with reference files describing the existing PostgreSQL database:

- `reference/schema.prisma`
- `reference/migrations/` containing the existing database migration SQL files, where available.

The Prisma schema contains these relevant models:

- Category
- Product
- VisitorSession
- NavigationEvent
- Order
- OrderItem
- DataSource
- DataImport
- IoTDevice
- IoTReading

The database uses PostgreSQL, UUID identifiers, UTC timestamps, and America/Sao_Paulo business dates.

Treat the supplied schema and migrations as the database contract.

**Schema stability:**

- Do not modify the supplied schema or migrations.
- Do not independently create or modify database tables.
- If a concrete requirement cannot be represented using the existing models, document the incompatibility and propose the smallest necessary schema change before proceeding.
- Treat files under `reference/` as read-only.

### 4. Synthetic e-commerce environment

Simulate an imaginary electronics and accessories retailer.

Initial configuration:

- Two years of historical activity.
- Approximately 10 product categories.
- Approximately 100–150 products.
- Configurable session volume, with an initial final-dataset target around 300,000 sessions.
- Millions of navigation events where appropriate.
- Configurable random seed.

Product catalogs must be generated programmatically, without requiring external LLM APIs.

Products should have different popularity, pricing, and demand characteristics.

Keep catalog generation separate from behavioral simulation so catalogs can be reused across runs.

### 5. Behavioral simulation

Generate coherent visitor sessions and navigation events.

Supported journey:

1. Session starts.
2. Visitor views products.
3. Visitor adds products to cart.
4. Visitor starts checkout.
5. Visitor may complete a purchase.

Not all sessions should follow the complete journey.

Simulate different visitor behaviors, engagement levels, price sensitivities, and purchase intentions.

Purchases must emerge probabilistically from session behavior rather than being assigned independently.

Include:

- Short and long sessions.
- Returning visitors.
- Cart abandonment.
- Checkout abandonment.
- Successful purchases.
- Different device types and acquisition channels.
- Product popularity differences.

Introduce realistic temporal variation, including weekday patterns, seasonal changes, promotions, and long-term trends.

Ensure that all event sequences and monetary values are internally consistent.

### 6. Machine Learning — Session Conversion Prediction

Implement a supervised binary classification model to predict whether an e-commerce visitor session will result in a purchase.

**Algorithm:** Decision Tree Classifier (scikit-learn).

**Input features:**

- Number of product views.
- Number of cart additions.
- Session duration up to prediction time.
- Device type.
- Traffic source.

**Target:** Whether the session subsequently results in a completed purchase.

**Prediction landmark:** Use five minutes after session start as the prediction point.

Generate one prediction example for each session that is still active and has not completed a purchase at the five-minute landmark.

Calculate input features using only information available during those first five minutes.

Exclude sessions that ended or completed a purchase at or before the prediction landmark. The target indicates whether the session completes a purchase after the landmark and before the session ends.

**Implementation requirements:**

- Extract features from simulated session events.
- Ensure input features contain no information from events occurring after the prediction landmark.
- Use elapsed time up to the prediction landmark rather than the final session duration.
- Split data chronologically into training and testing sets.
- Train and evaluate the classifier.
- Report accuracy, precision, recall, F1-score and ROC-AUC.
- Visualize the decision tree or feature importance.
- Avoid target leakage and assess overfitting.
- Constrain tree depth and minimum leaf size to reduce overfitting.
- Compare training and test performance.
- Evaluate predicted probabilities using ROC-AUC.
- Include a simple baseline for comparison.

The model should be simple, reproducible, and easy to explain during the project presentation.

### 7. IoT simulation

Implement a separate, continuously running telemetry simulator.

Simulate a small number of warehouse environmental monitoring devices.

Each device produces:

- Temperature.
- Relative humidity.
- Energy consumption.

Generate readings at configurable intervals, with normal fluctuations and occasional anomalies.

Persist readings using the existing IoTDevice and IoTReading models.

The simulator should support continuous ingestion and allow the future dashboard to display automatically refreshing telemetry.

Do not implement physical hardware integrations or complex streaming infrastructure.

### 8. Generation and export

Organize the generator into small, independent modules.

Support:

- Configurable generation parameters.
- Reproducible random seeds.
- Incremental generation and batch processing.
- CSV and/or Parquet export.
- Stable identifiers and foreign-key relationships.

Keep catalog generation separate from behavioral simulation so catalogs can be reused across runs.

The historical generation process should support large datasets without requiring all generated records to remain in memory simultaneously.

### 9. Dashboard analytical requirements

Generated data should support the planned Merchant Pulse dashboard, consisting of five main areas:

**Overview**

- Visits.
- Completed orders.
- Revenue.
- Average order value.
- Conversion rate.
- Cart abandonment.
- Summary funnel.
- Weekly conversion trend.

**Purchase funnel**

- Visits.
- Product views.
- Cart additions.
- Checkout starts.
- Completed purchases.
- Stage conversion rates and drop-offs.

**Products and categories**

- Product views.
- Cart additions.
- Orders.
- Product conversion.
- Product revenue.
- Revenue by category.

**Temporal evolution**

- Visits over time.
- Conversion over time.
- Orders and revenue over time.
- Daily, weekly, and monthly aggregation.

**Data considerations**

- Funnel stages must count distinct sessions, not raw event counts.
- Product and category metrics must be derived from actual events and orders.
- Revenue must be derived from order records and their statuses.
- Historical business dates must use America/Sao_Paulo.
- Do not generate artificial dashboard KPIs independently of the underlying records.

The generator is responsible for producing the data, not implementing the dashboard itself.

### 10. Validation

Implement automated checks for:

- Referential integrity.
- Duplicate records.
- Valid event sequences.
- Session and order timestamp consistency.
- Positive quantities and valid monetary values.
- Correct order totals.
- Plausible behavioral distributions.
- Conversion rates and funnel metrics.
- ML target distribution and suitability.

Generate a summary report with record counts, key distributions, and validation results.

### 11. Scope restrictions

Do not implement:

- Real customer accounts or personal information.
- Payment processing.
- Shipping logistics.
- Full inventory management.
- Customer segmentation.
- Product recommendation systems.
- Marketing campaign management.
- Multi-store or multi-tenant architecture.
- Complex IoT infrastructure.
- Additional ML models without a demonstrated requirement.
- A dashboard or frontend inside the Python project.

Do not introduce additional infrastructure or services unless a concrete requirement justifies them.

### 12. PostgreSQL persistence

Implement direct PostgreSQL persistence using a suitable Python database library, such as psycopg.

**Local development connection:**

`postgresql://pulse_user:local_password@127.0.0.1:5432/pulse_dev_db`

This connection is exclusively for local development.

The connection string may be hardcoded in one centralized configuration location. Do not scatter connection details throughout the code.

**Implementation requirements:**

- Create the database connection and persistence layer from scratch.
- Use the supplied Prisma schema and migrations to understand table definitions, relationships, constraints, indexes, enums, defaults, and data types.
- Generate records compatible with the existing PostgreSQL schema.
- Implement safe insertion, transaction handling, and batching for large datasets.
- Avoid destructive operations such as dropping tables or clearing existing records.
- Do not create or modify database tables or execute migrations.
- Support running the Python project directly on the host machine using `127.0.0.1:5432`.

The generator must be independently executable and connect to the supplied PostgreSQL database.

### 13. Expected deliverables

Produce:

1. A runnable standalone Python project.
2. A centralized configuration file.
3. A synthetic catalog generator.
4. An e-commerce behavioral simulation engine.
5. An IoT telemetry simulator.
6. A PostgreSQL persistence layer.
7. Optional CSV and/or Parquet exports.
8. Automated validation and profiling.
9. A session conversion ML training and evaluation pipeline.
10. Documentation explaining setup, execution, configuration, database integration, and generated datasets.

Provide a simple execution interface, preferably with separate commands for:

- Generating historical e-commerce data.
- Training and evaluating the conversion model.
- Running continuous IoT telemetry ingestion.

The implementation should be straightforward to run, test, and integrate with the future Merchant Pulse application.

### Guiding principle

Generate the minimum amount of complexity necessary to produce a believable e-commerce dataset, meaningful analytical results, and a defensible machine-learning experiment.

### Git workflow and version control

Use Git throughout the implementation. The repository is expected to be initialized as a Git repository before development begins.

**Requirements:**

- Check the repository status before making changes.
- If Git has not been initialized, initialize it.
- Create an appropriate `.gitignore` before committing project files.
- Never commit virtual environments, generated datasets, temporary files, local credentials, or other machine-specific artifacts.
- Commit changes incrementally as implementation progresses.
- Organize commits around logical, independently understandable milestones rather than accumulating all changes into one large commit.
- Use clear, descriptive commit messages.
- Ensure the project remains in a working state at each meaningful commit.
- Run relevant tests and validation before committing.
- Do not amend, rewrite, or discard existing commits without explicit authorization.
- Do not force-push or modify remote repository history.
- Do not commit incomplete or knowingly broken work merely to create a checkpoint.

Suggested implementation milestones:

1. Initial project structure, Python configuration, dependencies, and documentation.
2. PostgreSQL connection and persistence layer.
3. Synthetic catalog generation.
4. E-commerce behavioral simulation.
5. Historical data persistence and export.
6. Automated validation and profiling.
7. Session conversion ML pipeline.
8. IoT telemetry simulator.
9. Integration testing, documentation, and final cleanup.

These milestones are suggestions; combine or divide them when appropriate to preserve coherent commits.

At the end, provide a concise summary of implemented functionality, tests performed, and commits created.
