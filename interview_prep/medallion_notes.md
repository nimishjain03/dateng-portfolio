## Medallion Architecture

### Bronze Layer
- Purpose: Store raw data exactly as received from source systems.
- Data format: 
	JSON
	CSV
	Avro
	Parquet
	Raw Delta Tables
- Quality checks: 
	Usually minimal.

	Typical checks:

		File received successfully
		Metadata captured
		Load timestamp added
- Interview Q&A:
	Q: What transformations happen in Bronze?

	A:Very limited transformations. Typically:

		Metadata addition
		Ingestion timestamp
		Source tracking

	Raw data should remain unchanged.

### Silver Layer
- Purpose: Store cleaned, validated, and conformed data.
- Data format:
	Delta Tables
	Partitioned Parquet
- Transformations:
	Data cleansing
	Deduplication
	Null handling
	Schema enforcement
	Data validation
	Standardization
	Source integration
- Interview Q&A:
	Q: Why not directly build dashboards from Bronze?

	A:

	Because Bronze data:

		Contains duplicates
		Has inconsistent schemas
		May contain invalid records
		Requires extensive processing

	Silver provides trusted and cleansed data.

### Gold Layer
- Purpose: Serve analytics, BI, reporting, and machine learning use cases.
- Data format:
	Delta Tables
	Star Schema
	Fact Tables
	Dimension Tables
- Aggregations:
	Daily Sales:

	date	sales
	2026-06-01	$500,000

	Customer Lifetime Value

	Product Revenue

	Monthly Revenue

	Inventory KPIs

- Interview Q&A:
	Q: Why not query Silver directly?

	A:
	You can, but Gold provides:

		Faster performance
		Precomputed aggregations
		Business-friendly schema
		Lower query cost

### Common Mistakes:
1. Performing Heavy Transformations in Bronze

	Wrong:

		Deduplication
		Business logic
		Complex joins

	Bronze should remain raw.

2. Putting Business Aggregations in Silver

	Wrong:

	Creating:

		Revenue KPIs
		Sales summaries
		Executive dashboards

	These belong in Gold.

3. Skipping Silver Layer

Some teams build:

Bronze → Gold

Problems:

	Poor data quality
	Difficult troubleshooting
	Repeated cleansing logic
4. Not Maintaining Data Lineage

Unable to trace:

Gold → Silver → Bronze

This creates audit and compliance issues.

5. Over-Aggregating Gold Tables

Too many precomputed tables:

	Increased storage
	Data duplication
	Maintenance complexity
