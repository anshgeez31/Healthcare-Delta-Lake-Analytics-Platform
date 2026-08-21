# Healthcare-Delta-Lake-Analytics-Platform
# Healthcare Lakehouse Analytics Platform

An end-to-end **Databricks Delta Live Tables (DLT)** data engineering solution for processing healthcare and patient admission data using the **Medallion Architecture**.

The platform implements a scalable **Bronze–Silver–Gold data pipeline** with incremental streaming ingestion, data quality enforcement, schema management, reference-data enrichment, and multidimensional analytics.

## 🏗️ Architecture

```text
                         ┌─────────────────────────┐
                         │      Source Systems     │
                         │                         │
                         │  Patient Admission Data │
                         │  Diagnosis Reference    │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │      BRONZE LAYER       │
                         │                         │
                         │  • Raw Data Ingestion   │
                         │  • Schema Validation    │
                         │  • Data Quality Checks  │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │      SILVER LAYER       │
                         │                         │
                         │  • Data Transformation  │
                         │  • Data Enrichment      │
                         │  • Business Rules       │
                         │  • Data Validation      │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │       GOLD LAYER        │
                         │                         │
                         │  • Aggregations         │
                         │  • KPIs & Statistics    │
                         │  • Analytical Datasets  │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │    Analytics & BI       │
                         │ Reporting & Research    │
                         └─────────────────────────┘
```

The pipeline follows a layered architecture in which Bronze handles ingestion, Silver performs cleansing and enrichment, and Gold delivers curated analytical datasets.

## 🚀 Key Features

* **Delta Live Tables (DLT)** based pipeline orchestration
* **Medallion Architecture** implementation
* **Incremental and streaming data ingestion**
* Declarative **data quality expectations**
* Automated handling of invalid records
* **Schema validation and evolution**
* Reference-data enrichment using joins
* Multi-dimensional healthcare analytics
* Pipeline **monitoring and observability**
* Data lineage tracking
* Scalable distributed processing

## 📂 Project Structure

```text
Healthcare_Lakehouse_Analytics_Platform/
│
├── notebook/
│   ├── healthcare_dlt_processing.ipynb
│   │   └── DLT pipeline definitions and transformations
│   │
│   └── feed_raw_tables.ipynb
│       └── Source-data ingestion and preparation
│
├── data/
│   ├── diagnosis_mapping.csv
│   └── patients_daily_file_*.csv
│
└── README.md
```

## 📊 Data Sources

### Diagnosis Reference Data

**File:** `diagnosis_mapping.csv`

Reference/master data used to map diagnosis codes to human-readable descriptions.

```text
diagnosis_code
diagnosis_description
```

### Patient Admission Data

**Files:** `patients_daily_file_*.csv`

Daily patient admission records used as the primary transactional input for the streaming pipeline.

```text
patient_id
name
age
gender
address
contact_number
admission_date
diagnosis_code
```

The patient files are processed incrementally, allowing newly arriving records to flow through the downstream pipeline automatically.

## 🥉 Bronze Layer

The Bronze layer provides controlled ingestion of source datasets while applying foundational validation rules.

### `diagnostic_mapping`

Stores diagnosis reference data used for downstream enrichment.

**Data quality constraints:**

* `diag_code_not_null`
* `diag_desc_not_null`

Invalid records are removed using:

```text
ON VIOLATION DROP ROW
```

### `daily_patients`

Streaming table responsible for ingesting patient admission records.

**Data quality constraints:**

* `pk_not_null`
* `required_fields`

Critical records that fail validation are excluded before downstream processing.

## 🥈 Silver Layer

The Silver layer transforms Bronze datasets into validated, standardized, and enriched business-ready records.

### `processed_patient_data`

The table enriches patient records with diagnosis descriptions through a `LEFT JOIN` using `diagnosis_code`.

```sql
Patient Data
     │
     │ diagnosis_code
     ▼
Diagnosis Reference
     │
     ▼
Enriched Patient Dataset
```

`COALESCE` is used to provide controlled handling for unmapped diagnosis values.

The `has_diagnosis` quality rule ensures that enriched records contain a valid diagnosis description.

## 🥇 Gold Layer

The Gold layer exposes curated datasets optimized for reporting, operational analytics, and downstream consumption.

### `patient_statistics_by_admission_date`

Provides admission-date-based operational analytics.

**Key metrics:**

* Total patient admissions
* Average patient age
* Patient counts by diagnosis
* Daily admission statistics

**Use cases:**

* Hospital capacity planning
* Admission trend analysis
* Workforce planning
* Resource allocation

### `patient_statistics_by_diagnosis`

Provides diagnosis-centric healthcare analytics.

**Key metrics:**

* Patient count by diagnosis
* Average age
* Minimum and maximum age
* Gender distribution
* Diagnosis prevalence

**Use cases:**

* Medical research
* Disease prevalence analysis
* Population health analytics
* Diagnosis trend analysis

### `patient_statistics_by_gender`

Provides demographic analytics segmented by gender.

**Key metrics:**

* Patient population by gender
* Average age
* Age distribution
* Diagnosis diversity

**Use cases:**

* Demographic analysis
* Population health studies
* Healthcare segmentation
* Comparative analysis

## 🛡️ Data Quality Framework

The pipeline implements **Delta Live Tables expectations** and constraint-based validation to maintain data integrity throughout the processing lifecycle.

### Validation Mechanisms

| Mechanism               | Purpose                                             |
| ----------------------- | --------------------------------------------------- |
| `EXPECT`                | Defines declarative data quality rules              |
| `ON VIOLATION DROP ROW` | Removes records that fail critical validation       |
| `ON VIOLATION SET`      | Assigns controlled fallback values where applicable |

### Quality Metrics

The pipeline can monitor:

* Total records processed
* Valid records
* Rejected records
* Constraint violations
* Violation percentages
* Data freshness
* Schema changes
* Processing throughput

## ⚡ Streaming & Incremental Processing

The pipeline uses streaming semantics to process newly arriving patient records incrementally.

Key capabilities include:

* Continuous/incremental ingestion
* Automatic checkpoint management
* Fault recovery
* Low-latency processing
* Incremental propagation through Bronze → Silver → Gold

This approach avoids repeatedly reprocessing the complete historical dataset.

## 🔄 Schema Management

The pipeline incorporates schema-management capabilities including:

* Schema inference
* Explicit type casting
* Schema validation
* Controlled schema evolution
* Compatibility management

## 📈 Multi-Dimensional Analytics

The Gold layer enables analytics across multiple business dimensions:

```text
                    Patient Dataset
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
      Admission Date   Diagnosis      Gender
             │             │             │
             ▼             ▼             ▼
       Time-Series      Medical      Demographic
        Analytics      Analytics      Analytics
```

This enables the same curated patient dataset to support operational, clinical, and demographic analysis.

## 🔍 Monitoring & Observability

The Databricks DLT interface provides centralized monitoring across pipeline execution, data quality, lineage, and performance.

### Pipeline Monitoring

* Pipeline state
* Successful and failed updates
* Execution duration
* Processing status

### Data Quality Monitoring

* Constraint evaluation results
* Violation counts
* Quality trends
* Rejected-record statistics

### Data Lineage

```text
Source
  ↓
Bronze
  ↓
Silver
  ↓
Gold
```

### Performance Monitoring

* Records processed per update
* Pipeline execution duration
* Processing throughput
* Failed records
* Data-quality violation rate
* Schema modification events

## 🛠️ Technology Stack

| Technology                 | Purpose                            |
| -------------------------- | ---------------------------------- |
| **Databricks**             | Cloud data engineering platform    |
| **Delta Live Tables**      | Declarative pipeline development   |
| **Delta Lake**             | Reliable lakehouse storage         |
| **Apache Spark**           | Distributed data processing        |
| **Python / PySpark**       | Transformation logic               |
| **Unity Catalog**          | Data governance and access control |
| **Medallion Architecture** | Layered data processing            |

## ⚙️ Prerequisites

* Databricks workspace
* Delta Live Tables capabilities
* Unity Catalog
* Required catalog/schema permissions
* Access to source datasets
* Configured compute resources

## ▶️ Pipeline Execution

### 1. Prepare Source Data

Run:

```text
feed_raw_tables.ipynb
```

This prepares the source datasets and creates the required raw tables:

```text
gds_de_bootcamp_new.default.raw_diagnosis_map
gds_de_bootcamp_new.default.raw_patients_daily
```

### 2. Configure the DLT Pipeline

Example configuration:

```json
{
  "name": "healthcare_dlt_pipeline",
  "notebook_path": "/notebooks/healthcare_dlt_processing",
  "target": "gds_de_bootcamp_new.default",
  "configuration": {
    "quality": "bronze"
  }
}
```

### 3. Execute the Pipeline

1. Configure the DLT pipeline in Databricks.
2. Associate the processing notebook.
3. Configure the target catalog and schema.
4. Start the pipeline.
5. Monitor pipeline execution.
6. Validate data-quality metrics.
7. Verify Bronze, Silver, and Gold outputs.
8. Review lineage and performance metrics.

## 💼 Business Use Cases

### Hospital Operations

* Patient admission monitoring
* Capacity planning
* Resource allocation
* Patient-volume analysis
* Operational trend detection

### Healthcare Analytics

* Diagnosis prevalence analysis
* Demographic health analysis
* Age-based population analysis
* Disease trend identification
* Healthcare research

### Data Governance

* Data-quality monitoring
* Compliance reporting
* Auditability
* Data lineage
* Controlled transformations

## 🔧 Troubleshooting

### Data Quality Violations

**Issue:** High DLT expectation violation rates.

**Resolution:**

* Review source data
* Identify violated constraints
* Inspect null or malformed fields
* Validate source-system data
* Reassess constraints against business requirements

### Schema Evolution

**Issue:** Pipeline failures following source-schema changes.

**Resolution:**

* Compare expected and incoming schemas
* Identify changed or newly introduced columns
* Review schema-evolution configuration
* Apply appropriate schema evolution strategies
* Update transformation and casting logic

### Performance Issues

**Issue:** Increased pipeline latency or execution time.

**Resolution:**

* Review input data volume
* Analyze join operations
* Optimize aggregations
* Evaluate compute resources
* Review partitioning and storage layout
* Remove unnecessary transformations

## 🚀 Future Enhancements

### Advanced Analytics

* Predictive analytics
* Machine learning integration
* Patient-risk prediction
* Admission forecasting
* Anomaly detection
* Statistical modeling

### Data Governance

* Fine-grained access control
* Column-level security
* Sensitive-data protection
* Enhanced lineage
* Audit logging
* Regulatory compliance

### Performance Engineering

* Z-Ordering
* Advanced partitioning
* Storage optimization
* Caching
* Query optimization
* Workload-specific tuning

## 🎯 Project Objective

The objective of this project is to demonstrate how a modern **lakehouse-based data engineering architecture** can transform raw healthcare records into trusted, analytics-ready datasets while maintaining **data quality, scalability, observability, and governance**.

The implementation combines **Delta Live Tables, streaming ingestion, declarative data-quality expectations, layered transformations, reference-data enrichment, and analytical aggregations** to establish a scalable healthcare data-processing foundation.

## 👨‍💻 Project Summary

**Healthcare Lakehouse Analytics Platform** demonstrates an enterprise-style healthcare data pipeline built around the **Databricks Lakehouse architecture**.

The project showcases:

```text
Raw Healthcare Data
        ↓
Streaming Ingestion
        ↓
Bronze — Raw & Validated
        ↓
Silver — Cleaned & Enriched
        ↓
Gold — Curated & Aggregated
        ↓
Analytics / BI / Research
```

This architecture provides a strong foundation for building scalable healthcare analytics solutions and can be extended with machine learning, predictive analytics, advanced governance, and enterprise-grade optimization.
