# Daily PDF Chat

> **High-Performance Tabular Exploratory Data Analysis (EDA) Engine & Automated Data Profiling Suite**

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Build Status](https://img.shields.io/badge/CI-passing-brightgreen.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

---

## 🎯 Problem Statement & Target Audience

### The Problem
Data engineers, analysts, and data scientists spend a disproportionate amount of time writing repetitive boilerplate code in Jupyter notebooks to perform initial data inspection. When presented with unknown CSV datasets, critical tasks like detecting data type anomalies, identifying missing value clusters, assessing memory consumption, and plotting basic distribution charts require manual execution. This lack of standardization leads to:
- Unspotted schema drift and subtle anomaly propagation downstream.
- Inconsistent data documentation and auditability across analytics teams.
- Significant delay between raw data ingestion and exploratory insights.

### Target Audience
- **Data Analysts**: Needing instant visual summaries and dataset health checks without building ad-hoc plotting code.
- **Analytics Engineers**: Validating tabular datasets and schema integrity prior to writing dbt models or pipeline transformations.
- **MLOps Engineers**: Auditing input features, checking for missingness, and logging structured JSON profile metadata.

---

## ✨ Key Features

- **⚡ Instant Schema & Memory Profiling**: Automatically infer column data types, identify exact schema structures, and calculate byte-level memory footprints leveraging DuckDB and Pandas.
- **🛡️ Automated Data Hygiene & Anomaly Detection**: Uncover missing value clusters with missingness heatmaps, uncover high-cardinality flags, and detect baseline schema anomalies.
- **📊 Interactive Visualization Engine**: Generates production-grade, interactive distribution histograms, box plots, and cardinality charts powered by Plotly.
- **📄 Dual Export Engine (HTML & JSON)**: Export clean, standalone interactive HTML reports for stakeholder presentation alongside structured JSON payloads for pipeline audit trails.

---

## 🏗️ Architecture & System Design

`Daily PDF Chat` features a decoupled architecture separating the analytical compute engine from API delivery and command-line execution interfaces.

### Data Flow & Component Architecture

```
                                  ┌───────────────────────────┐
                                  │      Input CSV File       │
                                  └─────────────┬─────────────┘
                                                │
                                                ▼
                                  ┌───────────────────────────┐
                                  │    src/main.py (CLI)      │
                                  │     or src/api.py (REST)  │
                                  └─────────────┬─────────────┘
                                                │
                                                ▼
                                  ┌───────────────────────────┐
                                  │    Core EDA Engine        │
                                  │      (src/core.py)        │
                                  │  [DuckDB + Pandas Compute]│
                                  └─────────────┬─────────────┘
                                                │
                         ┌──────────────────────┴──────────────────────┐
                         ▼                                             ▼
          ┌────────────────────────────┐                ┌────────────────────────────┐
          │     Domain Validation      │                │    Plotly Chart Engine     │
          │      (src/models.py)       │                │     (Histogram / Box)      │
          └──────────────┬─────────────┘                └──────────────┬─────────────┘
                         │                                             │
                         └──────────────────────┬──────────────────────┘
                                                │
                                                ▼
                                  ┌───────────────────────────┐
                                  │ HTML / JSON Profile Report│
                                  └───────────────────────────┘
```

### Module Responsibilities

| Module | Purpose |
| :--- | :--- |
| **`src/models.py`** | Strong Pydantic domain models defining data schemas for column statistics, anomaly objects, chart configurations, and API response payloads. |
| **`src/core.py`** | High-performance compute engine executing SQL operations via DuckDB and Pandas for statistical aggregates, anomaly detection, and Plotly visualization generation. |
| **`src/api.py`** | Production FastAPI routes handling file upload streaming, synchronous profiling execution, and JSON chart payload delivery. |
| **`src/main.py`** | Core application entrypoint supporting both Typer/Argparse CLI execution and Uvicorn web service hosting. |

---

## 💻 Technology Stack

- **Core Engine & Aggregations**: [DuckDB](https://duckdb.org/) (In-memory SQL OLAP engine), [Pandas](https://pandas.pydata.org/), [NumPy](https://numpy.org/)
- **Visualizations**: [Plotly Py](https://plotly.com/python/), [Streamlit](https://streamlit.io/)
- **API Architecture**: [FastAPI](https://fastapi.tiangolo.com/), [Pydantic v2](https://docs.pydantic.dev/)
- **Server Execution**: [Uvicorn](https://www.uvicorn.org/)

---

## ⚙️ Installation & Virtual Environment Setup

### Prerequisites
- Python 3.10 or higher
- `pip` package manager

### 1. Clone the Repository
```bash
git clone https://github.com/daily-project-factory/daily-pdf-chat.git
cd daily-pdf-chat
```

### 2. Create and Activate Virtual Environment

**Linux / macOS:**
```bash
python3 -m venv venv
source venv/bin/activate
```

**Windows (PowerShell):**
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 🔧 Environment Variables Configuration

The application uses environment variables to adjust runtime limits and engine performance parameters. Create a `.env` file in the root directory based on the included template:

```bash
cp .env.example .env
```

### Supported Variables (`.env.example`)
```env
# Application Settings
APP_NAME="Daily PDF Chat EDA Engine"
ENVIRONMENT="development"
LOG_LEVEL="INFO"

# Server Configuration
HOST="0.0.0.0"
PORT=8000

# Compute Limits
MAX_UPLOAD_SIZE_MB=250
DUCKDB_MEMORY_LIMIT="2GB"
DUCKDB_THREADS=4
```

---

## 🚀 Usage

`Daily PDF Chat` operates as both a standalone Command Line Interface (CLI) tool and a RESTful API service.

### Command Line Interface (CLI)

#### Profile a Local CSV File
Generate an instant summary output directly in your terminal:
```bash
python -m src.main profile path/to/dataset.csv
```

#### Export Profile Report to HTML or JSON
```bash
# Export interactive HTML report
python -m src.main profile path/to/dataset.csv --format html --output report.html

# Export machine-readable JSON metadata
python -m src.main profile path/to/dataset.csv --format json --output summary.json
```

#### Launch the Web Application Server
```bash
python -m src.main serve --host 0.0.0.0 --port 8000
```

---

### REST API Endpoints & Curl Examples

When running the web server (`python -m src.main serve`), access interactive OpenAPI documentation at `http://localhost:8000/docs`.

#### 1. System Health Check
```bash
curl -X GET "http://localhost:8000/api/v1/health"
```

#### 2. Profile a CSV Dataset
Upload a CSV file to retrieve structural statistics, memory metrics, and detected anomalies.
```bash
curl -X POST "http://localhost:8000/api/v1/profile" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@data/sample.csv"
```

*Example Response:*
```json
{
  "filename": "sample.csv",
  "row_count": 10500,
  "column_count": 8,
  "memory_footprint_bytes": 672400,
  "schema_anomalies": [
    {
      "column_name": "user_id",
      "anomaly_type": "HIGH_CARDINALITY",
      "severity": "WARNING",
      "description": "Column exhibits 100% unique values."
    }
  ],
  "columns": {
    "age": {
      "dtype": "int64",
      "null_count": 42,
      "null_percentage": 0.4,
      "mean": 34.2,
      "std": 11.5,
      "min": 18,
      "max": 82
    }
  }
}
```

#### 3. Render Interactive Distribution Chart Spec
Fetch Plotly-compatible JSON chart specifications for a target numeric column.
```bash
curl -X POST "http://localhost:8000/api/v1/charts/distribution?column=age" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@data/sample.csv"
```

---

## 🧪 Automated Testing Instructions

The project uses `pytest` for unit and integration testing across core analytics and API endpoints.

### Execute All Tests
```bash
pytest
```

### Run Module-Specific Test Suites
```bash
# Core EDA engine tests
pytest tests/test_core.py -v

# FastAPI integration tests
pytest tests/test_api.py -v
```

### Generate Coverage Report
```bash
pytest --cov=src --cov-report=term-missing
```

---

## 📁 Project Structure

```
daily-pdf-chat/
├── .env.example            # Environment configuration template
├── .gitignore              # Standard Python and execution ignore patterns
├── requirements.txt        # Production dependencies
├── src/
│   ├── __init__.py         # Package initialization
│   ├── models.py           # Pydantic schemas for metrics, reports, and payloads
│   ├── core.py             # DuckDB/Pandas analytical compute & chart engine
│   ├── api.py              # FastAPI endpoints and router logic
│   └── main.py             # CLI commands and Uvicorn server launcher
└── tests/
    ├── __init__.py         # Test package initialization
    ├── test_core.py        # Core EDA and statistical engine tests
    └── test_api.py         # HTTP endpoint tests and payload assertions
```

---

## 🛣️ Roadmap / Future Enhancements

- [ ] **Multi-Table Comparison & Schema Drift**: Side-by-side comparative analysis between historical dataset snapshots to detect schema changes.
- [ ] **Direct Cloud Storage Ingestion**: Direct streaming profiling for AWS S3, Google Cloud Storage, and Azure Blob storage paths using DuckDB extensions.
- [ ] **Code Generation**: Export auto-generated Python and PySpark cleaning scripts directly from detected data anomalies.
- [ ] **Parquet & Arrow Native Support**: Native support for memory-mapped Apache Arrow tables and Parquet file formats.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.