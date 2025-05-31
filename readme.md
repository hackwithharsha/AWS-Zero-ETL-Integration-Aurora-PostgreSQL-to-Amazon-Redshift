# Zero-ETL Integration: Aurora PostgreSQL to Amazon Redshift

The goal of this project is to practice AWS Zero-ETL by replicating data in near real-time from an Amazon Aurora PostgreSQL database to Amazon Redshift using AWS-managed features — **without writing any ETL code**.

This exercise simulates a real-time analytics use case where operational data in Aurora is mirrored in Redshift for analysis.

### Tech Stack

- **Amazon Aurora PostgreSQL** (Serverless v2 -[Version 16.4 and higher](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html))
- **Amazon Redshift Serverless**
- **AWS Zero-ETL Integration**

### Set up Aurora PostgreSQL Serverless v2

1. Go to **RDS > Create database**.
2. Select:
   - Database Creation Method : **Standard Create**
   - Engine: **Aurora (PostgreSQL Compatible)**
   - Available Version: **Aurora PostgreSQL (Compatible with PostgreSQL 16.6)**
   - Template: **Dev/Test**
   - Capacity type: **Serverless**
   - DB cluster identifier: `aurora-zeroetl-demo`
   - Username: `svc_customer`
   - Credentials Management : `Self managed` - checkmark `Auto generate password`
   - Cluster storage configuration: `Aurora Standard`
   - Instance Configuration `Serverless v2`
   - Min/Max ACU: `0.5–2`
   - Initial database name : `customer`
   - Enable `Data API`
3. Launch the DB.

[Please understand pricing for Aurora Postgres Serverless v2](https://repost.aws/questions/QUbtHMLZXiS4Kppi7KMIB5YQ/aurora-serverless-v2-minimum-cost-setup-for-development-environment)

### Load Sample Data in Aurora

1. Connect using any SQL client (e.g., DBeaver, pgAdmin, or psql).
2. Run:

```sql
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50),
  email VARCHAR(100),
  created_at TIMESTAMP DEFAULT now()
);

INSERT INTO public.customers (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com'),
('Charlie', 'charlie@example.com');

-- select * from information_schema.tables;
SELECT * FROM customers;
```

### Set up Amazon Redshift Serverless

1. Go to **Amazon Redshift > Serverless > Create Workgroup**.
2. Configure:

   * Workgroup name: `redshift-zeroetl-demo`
   * Base Capacity RPU: `8`
   * Namespace: `zeroetl-demo`
   * Select CheckMark: `Customize admin user credentials`
   * `Generate a password`

### Create Zero-ETL Integration

1. Go to **Amazon Aurora > `Customer` Database > Zero-ETL Integrations**.
2. Click **Create integration**.
3. Select:

   * Source: Aurora PostgreSQL cluster
   * Database: `customer`
   * Target: Redshift namespace
4. Configure:
   * Select `customers` table only.
5. Confirm and create.

6. Go to **Amazon Redshift > `Customer` Database > Zero-ETL Integrations**. - `Seelct Create Database Popup`

Wait \~5–10 minutes for the initial sync to complete.

### Query Data in Redshift

```sql
SELECT * 
FROM svv_all_tables
WHERE table_name ILIKE '%customers%';
```

1. Go to **Query Editor v2** in Redshift.
2. Connect using your workgroup.
3. Run:

```sql
SELECT * FROM zeroetl_demo.public.customers;
```

You should see the replicated data from Aurora.

4. Test insert/update in Aurora and re-run Redshift query to see live sync:

```sql
-- Aurora
INSERT INTO customers (name, email) VALUES ('Diana', 'diana@example.com');
```

### Next Steps

* Try schema changes (e.g., `ALTER TABLE`) and observe impact

### Cleanup (`IMPORTANT`)

* Delete:

  * Aurora cluster
  * Redshift workgroup and namespace
  * Zero-ETL integration

### Cost Estimate

| Resource            | Duration | Est. Cost         |
| ------------------- | -------- | ----------------- |
| Aurora Serverless   | 5 hours  | \~\$1.00          |
| Redshift Serverless | 4 hours  | \~\$2.50          |
| **Total**           | One-time | **\~\$3.50–5.00** |