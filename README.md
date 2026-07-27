# Prediction Fulfillment With View
Модифікувати код з використанням представлення. Вивести частину з підрахунком revenue в окреме представлення з назвою v_surname_view_task. Після цього замінити в коді CTE на представлення.

**Код до модифікації:**

WITH union_table AS (
  SELECT
    s.date AS date,
    SUM(p.price) AS revenue,
    0 AS predict
  FROM `DA.order` o
  JOIN `DA.product` p
    ON o.item_id = p.item_id
  JOIN `DA.session` s
    ON o.ga_session_id = s.ga_session_id
  GROUP BY s.date


  UNION ALL


  SELECT
    rp.date AS date,
    0 AS revenue,
    SUM(rp.predict) AS predict
  FROM `DA.revenue_predict` rp
  GROUP BY rp.date
),


data_table AS (
  SELECT
    date,
    SUM(revenue) AS revenue,
    SUM(predict) AS predict
  FROM union_table
  GROUP BY date
)


SELECT
  date,
  SUM(revenue) OVER (ORDER BY date) AS running_revenue,
  SUM(predict) OVER (ORDER BY date) AS running_predict,
  SAFE_DIVIDE(
    SUM(revenue) OVER (ORDER BY date),
    SUM(predict) OVER (ORDER BY date)
  ) * 100 AS percent
FROM data_table
ORDER BY date;
