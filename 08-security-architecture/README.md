# Project 7: Serverless Data Lake and Analytics Pipeline

**Architecture type:** Data & Analytics

## Table of Contents
- [Solution Overview](#solution-overview)
- [Architecture Diagram](#architecture-diagram)
- [Key AWS Services](#key-aws-services)
- [How It Works](#how-it-works)
- [Deploying This Solution](#deploying-this-solution)
- [Learning Outcomes](#learning-outcomes)
- [Cost Considerations](#cost-considerations)
- [Possible Enhancements](#possible-enhancements)
- [License](#license)

## Solution Overview
This project builds a fully serverless data lake on Amazon S3 for a retail use case. Raw transactional data lands in S3 via Kinesis Data Firehose. AWS Glue crawlers infer schemas and populate the Data Catalog, and Glue ETL jobs transform raw JSON into partitioned Parquet format. Amazon Athena queries the transformed data with zero infrastructure to manage, and Amazon QuickSight visualizes KPIs in an executive dashboard using SPICE caching.

## Architecture Diagram
![Architecture Diagram](architecture-diagram.svg)
*Figure 1: Streaming ingestion via Kinesis Firehose into a three-zone S3 data lake, cataloged and transformed by Glue, queried by Athena, and visualized in QuickSight.*

## Key AWS Services
| Service | Role |
|---|---|
| S3 (data lake) | Raw, curated, and aggregated zones; Intelligent Tiering; lifecycle policies |
| Kinesis Data Firehose | Real-time ingestion with buffering, compression, and S3 delivery |
| AWS Glue | Crawlers for schema discovery; Spark ETL jobs; centralized Data Catalog |
| Amazon Athena | Serverless SQL on S3; partitioning and bucketing for cost control |
| AWS Lake Formation | Fine-grained column-level and row-level access control |
| Amazon QuickSight | SPICE-powered dashboards with row-level security |
| EventBridge Scheduler | Triggers Glue jobs on schedule; orchestrates pipeline steps |
| IAM + KMS | Encryption at rest; lake-level and table-level access policies |

## How It Works
1. Simulated retail transaction events stream into Kinesis Data Firehose, which buffers, compresses, and delivers them to the S3 raw zone.
2. A Glue crawler scans the raw zone, infers the schema, and registers it in the Data Catalog.
3. A scheduled Glue ETL (Spark) job transforms the raw JSON into partitioned Parquet files, writing to the curated zone.
4. Athena queries the curated (and further-aggregated) data directly via SQL, using partition filters to minimize the data scanned and control cost.
5. Lake Formation enforces column-level and row-level permissions on top of the catalog, so different users see different slices of the data.
6. QuickSight connects to Athena and caches results in SPICE for fast dashboarding, with row-level security matching the Lake Formation policies.
7. EventBridge Scheduler triggers the Glue job on a recurring schedule to keep the lake up to date.

## Deploying This Solution

### Prerequisites
- An AWS account with console/CLI access
- A QuickSight subscription (for the dashboard step)

### Steps
1. Create the S3 bucket with raw/curated/aggregated prefixes (zones).
2. Set up Kinesis Data Firehose to deliver sample transaction data into the raw zone.
3. Configure a Glue crawler to catalog the raw zone.
4. Write a Glue ETL job to convert JSON to partitioned Parquet in the curated zone.
5. Query the curated data with Athena, verifying partition pruning reduces scan cost.
6. Apply Lake Formation permissions for column/row-level governance.
7. Connect QuickSight to Athena and build a SPICE-backed dashboard.
8. Schedule the Glue job with EventBridge Scheduler.

## Learning Outcomes
- Design a three-zone (raw / curated / aggregated) data lake architecture on S3.
- Ingest streaming data into S3 using Kinesis Data Firehose with inline transformation.
- Use Glue crawlers and ETL jobs to convert JSON into partitioned Parquet format.
- Write Athena SQL with partitioning filters to minimize data scanned and reduce cost.
- Apply Lake Formation to enforce column-level and row-level data governance.
- Build QuickSight dashboards connected to Athena with SPICE in-memory caching.

## Cost Considerations
Glue jobs and QuickSight are the main costs — Glue bills per DPU-hour while jobs run, and QuickSight has a per-user monthly cost (or pay-per-session). Athena is pay-per-query, so keep queries partition-filtered to control spend.

## Possible Enhancements
- Add a Glue Data Quality job to validate incoming records before they reach the curated zone.
- Add near-real-time analytics using Kinesis Data Analytics alongside the batch pipeline.

## License
This project was built for educational purposes as part of an AWS Solutions Architecture project submission.
