# Data Quality Report - Day 1 (June 12, 2026)

## Executive Summary
✅ All Bronze layer tables loaded successfully
✅ Data quality checks passed
⚠️ Some minor data quality observations documented below

## Data Ingestion Summary

### Orders Table
| Metric | Value |
|--------|-------|
| Total Rows | 99,441 |
| Date Range | 2016-10-03 to 2018-10-17 |
| NULL Values in order_id | 0 ✅ |
| NULL Values in customer_id | 0 ✅ |
| Duplicate order_ids | 0 ✅ |

**Inference:** High quality source data, no duplicates detected

### Customers Table
| Metric | Value |
|--------|-------|
| Total Rows | 99,441 |
| States Covered | 27 |
| Customers with NULL state | 0 ✅ |
| Unique customer_ids | 99,441 ✅ |

**Inference:** Good referential integrity

### Products Table
| Metric | Value |
|--------|-------|
| Total Rows | 32,951 |
| Product Categories | 73 |
| Products with NULL weight | 407 |
| NULL weight percentage | 1.2% |

**Inference:** Minor data quality issue - 1.2% of products missing weight. Handle with DEFAULT or COALESCE in Silver layer.

## Issues Identified

### Issue #1: Missing Product Weights
- **Severity:** LOW
- **Row Count Affected:** 407 rows
- **Remediation:** Impute with category average in Silver layer
- **SQL Fix:**
```sql
WITH avg_weights AS (
    SELECT 
        product_category_name,
        AVG(product_weight_g) as avg_weight
    FROM ecommerce_bronze.products
    WHERE product_weight_g IS NOT NULL
    GROUP BY product_category_name
)
SELECT 
    p.*,
    COALESCE(p.product_weight_g, aw.avg_weight) as product_weight_g_imputed
FROM ecommerce_bronze.products p
LEFT JOIN avg_weights aw 
    ON p.product_category_name = aw.product_category_name
```

## Next Steps
1. Silver layer: Handle NULL weights via imputation
2. Silver layer: Implement SCD Type 2 for customer dimension
3. Gold layer: Create fact table with denormalized columns

## Sign-Off
- Date: June 12, 2026
- Checked by: Data Engineer (You)
- Status: APPROVED FOR SILVER LAYER
