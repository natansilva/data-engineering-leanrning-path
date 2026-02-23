# Data Engineering Learning Path

Roadmap developed to guide my mentoring program in Data Engineering

## Roadmap

### Environment Setup and Configuration

Installation and configuration of tools that assist and streamline daily development work.

#### Shell

* Installation and configuration of Zsh and Oh My Zsh
* Configuration management (.zshrc)
* Custom aliases and functions
* Plugins
  * git
  * zsh-autosuggestions
  * zsh-autocomplete
  * zsh-syntax-highlighting
* Shell tools
  * git
  * jq
  * yq
  * curl

#### Python

Installation and creation of a development environment that can use multiple Python versions and manage pip packages.

* Python installation
* Virtual environment management
  * pyenv
  * venv
  * virtualenv
* uv
  * ruff

#### IDE

Tools that can be used with IDEs to increase productivity and facilitate daily work.

* Creation of [skills](https://agentskills.io/home).
* Copilot integration.
* MCP integration.

### Data Ingestion

Data ingestion from different sources using Python and creation of DAGs to orchestrate ingestion.

#### API Ingestion

* Ingestion of data from [OpenF1](https://openf1.org/docs/#introduction)
* Ingestion of unstructured data from [openfootball](https://github.com/openfootball)
* Ingestion of CSV data from [Kaggle on Olympics](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results)

#### Orchestration

* Cron
* Airflow
  * Scheduler
  * Executor
  * Workers
  * DAGs
  * Tasks
  * Operators

### Lakehouse

Here we store data in the bronze layer (raw), clean and standardize in the silver layer (trusted), and prepare data for BI or AI/ML models in the gold layer (refined).

* [Apache Iceberg](https://iceberg.apache.org) for data storage
* [Apache Spark](https://spark.apache.org) for data cleaning and standardization
* [SQLMesh](https://sqlmesh.readthedocs.io/en/stable/) with [Duckdb](https://duckdb.org)/[Trino](https://trino.io) for data refinement.
* Catalog service through [Project Nessie](https://projectnessie.org).

### Infrastructure

* Docker
* Kubernetes
* Installation of tools used in a k8s environment.
* Environment monitoring

### Data visualization

To be defined.

### IA/ML

Use machine learning models in tables to predict values and implements tools to work with AI agents.
