# Understanding the Medallion Architecture

## What Is the Medallion Architecture?

The **Medallion Architecture** (also known as "multi-hop" architecture) is a data organization pattern for Databricks Lakehouse. Data flows through three primary layers, each improving quality and usability:

| Layer | Purpose |
|-------|---------|
| Bronze | Raw data ingestion |
| Silver | Cleaned and standardized data |
| Gold | Business-ready analytics |

![Lakehouse architecture diagram](images\databricks_lakehouse.png)

## Why It Matters for Building a Lakehouse

### Clarity & Role Separation
- **Bronze**: Pure ingestion layer
- **Silver**: Data cleaning and conformance
- **Gold**: Business analytics ready

### Resilience & Reprocessing
- Bronze layer preserves raw data
- Enables rebuilding of Silver/Gold layers
- Reduces risk in transformation changes

### Data Quality & Governance
- Schema enforcement in Silver layer
- Deduplication and validation
- Trusted Gold layer for end users

### Scalability & Modularity
- Independent layer evolution
- Optimized transformations
- Flexible BI integration

## Sample Implementation

Here's a practical example using PySpark:

```python
from pyspark.sql.functions import current_timestamp, count, sum

# Bronze Layer - Raw Ingestion
bronze = (spark.read
           .format("csv")
           .option("header", "true")
           .load("/Volumes/main/default/raw_volume/events/")
           .withColumn("_ingest_time", current_timestamp()))

bronze.write.format("delta").mode("overwrite").saveAsTable("bronze.raw_events")

# Silver Layer - Cleaning & Conformance
silver = (spark.table("bronze.raw_events")
           .filter("event_type is not null")
           .dropDuplicates(["event_id"]))

silver.write.format("delta").mode("overwrite").saveAsTable("silver.user_events")

# Gold Layer - Analytics Ready
gold = (spark.table("silver.user_events")
         .groupBy("user_id")
         .agg(count("*").alias("total_events"),
              sum("value").alias("total_value")))

gold.write.format("delta").mode("overwrite").saveAsTable("gold.user_metrics")
```

## Important Notes

### Volume Path Convention
- Format: `/Volumes/<catalog>/<schema>/<volume-name>/<subfolder-or-file>`
- Example: `/Volumes/main/default/raw_volume/events/`

### Access Control
- Volumes inherit Unity Catalog permissions
- Granular read/write access control

### Best Practices
1. Keep Bronze layer transformations minimal
2. Use clear namespace separation (bronze, silver, gold)
3. Plan reprocessing schedules carefully
4. Document data lineage
5. Consider additional layers if needed (pre-bronze, platinum)

---

## Connect with me

**Dilorom Abdullah** 

[LinkedIn](https://www.linkedin.com/in/diloromabdullah)

![LinkedIn QR Code](images\dilorom_linkedin_qr_code.png)

[Medium](https://medium.com/@dilorom) 

![Medium QR Code](images\dilorom_medium_qr_code.png)
