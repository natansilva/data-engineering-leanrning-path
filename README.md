# Data Engineering Learning Path

Roadmap developed to guide my mentoring program in Data Engineering

## Roadmap

### Environment Setup and Configuration

Installation and configuration of tools to facilitate daily development work.

#### Shell

* Installation and configuration of Zsh and Oh My Zsh
* Configuration management (.zshrc)
* Custom aliases and functions
* Plugins
  * [git](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)
  * [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
  * [zsh-autocomplete](https://github.com/marlonrichert/zsh-autocomplete)
  * [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
* Shell tools
  * git
  * [jq](https://jqlang.org)
  * [yq](https://github.com/mikefarah/yq)
  * [curl](https://curl.se)

#### Python

Installation and creation of a development environment that can use multiple Python versions and manage pip packages.

* Python installation
* Virtual environment management
  * [pyenv](https://github.com/pyenv/pyenv)
  * venv
* [uv](https://docs.astral.sh/uv/)
* [ruff](https://docs.astral.sh/ruff/)

### Lakehouse

Building a Data Lakehouse following the medallion architecture to organize data flow into three layers. Data files from all lakehouse layers will be stored in S3-compatible object storage through RustFS, allowing Apache Iceberg tables to manage metadata, partitioning, and time travel without relying on direct local disk storage as the primary persistence layer.

* **Bronze (Raw):** Stores raw data ingested from source systems without transformations, preserving original format and structure. This layer serves as the source of truth and supports reprocessing when needed.
* **Silver (Trusted):** Data goes through cleaning, deduplication, type enforcement, and standardization. At this stage, datasets have a defined schema and are ready to be combined and enriched. The following processing and validation steps will be applied:
  * Creation of `hash_key`: an MD5 or SHA-256 hash generated from deduplication key fields, used to identify and track duplicates across runs.
  * Creation of `processed_at`: pipeline execution timestamp indicating when the record was produced in the silver layer.
  * Deduplication: removal of duplicate records based on a dataset-specific set of key fields. Duplicate detection uses the deduplication hash described in the metadata above.
  * Timestamp standardization: all date and time fields are converted to the `UTC-3 (America/Sao_Paulo)` time zone and stored in ISO 8601 format.
* **Gold (Refined):** Layer with aggregated and modeled datasets for BI tools, dashboards, and AI/ML models. Business rules are applied here to create final analytical views.

#### Technologies Used

* [Apache Iceberg](https://iceberg.apache.org) — Used as the table storage format across all lakehouse layers. Tables will be created and managed with Iceberg, using partitioning to optimize queries and time travel for reprocessing and auditability. Table data and metadata files will be persisted in RustFS through its S3-compatible interface.
* [RustFS](https://rustfs.com/) — Used as the local S3-compatible object storage layer for the lakehouse. RustFS will store Iceberg data files for bronze, silver, and gold tables, replacing direct local disk persistence and providing an object storage interface closer to production-style lakehouse environments.
* [Apache Spark](https://spark.apache.org) — Used to run bronze-to-silver transformations, applying the defined cleaning, deduplication, timestamp standardization, and data enrichment rules.
* [Polars](https://pola.rs) with [PyIceberg](https://py.iceberg.apache.org) — Used as a local alternative to Spark during development and testing. Polars runs the same bronze-to-silver transformations locally, while PyIceberg handles reading and writing Iceberg tables.
* [SQLMesh](https://sqlmesh.readthedocs.io/en/stable/) with [DuckDB](https://duckdb.org) — Used to model and materialize gold-layer datasets. SQL models are versioned and tested with SQLMesh, while DuckDB serves as the local execution engine for analytical queries.
* [Project Nessie](https://projectnessie.org) — Used as the catalog service for Iceberg tables. Branches will be created to isolate development and validate changes before promoting them to production.

### Data Ingestion

In this stage, we will ingest data from three sources and store the raw outputs in the data lake (bronze layer). After ingestion, we will apply the defined data treatment and standardization rules before promoting datasets to the silver layer.

#### API Ingestion

* Ingestion of API data from [OpenF1](https://openf1.org/docs/#introduction)
* Ingestion of unstructured data from [openfootball](https://github.com/openfootball)
* Ingestion of CSV data from [Olympics Games](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results)

All ingestion pipelines will land the extracted data in Iceberg tables in the bronze layer, preserving source granularity for traceability and reprocessing.

#### Orchestration

* Local Airflow setup with Docker and/or Docker Compose.
* Creation of DAGs to control ingestion workflows for each source.
* Execution of processing steps to clean, deduplicate, and standardize data before loading into the silver layer.
* Scheduled and manual DAG runs to support both recurring loads and ad-hoc reprocessing.

### Data Modeling

In this stage, we will model trusted datasets into a dimensional model following a star schema approach for analytical consumption.

The gold layer will be organized with:

* **Fact tables** for business events and measurable metrics (for example, counts, durations, and aggregates).
* **Dimension tables** for descriptive business attributes (for example, team, driver, event, date, and location).
* **Defined grain and keys** to ensure consistency in joins, filtering, and KPI calculations.

### Infrastructure

In this stage, we will build a local Kubernetes environment and migrate the project components from Docker Compose to the cluster.

#### Local Kubernetes Cluster

* Provision a local Kubernetes cluster using [Minikube](https://minikube.sigs.k8s.io/docs/) (primary option).
* Optionally evaluate [kind](https://kind.sigs.k8s.io/) or [k3d](https://k3d.io/) for faster CI-like local environments.
* Standardize deployments with Helm charts and Kubernetes manifests.

#### Migration Scope

* [Apache Airflow](https://airflow.apache.org) — Deploy on Kubernetes (for example with the official Helm chart), including scheduler, webserver, triggerer, and workers.
* [Project Nessie](https://projectnessie.org) — Run as a Kubernetes service for catalog/versioning APIs.
* [Spark Operator](https://github.com/kubeflow/spark-operator) — Deploy on Kubernetes to manage Spark applications submitted as part of the ingestion process, enabling batch execution for extraction, transformation, and bronze-to-silver workloads.
* Spark workloads — Execute batch jobs on Kubernetes (for example with the Spark Operator) for bronze-to-silver processing at scale.
* Ingestion services — Run extract/load pipelines as Kubernetes CronJobs or Jobs orchestrated by Airflow.
* Object storage for Iceberg tables — Use [RustFS](https://rustfs.com/) as the local S3-compatible object storage service to persist bronze, silver, and gold data files managed by Apache Iceberg.

#### Cluster Operations

* Define namespaces by domain (`orchestration`, `lakehouse`, `observability`).
* Manage secrets and connection configs for APIs, object storage, and metadata services.
* Configure autoscaling/resource limits to understand workload behavior and cost/performance trade-offs.

#### Cluster observability

In this stage, we will instrument the local cluster to monitor health, performance, and failures across ingestion and data processing workloads.

* [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) (Prometheus + Alertmanager + Grafana) — Cluster and application metrics.
* [Loki](https://grafana.com/oss/loki/) with [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/) or [Fluent Bit](https://fluentbit.io/) — Centralized logs from pods and jobs.
* [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) — Optional telemetry pipeline for traces and standardized metrics export.
* [K9s](https://k9scli.io/) and `kubectl` — Day-to-day troubleshooting and workload inspection.

#### Key Signals to Track

* **Cluster health:** node and pod status, CPU/memory pressure, restart count.
* **Airflow health:** DAG success/failure rate, task duration, queue/backlog, worker utilization.
* **Data pipeline quality:** ingestion latency, records processed, retries, and failed runs.
* **Platform reliability:** API error rates, catalog availability (Nessie), and storage access failures.

### Data visualization

In this stage, we will deploy and configure [Apache Superset](https://superset.apache.org) in the local Kubernetes cluster to provide a self-service analytics and dashboarding layer on top of the gold datasets produced in the lakehouse.

#### Apache Superset on Kubernetes

* Deploy Apache Superset on Kubernetes using Helm charts and Kubernetes manifests.
* Configure Superset services, application settings, secrets, and persistent components required for local development.
* Expose the Superset web interface inside the cluster for dashboard authoring and exploration.
* Configure database connections so Superset can query curated gold-layer datasets used for analytics.
* Organize access, saved connections, and semantic dataset definitions to support reusable business views.

#### Initial Dashboard

* Create a simple dashboard in Superset using one or more gold-layer datasets.
* Build a first set of charts to validate the end-to-end analytical flow from ingestion to visualization.
* Standardize dashboard creation as part of the platform setup so new curated datasets can be exposed quickly for analysis.

### IA/ML

In this stage, we will explore the use of AI/ML both as a data product capability and as an accelerator for software engineering workflows. The objective is to connect curated lakehouse datasets to predictive use cases while also establishing practical AI-assisted development standards for the project.

#### Development of IA

* Implement machine learning models such as Time Series for predictive analytics and value forecasting.
* Prepare gold-layer datasets so they can be consumed by training and inference workflows with clear feature definitions and reproducible inputs.
* Evaluate model quality using consistent validation criteria to compare forecasting approaches and measure business relevance.
* Organize experimentation workflows to support iterative development of predictive use cases based on the datasets produced in the platform.

#### Use IA for coding

* Creation of [skills](https://agentskills.io/home), `AGENTS.md` and [copilot instrucions](https://docs.github.com/pt/copilot/how-tos/configure-custom-instructions/add-repository-instructions).
* Apply [spec-driven development](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) practices to ensure effective and reliable LLM integration in code.
* Use [OpenCode](https://opencode.ai) and [Oh My Open Code](https://github.com/code-yeongyu/oh-my-openagent) to support AI-assisted implementation workflows during development.
* Use [Spec Kit](https://github.com/github/spec-kit) to structure requirements, implementation plans, and delivery criteria for AI-assisted development.
* Define reusable agent instructions, prompts, and repository conventions so coding assistants operate with project-specific context and engineering standards.
* Apply AI tooling to accelerate documentation, implementation scaffolding, and review workflows while keeping human validation for architecture, quality, and correctness.
