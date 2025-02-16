---
title: 当日データ
---


```sql daily_data
  SELECT
    t.report_date AS today,
    SUM(t.cost) AS today_cost,
    SUM(t.cv) AS today_cv,
    today_cost / today_cv AS today_cpa,
    SUM(t.sales) AS today_sales,
    DATE_TRUNC('month', t.report_date) AS month,
    today_cost / ANY_VALUE(y.yesterday_cost) - 1 AS cost_growth,
    today_cpa / ANY_VALUE(y.yesterday_cv) - 1 AS cpa_growth,
    today_sales / ANY_VALUE(y.yesterday_sales) - 1 AS sales_growth
  FROM
    ad_reports.ad_daily AS t
  LEFT JOIN (
    SELECT
      report_date AS yesterday,
      SUM(cost) AS yesterday_cost,
      SUM(cv) AS yesterday_cv,
      yesterday_cost / yesterday_cv AS yesterday_cpa,
      SUM(sales) AS yesterday_sales,
    FROM
      ad_reports.ad_daily
    where report_date = DATE_ADD(CURRENT_DATE, INTERVAL (-1) DAY)
    GROUP BY report_date
  ) AS y
  ON DATE_ADD(t.report_date, INTERVAL (-1) DAY) = y.yesterday
  WHERE
    t.report_date <= CURRENT_DATE
    AND t.report_date >= DATE_ADD(CURRENT_DATE, INTERVAL (-1) WEEK)
  GROUP BY t.report_date
  ORDER BY t.report_date DESC
  ;
```

<BigValue
  title="Cost"
  data={daily_data}
  value=today_cost
  fmt='jpy0'
  sparkline=today
  sparklineType=area
  downIsGood=true
  comparison=cost_growth
  comparisonFmt=pct1
  comparisonTitle="前日対比"
/>
<BigValue 
  title="CPA"
  data={daily_data} 
  value=today_cpa
  fmt='jpy0'
  sparkline=today
  sparklineType=area
  downIsGood=true
  comparison=cpa_growth
  comparisonFmt=pct1
  comparisonTitle="前日対比"
/>
<BigValue 
  title="Sales"
  data={daily_data} 
  value=today_sales
  fmt='jpy0'
  sparkline=today
  sparklineType=area
  downIsGood=false
  comparison=sales_growth
  comparisonFmt=pct1
  comparisonTitle="前日対比"
/>