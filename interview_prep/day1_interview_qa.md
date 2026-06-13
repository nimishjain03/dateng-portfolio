# Day 1: Medallion Architecture & Delta Lake Interview Q&A

## Question 1: Explain Medallion Architecture (L2 Difficulty)

**Question:**
"You're designing a data lake for an e-commerce company that gets 100M events daily. Walk me through your architecture."

**Your Answer (2 minutes):**

"I would design a medallion architecture with three layers:

**Bronze Layer:**
- Purpose: Single source of truth for raw data
- Stores data exactly as received from sources (CSV, JSON, API, Kafka)
- Maintains audit columns: _ingestion_timestamp, _source_system
- Uses Delta Lake for ACID guarantees and schema evolution
- Partitioned by ingestion_date for efficient deletion of stale data
- In this e-commerce example: Raw orders, customers, products stored as-is

**Silver Layer:**
- Purpose: Transform and cleanse
- Removes duplicates (ORDER BY customer_id, timestamp LIMIT 1)
- Enforces data types and NULL handling
- Implements slowly changing dimensions (SCD Type 2) for master data
- Stores deduped, type-safe data
- In this example: Deduplicated orders, customer dimension with effective dates

**Gold Layer:**
- Purpose: Business-ready analytics
- Denormalized fact tables (order_id, customer_id, amount, date)
- Aggregated tables (daily revenue by product, customer cohorts)
- Optimized for query speed (Z-ordered by date, product)
- In this example: fact_orders, revenue_daily, customer_segments

**Why this works:**
- Clear separation of concerns
- Easy to debug (which layer broke?)
- Scales well (each layer can be optimized independently)
- Follows Lambda architecture principles
"

---

## Question 2: Why Delta Lake Over Parquet? (L2 Difficulty)

**Question:**
"Your company currently uses Parquet files for data storage. Why migrate to Delta Lake?"

**Your Answer (1.5 minutes):**

"Three critical advantages:

**1. ACID Transactions**
- Problem: With Parquet, if an ETL fails mid-write, you have partial/corrupt data
- Solution: Delta guarantees Atomicity—either ALL data is written or NONE
- Business impact: No data loss, no corruption

**2. Schema Enforcement**
- Problem: Parquet allows writes with mismatched schemas (column drift)
- Solution: Delta rejects writes if schema doesn't match
- Impact: Prevents silent data quality issues

**3. Time Travel / Versioning**
- Problem: Parquet has no version history
- Solution: Delta stores transaction log, can query any point-in-time
```sql
SELECT * FROM orders VERSION AS OF 0  -- Query 10 days ago
```
- Impact: Data recovery, audit trails, debugging

**4. Deletion Vector Support (DV)**
- Problem: Parquet requires full rewrite to delete rows
- Solution: Delta marks rows as deleted without rewrite
- Impact: Fast deletes for GDPR compliance

**In production:** 
- Parquet: ~5% failure rate on 100M row loads (occasional partial writes)
- Delta: 0% failure rate (ACID guarantees)
"

---

## Question 3: Medallion vs Data Mesh (L3 Difficulty)

**Question:**
"Some companies advocate for 'data mesh' architecture instead of medallion. What are the tradeoffs?"

**Your Answer (2 minutes):**

"Good question. They solve different problems:

**Medallion (Centralized):**
- Single source of truth (all raw data in one Bronze)
- Easier to govern (one place to audit)
- One team owns the pipeline
- Good for: Startups, medium companies, single business domain

**Data Mesh (Decentralized):**
- Each team owns their data product
- Domains have their own Bronze/Silver/Gold
- Harder to reconcile across domains
- Good for: Large enterprises (100+ teams), multiple business units

**My recommendation for e-commerce:**
- Start with medallion (simpler to build, debug, scale)
- As company grows, migrate to mesh if you have >10 data teams
- For a startup: Medallion + shared governance layer

**In this project:** Using medallion because it's most common in Senior interviews.
"

---

## Question 4: How to Handle Late-Arriving Data in Bronze? (L3 Difficulty)

**Question:**
"In your e-commerce data lake, a customer's order arrives 3 days late from the source system. How do you handle it?"

**Your Answer (1.5 minutes):**

"Three options, in order of preference:

**Option 1: Immutable Append (Recommended)**
```python
# Append late-arriving data as new row
orders_bronze.write.mode('append').partitionBy('order_date').saveAsTable(...)

# Then in Silver, handle with:
orders_deduplicated = spark.sql('''
    SELECT * 
    FROM ecommerce_bronze.orders
    QUALIFY ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY _ingestion_timestamp DESC) = 1
''')
```
- Pros: Maintains audit trail, immutable, GDPR-friendly
- Cons: Slightly more storage

**Option 2: MERGE + SCD Type 2**
```python
delta_table.merge(
    new_orders,
    'old.order_id = new.order_id'
).whenMatchedUpdate(
    set={'order_status': 'new.order_status', ...}
).whenNotMatchedInsert(...)
```
- Pros: Handles updates elegantly
- Cons: More complex, requires surrogate keys

**Option 3: Overwrite (NOT Recommended)**
```python
# Overwrites entire day—risky!
orders_bronze.write.mode('overwrite').partitionBy('order_date')
```
- Pros: Simple
- Cons: Loses audit trail, if merge fails you lose everything

**My choice:** Option 1 (immutable append) because it's safest for production.
"

---

## Key Takeaways for Day 1

✅ **Understand medallion for any data lake question**
✅ **Know why Delta > Parquet (ACID, versioning, schema)**
✅ **Can explain late-arriving data handling**
✅ **Ready for L2-L3 interview questions**

Next: Day 2-3 will cover SCD Type 2 implementation and performance optimization.

