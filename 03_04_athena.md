# Big Data, and Analytics

## Athena

It is completely serverless.
Athena is used to query S3 data, perform ad-hoc queries. It's a pay as you consume. You only pay for the data consumed.

**It is a Schema-On-Read** querying service. 
The original data is not modified, but is displayed through a schema you define. The schema translates the data for you, but the original remains untouched.

Supported data formats:
- XML
- JSON
- CSV
- AVRO
- Parquet
- Cloudtrail logs
- VPC Flowlogs

A common setup is source -> athena -> quicksight (for visualization)
**Is ideal for VPC Flowlogs, CloudTrail logs, ELB Logs, cost reports, but also Glue Data Catalog and Web Server Logs.**

There are now connectors to other data sources:
- DynamoDB
- RDS
- MySQL
- Postgres