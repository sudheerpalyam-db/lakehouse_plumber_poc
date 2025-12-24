# Lakehouse Plumber POC

A proof-of-concept implementation of Lakeflow Declarative Pipelines using the [Lakehouse Plumber](https://lakehouse-plumber.readthedocs.io/) framework.

## Overview

This project demonstrates a medallion architecture data pipeline for an e-commerce use case:

- **Customers** - Customer master data
- **Orders** - Sales order transactions
- **Products** - Product catalog

## Architecture

```
Landing (CSV) --> Raw Layer --> Bronze Layer --> Silver Layer
     |               |              |               |
  CloudFiles    raw_data.*     bronze.*        silver.*
  Auto Loader   (streaming)    (cleansed)      (enriched)
```

## Project Structure

```
lakehouse_plumber_poc/
├── lhp.yaml                    # LHP project configuration
├── databricks.yml              # Databricks Asset Bundle config
├── pipelines/                  # Pipeline YAML definitions
│   ├── 01_raw_ingestion/       # CSV ingestion using CloudFiles
│   │   └── csv_ingestions/
│   │       ├── customer_ingestion.yaml
│   │       ├── orders_ingestion.yaml
│   │       └── products_ingestion.yaml
│   ├── 02_bronze/              # Bronze layer transforms
│   │   ├── customer_bronze.yaml
│   │   └── orders_bronze.yaml
│   └── 03_silver/              # Silver layer transforms
│       ├── dim/
│       │   └── customer_silver_dim.yaml
│       └── fct/
│           └── orders_silver_fct.yaml
├── templates/                  # Reusable action templates
│   └── csv_ingestion_template.yaml
├── presets/                    # Table property presets
│   └── bronze_layer.yaml
├── schemas/                    # Schema definitions
│   ├── customers_schema.yaml
│   ├── orders_schema.yaml
│   └── products_schema.yaml
├── substitutions/              # Environment-specific variables
│   ├── dev.yaml
│   ├── tst.yaml
│   └── prod.yaml
├── resources/lhp/              # Generated bundle resources
└── generated/                  # Generated Python DLT code
```

## Prerequisites

- Python 3.10+ (required for LHP CLI)
- Databricks CLI configured
- Access to Databricks workspace

## Quick Start

1. **Create virtual environment and install LHP**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install lakehouse-plumber
   ```

2. **Configure environment variables**

   Edit `substitutions/dev.yaml` with your catalog and schema names:
   ```yaml
   dev:
     catalog: your_catalog
     raw_schema: raw_data
     bronze_schema: bronze
     silver_schema: silver
     landing_volume: /Volumes/your_catalog/raw_data/landing
   ```

3. **Validate configuration**
   ```bash
   lhp validate -e dev --verbose
   ```

4. **Generate DLT code**
   ```bash
   lhp generate -e dev
   ```

5. **Deploy to Databricks**
   ```bash
   databricks bundle deploy --target dev
   ```

6. **Run pipelines**
   ```bash
   databricks bundle run raw_ingestions_pipeline --target dev
   databricks bundle run bronze_load_pipeline --target dev
   databricks bundle run silver_load_pipeline --target dev
   ```

## LHP CLI Commands

| Command | Description |
|---------|-------------|
| `lhp validate -e <env>` | Validate pipeline configurations |
| `lhp generate -e <env>` | Generate DLT Python code |
| `lhp generate -e <env> --force` | Regenerate all files |
| `lhp list-templates` | List available templates |
| `lhp list-presets` | List available presets |
| `lhp show <flowgroup>` | Show resolved configuration |
| `lhp deps -f dot` | Generate dependency graph |

## Pipelines

### raw_ingestions
Ingests CSV files from landing volume using CloudFiles Auto Loader:
- `customers` - Customer master data
- `orders` - Order transactions
- `products` - Product catalog

### bronze_load
Cleanses and transforms raw data:
- Applies data quality expectations
- Adds audit metadata columns

### silver_load
Creates enriched dimension and fact tables:
- `customer_silver_dim` - Customer dimension
- `orders_silver_fct` - Orders fact table

## Data Flow

1. CSV files land in `/Volumes/{catalog}/raw_data/landing/{table}/`
2. `raw_ingestions` pipeline streams data to `raw_data.*` tables
3. `bronze_load` pipeline cleanses and writes to `bronze.*` tables
4. `silver_load` pipeline enriches and writes to `silver.*` tables

## Resources

- [Lakehouse Plumber Documentation](https://lakehouse-plumber.readthedocs.io/)
- [Lakeflow Declarative Pipelines](https://docs.databricks.com/en/delta-live-tables/)
- [Databricks Asset Bundles](https://docs.databricks.com/en/dev-tools/bundles/)
