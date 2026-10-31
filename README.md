# Platform-Engineering-Energy-Consumption-Data-Platform

Build a complete data platform on AWS that ingests French grid data from the eCO2mix API once, stores it in an open format, and serves the same curated tables to Snowflake, Databricks, dashboards and a data API.

The project is a portfolio piece. It is meant to show platform engineering skills:

• Infrastructure as code: everything is provisioned with Terraform modules and deployed through GitHub Actions
• Self-service: a new tenant or dataset is added with one YAML entry and one pull request
• Multi-platform serving: one curated copy of the data, several consumers, none of them owning it
• Governance: per-tenant access enforced in one place
• Reliability and cost control: freshness checks, drift detection, budget alerts and destroyable compute
Scope

A fictional energy consultancy frames the project: it needs trusted, fresh grid data for its analysts, consultants and clients. The company is only a story that gives each design choice a reason to exist.

### In scope:

• eCO2mix national data first (consumption, production by sector, exchanges, CO2 per kWh), regional data as a second step
• Two fictional tenants, for example a data center and an energy producer, to show per-tenant isolation, API keys and cost tagging
• One dashboard and one API endpoint, enough to prove the serving layer
• A README, a decision log and a short demo

### Out of scope:

• Real clients, pricing, contracts and billing
• Forecasting models or optimization algorithms
• Market data and other sources, until the first version works

### Data flow step by step :

Data moves in one direction, from the API to the consumers, and each step hands over to the next through S3 or a trigger.
1. Trigger: EventBridge starts a Step Functions run every 15 minutes.
2. Pull: the run calls a Lambda, which requests from the ODRÉ API only the window since the last successful load, over HTTPS.
3. Land: the Lambda writes the response unchanged to S3 raw, partitioned by load date, so any table can be rebuilt.
4. Transform: a Glue job validates the schema, converts timestamps to UTC and writes curated Parquet tables, then builds the gold tables such as hourly carbon intensity and daily production mix.
5. Check: the run verifies freshness and gaps in the series, and a failure raises a CloudWatch alert.
6. Govern: the Glue Data Catalog registers the tables, and Lake Formation grants each tenant access only to its own slice.
7. Serve: Snowflake and Databricks read the same gold tables from S3, Athena queries them for analysts, dashboards connect to Snowflake or Athena, and the API's Lambda reads them behind API Gateway, which checks each client's key.
Terraform provisions every brick above, and GitHub Actions applies the changes.

### The project is finished when a visitor can see each of these working:

1. Fresh eCO2mix data arrives automatically every 15 minutes, with a visible freshness check
2. The same curated tables are readable from Snowflake, Databricks, Athena and a dashboard
3. One API endpoint returns data for a tenant and rejects a request with a missing or wrong key
4. Adding a tenant takes one YAML entry and one pull request, shown in a short demo
5. Two tenants see only their own slice of the data
6. A pull request shows a plan and a policy scan, and a merge applies the change
7. The compute stacks can be destroyed and rebuilt from GitHub Actions
8. A budget alert and a cost view exist
9. The README states the fictional scenario, links the architecture and explains the main design decisions







