# 🏎 Formula 1 Data Engineering Project on Azure

This project is a full end-to-end data engineering solution using Formula 1 racing data. It leverages **Azure Databricks**, **Azure Data Lake Storage Gen2**, **Azure Data Factory**, and **Power BI** to build a scalable, automated pipeline for ingesting, transforming, and visualizing motorsport data.

---

## 🔧 Project Description
We process historical and incremental race data from the [Ergast Developer API](http://ergast.com/mrd/) to:
- Ingest raw data files into ADLS Gen2
- Transform raw files using PySpark and convert to Delta format
- Store clean data in structured databases (raw, processed, presentation)
- Automate the ETL process using Azure Data Factory
- Visualize key racing insights in Power BI

**File Types Ingested:**
| File Name       | File Type                | Incremental? |
|----------------|--------------------------|--------------|
| Circuits       | CSV                      | No           |
| Races          | CSV                      | No           |
| Constructors   | Single Line JSON         | No           |
| Drivers        | Single Line Nested JSON  | No           |
| Results        | Single Line JSON         | Yes          |
| PitStops       | Multi Line JSON          | Yes          |
| LapTimes       | Split CSV Files          | Yes          |
| Qualifying     | Split Multi Line JSON    | Yes          |

Incremental ingestion is implemented for the last four files. Each ingestion notebook accepts a `file_date` parameter to filter and load only relevant new data. A merge strategy using `MERGE INTO` with Delta Lake is used to upsert new records into target Delta tables.

---

## 🧱 Architecture Overview

- **Azure Data Lake Storage Gen2**: Hierarchical storage with 3 logical containers: `raw`, `processed`, and `presentation`
- **Azure Databricks**: Core compute layer used for all transformation logic
- **Azure Data Factory**: Pipeline orchestration for automated data workflows
- **Azure Key Vault**: Secure storage of client credentials
- **Power BI**: Final dashboard layer connected to presentation data

---

## 🏗️ Components and Setup

### 🧱 Azure Databricks
- **Workspace name**: `fomula1_azure_db`
- **Cluster name**: `formula1_azure_cluster`
  - Single Node | No Isolation Shared | Runtime: 11.3 LTS | `Standard_DS3_v2` | Auto-Terminate: 120 mins

### ☁️ Azure Data Lake Storage Gen2
- **Storage account name**: `formula1azuredlforuse`
- **Containers**:
  - `raw`: Source CSV/JSON data from Ergast API (organized by date)
  - `processed`: Cleaned Delta tables with schema enforcement
  - `presentation`: Final aggregated and joined datasets for analytics
  - `demo`: Optional test space

### 🔐 Secure Authentication
- **App Registration**: `formula1`
  - Client ID: 
  - Tenant ID: 
  - Secret Value: 
- **IAM Role**: `Storage Blob Data Contributor` assigned to app

---

## 🔮 Table Strategy

To enable efficient querying, data validation, and lifecycle management, we adopt the following structure:

- **Raw Layer**: External Delta Tables
  - These are defined with SQL `LOCATION` syntax pointing to the raw container. This avoids data duplication and provides schema-on-read flexibility.

- **Processed & Presentation Layers**: Managed Delta Tables
  - These are created within Databricks using the managed metastore. Tables benefit from:
    - ACID transactions
    - Schema evolution and enforcement
    - Data versioning (time travel)
    - Partitioning and indexing
    - Optimized reads and writes

Using tables provides better integration with SQL interfaces, easier reproducibility in reporting, and compatibility with tools like Power BI.

---

## 🔄 Incremental Ingestion Logic

For incremental files (`Results`, `PitStops`, `LapTimes`, `Qualifying`):
- A `file_date` is passed to the notebook
- Files are filtered to only load new records based on this date
- Deduplication and upsert logic is performed using `merge_delta_data` function
- Final Delta tables are stored in `processed` layer

---

## 🔬 Processing & Ingestion Notebooks
- **Ingestion notebooks** parameterized with `file_date` and `data_source`
- Applied schema definitions using `StructType`
- Columns renamed and curated
- Ingestion timestamp added
- Tables created with Delta Lake

---

## 📆 Azure Data Factory Pipelines

### Factory: `formula1-df-for-use`
- **Pipelines**:
  - `p1_ingest_formula1_data`: Loads raw files via Databricks notebooks
  - `transform_pipeline`: Builds derived tables like race results and standings
  - `process_pipeline`: Orchestrates full flow
- **Tumbling Trigger**: Sends dates to notebooks to support historical and incremental loads

---

## 📊 Power BI Dashboard
- Connected to **presentation** Delta tables
- Visualizations:
  - Driver Standings by year
  - Constructor Points by year
  - Detailed Race Results

---

## 💡 Why Delta Lake?
Delta Lake is used for all managed tables due to:
- **ACID guarantees**: reliable update and delete support
- **Time travel**: rollback data to previous versions
- **Scalability**: optimized for large-scale analytics
- **Compatibility**: native integration with Databricks, Power BI, and Azure Synapse

---

