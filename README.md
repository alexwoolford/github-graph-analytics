# GitHub Graph Analytics Demo

Demonstrates **Neo4j Aura Analytics** (serverless GDS) on a GitHub issues collaboration graph.

## Files

| File | Description | Runs On |
|------|-------------|---------|
| `aura_analytics_demo.ipynb` | Analytics notebook - runs GDS algorithms via Aura Analytics | Anywhere |
| `neo4j-load.ipynb` | ETL notebook - loads GitHub issues into Neo4j | **Databricks only** |
| `environment.yml` | Conda environment (Python 3.12.3) | Anywhere |

## Graph Model

```
(:Repository)-[:HAS_ISSUE]->(:Issue)-[:ASSIGNED_TO]->(:User)
                           (:Issue)-[:CREATED_BY]->(:User)
```

The analytics notebook projects an inferred **User→User collaboration graph** (users who work on the same issues).

## Setup

### 1. Create Conda Environment

```bash
conda env create -f environment.yml
conda activate github-graph-analytics
```

### 2. Create `.env` File

```env
# Neo4j Aura Database
NEO4J_URI=neo4j+s://xxxxxxxx.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your-password
NEO4J_DATABASE=neo4j

# Aura API credentials (from console.neo4j.io → Account → API Keys)
AURA_CLIENT_ID=your-client-id
AURA_CLIENT_SECRET=your-client-secret
AURA_TENANT_ID=your-tenant-id
```

### 3. Run Analytics Notebook

```bash
jupyter notebook aura_analytics_demo.ipynb
```

## What the Analytics Notebook Does

1. **Connects** to Neo4j and creates an Aura Analytics session (8GB)
2. **Projects** a User→User collaboration graph entirely in Neo4j (no data pulled to Python)
3. **Runs GDS algorithms** and writes results to the graph:
   - Degree Centrality → `u.degree`
   - PageRank → `u.pagerank`
   - Louvain Community Detection → `u.community`
4. **Queries results** from Neo4j showing:
   - Top collaboration hubs (by degree)
   - Most influential users (by PageRank)
   - Collaboration communities
   - Community validation (proving communities map to real organizations)

## Key Insight: Communities Are Real

The notebook validates that detected communities represent actual organizational boundaries:

| Community | Dominant Repo | Users |
|-----------|---------------|-------|
| 20767 | department-of-veterans-affairs/va.gov-team | 402 |
| 56040 | Expensify/App | 324 |
| 32908 | openjournals/joss-reviews | 299 |
| 52057 | Azure/azure-sdk-for-net | 58 |

## ETL Notebook (Databricks Only)

The `neo4j-load.ipynb` notebook is designed for **Databricks** and:

1. Reads GitHub issues from Parquet files
2. Creates `Issue`, `Repository`, `User` nodes
3. Creates `HAS_ISSUE`, `ASSIGNED_TO`, `CREATED_BY` relationships
4. Uses `neo4j-parallel-spark-loader` for efficient parallel ingestion

**Note**: This notebook uses `dbutils.secrets` for credentials and requires the Neo4j Spark Connector.

## Business Hypotheses

| Metric | Business Meaning | Actionable Insight |
|--------|------------------|---------------------|
| **Degree** | How many collaborators someone works with | High = coordination hub |
| **PageRank** | Influence weighted by connections | High = key for information spread |
| **Community** | Natural working clusters | Team/organizational boundaries |
