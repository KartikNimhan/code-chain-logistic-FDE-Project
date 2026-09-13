# Project Setup Instructions

## Phase 0 — Ingest Legacy Data

Simulates a legacy enterprise environment (MSSQL on Docker) and loads the
source dataset into it under a messy legacy schema.

### Prerequisites

- Docker (or an EC2 instance running Docker)
- Python 3.12
- ODBC Driver 18 for SQL Server installed locally

### 1. Get the dataset

Source: [`data/source/data.txt`](../data/source/data.txt)

Download the dataset linked there and place it at:

```
data/raw/dynamic_supply_chain_logistics_dataset.csv
```

### 2. Start the legacy MSSQL server

Run this on the target host (local Docker or an EC2 instance).

**Windows (single line):**

```powershell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=FdeEnterprisePass123!" -p 1433:1433 --name legacy-mssql -d mcr.microsoft.com/mssql/server:2022-latest
```

**Windows (multi-line):**

```powershell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=FdeEnterprisePass123!" `
   -p 1433:1433 --name legacy-mssql `
   -d mcr.microsoft.com/mssql/server:2022-latest
```

**macOS/Linux (multi-line):**

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=FdeEnterprisePass123!" \
   -p 1433:1433 --name legacy-mssql \
   -d mcr.microsoft.com/mssql/server:2022-latest
```

**With a persistent volume (e.g. inside EC2):**

```bash
docker run -v mssql_data:/var/opt/mssql \
  -e "ACCEPT_EULA=Y" \
  -e "MSSQL_SA_PASSWORD=FdeEnterprisePass123!" \
  -p 1433:1433 \
  --name legacy-mssql \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Set these in [`.env`](../.env) — [`scripts/ingest_legacy_data.py`](../scripts/ingest_legacy_data.py)
reads them via `python-dotenv`:

```
SQL_SERVER_HOST=localhost
SQL_SERVER_PORT=1433
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=FdeEnterprisePass123!
```

> `SQL_ADMIN_USER` and `SQL_ADMIN_PASSWORD` are currently missing from `.env`
> and must be added before the ingestion script will connect successfully.

### 5. Run the ingestion script

```bash
python scripts/ingest_legacy_data.py
```

This loads the CSV, remaps a subset of columns to legacy-style names, and
writes them to `dbo.TBL_SC_FLEET_HIST_RAW` in the `master` database.
