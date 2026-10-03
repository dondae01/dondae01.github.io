# Customer Segmentation in SQL (RFM Analysis)

## Summary
I segmented the customers of a UK online retailer by how recently they bought (Recency), how often (Frequency) and how much they spent (Monetary). The main finding: **39% of customers generate 81% of revenue.**

## Data
- **Source:** UCI Online Retail dataset, 1 Dec 2010 to 9 Dec 2011, 38 countries
- **Raw size:** 541,909 transaction lines
- **After cleaning:** 406,829 lines, 22,190 invoices and 4,372 customers (rows with no customer ID removed)

## Results

| Segment | Customers | Share of customers | Average spend | Share of revenue |
|---|---|---|---|---|
| Best Customers | 1,709 | 39.1% | £3,916 | 80.6% |
| Loyal Customers | 1,055 | 24.1% | £1,071 | 13.6% |
| At Risk | 870 | 19.9% | £396 | 4.1% |
| Lost Customers | 738 | 16.9% | £180 | 1.6% |

Total revenue analysed: £8.3M.

### What this means for the business
- **Protect the top segment.** Losing a Best Customer costs about ten times as much as losing an At Risk one.
- **Re-engage At Risk customers.** 870 customers have bought before but not recently, which makes them the cheapest group to win back.
- **Time promotions for Thursday.** More customers made their most recent purchase on a Thursday (1,006) than on any other day; Wednesday was next (771).

## Method

**1. RFM metrics in SQL**
```sql
SELECT CustomerID,
       MIN(JULIANDAY('2011-12-10') - JULIANDAY(InvoiceDate)) AS recency,
       COUNT(*)                                              AS frequency,
       SUM(Quantity * UnitPrice)                             AS monetary
FROM transactions
WHERE CustomerID IS NOT NULL
GROUP BY CustomerID;
```

**2. Ranking customers by spend (window function)**
```sql
SELECT CustomerID, monetary,
       RANK() OVER (ORDER BY monetary DESC) AS spending_rank
FROM (
  SELECT CustomerID, SUM(Quantity * UnitPrice) AS monetary
  FROM transactions
  GROUP BY CustomerID
);
```

**3. Most recent order per customer (window function)**
```sql
SELECT CustomerID, InvoiceNo, InvoiceDate,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY InvoiceDate DESC) AS recent_order
FROM transactions;
```

**4. Scoring and segments.** Each metric is scored 1, 4 or 5 against fixed thresholds (for example, a purchase within 30 days scores 5 on recency). The three scores are added, and the total places each customer in a segment: 12 or more is Best, 9 to 11 is Loyal, 5 to 8 is At Risk, and below 5 is Lost.

## Limitations and next steps
- Frequency counts transaction lines, not invoices, so customers with large baskets score higher. Counting distinct invoices would be more precise.
- Cancellations and returns are included in the totals and should be handled separately.
- The thresholds are fixed. Quintile-based scoring would adapt to the data.

## Tools
SQL (SQLite) · Python (Pandas, Seaborn, Matplotlib) · Google Colab

[Notebook with full code](code/CSA-SQL.ipynb)
