# Product & Basket Preference Segmentation

## Project Overview

This project analyzes product-level and basket-level purchase behavior to identify demand patterns, repeated customer preferences, and preference-based customer segments.

The analysis focuses on product attributes such as color, sugar level, country, region, vendor, and varietal information. Instead of looking only at total revenue or overall sales volume, the project examines how product attributes appear in purchases and whether these attributes can help explain customer behavior.

The goal is to understand which product characteristics drive demand, how customers combine products inside baskets, and whether repeated attribute patterns can be used as practical signals for segmentation or personalization.

## Business Problem

In e-commerce, customers often make repeated choices around specific product attributes. For example, some customers may consistently buy products from similar countries, colors, sweetness levels, or vendors. These patterns can be useful for product recommendations, CRM segmentation, merchandising, and demand analysis.

However, simple product-level sales totals do not show whether demand is broad or concentrated around specific attributes. They also do not show whether customers repeatedly return to the same product style or explore different product groups over time.

This project answers four main questions:

1. Which product attributes generate the most purchases and revenue?
2. What do customer baskets usually look like in terms of size, revenue, and attribute diversity?
3. Do customers repeatedly purchase products with similar attributes?
4. Can customers be grouped into simple preference-based segments?

## Dataset

The analysis uses anonymized e-commerce event and product data.

Main fields used:

* `client_id`
* `session_id`
* `event_time`
* `event_type`
* `product_id`
* `price`
* `product_category`
* `country`
* `region`
* `color`
* `sugar`
* `vendor`
* `varietals`

Raw business data is not included in this repository because it is based on anonymized real-world commercial data.

## Tools Used

* SQL
* DuckDB
* Python / pandas
* Tableau
* Data cleaning
* Product attribute analysis
* Basket-level aggregation
* Customer segmentation
* Business-oriented data interpretation

## Key Findings

*To be filled after analysis.*

* Finding 1
* Finding 2
* Finding 3
* Finding 4

## Analytical Approach

The project is divided into four analysis blocks:

1. Product attribute demand mix
2. Basket composition analysis
3. Repeat preference detection
4. Preference-based customer segmentation

---

# Analysis 1: Product Attribute Demand Mix

## Goal

The goal of this analysis is to identify which product attributes generate the most purchases, revenue, and customer reach.

This block focuses on product-level demand by attributes such as `color`, `sugar`, `country`, `region`, and `vendor`. The purpose is to understand which attribute groups dominate sales and whether demand is concentrated around a small number of product characteristics.

## Business Question

Which product attributes contribute most to purchases and revenue, and how concentrated is demand across these attributes?

## SQL Logic

This query transforms several product attributes into one unified attribute table and calculates demand metrics for each attribute value.  
The result shows purchases, revenue, customer reach, product variety, average price, and revenue share for the top attribute values inside each attribute group.

<details>
<summary>View SQL query</summary>

```sql
WITH purchase_events AS (
    SELECT
        client_id,
        session_id,
        product_id,
        price,
        country,
        region,
        color,
        sugar,
        vendor
    FROM events
    WHERE event_type = 'purchase'
      AND product_category = 'wine'
      AND client_id IS NOT NULL
      AND session_id IS NOT NULL
      AND product_id IS NOT NULL
      AND price IS NOT NULL
),

attribute_events AS (
    SELECT
        'color' AS attribute_type,
        color AS attribute_value,
        client_id,
        session_id,
        product_id,
        price
    FROM purchase_events
    WHERE color IS NOT NULL

    UNION ALL

    SELECT
        'sugar' AS attribute_type,
        sugar AS attribute_value,
        client_id,
        session_id,
        product_id,
        price
    FROM purchase_events
    WHERE sugar IS NOT NULL

    UNION ALL

    SELECT
        'country' AS attribute_type,
        country AS attribute_value,
        client_id,
        session_id,
        product_id,
        price
    FROM purchase_events
    WHERE country IS NOT NULL

    UNION ALL

    SELECT
        'region' AS attribute_type,
        region AS attribute_value,
        client_id,
        session_id,
        product_id,
        price
    FROM purchase_events
    WHERE region IS NOT NULL

    UNION ALL

    SELECT
        'vendor' AS attribute_type,
        vendor AS attribute_value,
        client_id,
        session_id,
        product_id,
        price
    FROM purchase_events
    WHERE vendor IS NOT NULL
),

attribute_stats AS (
    SELECT
        attribute_type,
        attribute_value,
        COUNT(*) AS purchases,
        ROUND(SUM(price), 2) AS revenue,
        COUNT(DISTINCT client_id) AS unique_customers,
        COUNT(DISTINCT session_id) AS unique_sessions,
        COUNT(DISTINCT product_id) AS unique_products,
        ROUND(AVG(price), 2) AS avg_price
    FROM attribute_events
    GROUP BY attribute_type, attribute_value
),

attribute_shares AS (
    SELECT
        attribute_type,
        attribute_value,
        purchases,
        revenue,
        unique_customers,
        unique_sessions,
        unique_products,
        avg_price,

        ROUND(
            purchases * 1.0
            / SUM(purchases) OVER (PARTITION BY attribute_type),
            4
        ) AS purchase_share,

        ROUND(
            revenue * 1.0
            / SUM(revenue) OVER (PARTITION BY attribute_type),
            4
        ) AS revenue_share,

        ROW_NUMBER() OVER (
            PARTITION BY attribute_type
            ORDER BY revenue DESC
        ) AS attribute_rank
    FROM attribute_stats
)

SELECT
    attribute_type,
    attribute_value,
    purchases,
    revenue,
    unique_customers,
    unique_sessions,
    unique_products,
    avg_price,
    purchase_share,
    revenue_share,
    attribute_rank
FROM attribute_shares
WHERE attribute_rank <= 10
ORDER BY attribute_type ASC, attribute_rank ASC;
```

</details>

## Result

*Result table will be added here.*

| attribute_type | attribute_value | purchases | revenue | unique_customers | avg_price | revenue_share |
| -------------- | --------------- | --------: | ------: | ---------------: | --------: | ------------: |
| color          | ...             |       ... |     ... |              ... |       ... |           ... |
| sugar          | ...             |       ... |     ... |              ... |       ... |           ... |
| country        | ...             |       ... |     ... |              ... |       ... |           ... |

## Visualization

![Product Attribute Demand Mix](images/product_attribute_demand_mix.png)

## Local Conclusion

*To be filled after analysis.*

This analysis should explain which product attributes dominate demand, whether revenue is concentrated around a small number of attribute values, and whether the same attributes also attract a broad customer base.

The conclusion should distinguish between high-revenue attributes and broadly purchased attributes. An attribute can generate high revenue because it has many buyers, because it has higher average prices, or because a small number of customers purchase expensive products from that group.
