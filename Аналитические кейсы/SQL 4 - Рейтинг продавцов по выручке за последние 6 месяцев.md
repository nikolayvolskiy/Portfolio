## Рейтинг продавцов по выручке за последние 6 месяцев

**Цель анализа:**

- Составить ранжированный список сотрудников по объёму продаж, выявить:

  - лидеров по генерации выручки;

  - различия в продуктивности персонала;

  - вклад каждого продавца в общий доход компании.

**Почему это важно:**

- позволяет объективно оценивать эффективность работы персонала;

- помогает формировать систему мотивации и премий;

- выявляет сотрудников, нуждающихся в дополнительном обучении;

- даёт основу для планирования KPI и развития команды;

- показывает, какие продавцы генерируют наибольшую долю выручки.

**Решение на SQLite:**

Сформируйте рейтинг продавцов по выручке.
```sql
-- Рейтинг продавцов по выручке за последние 6 месяцев

WITH analysis_period AS (
    -- Период: последние 6 месяцев
    SELECT 
        MAX(DATE(sales_time)) as max_date,
        DATE(MAX(DATE(sales_time)), '-6 months') as start_date
    FROM main_sales
),

employee_sales_data AS (
    -- Данные по продажам продавцов
    SELECT 
        e.employee_id,
        e.employee_name,
        SUM(ms.total_value) as total_revenue,
        COUNT(DISTINCT ms.sales_transaction_id) as transaction_count,
        SUM(ms.qty) as total_items_sold
    FROM main_sales ms
    JOIN employees e ON ms.employee_id = e.employee_id
    -- Фильтр по периоду
    WHERE DATE(ms.sales_time) BETWEEN (SELECT start_date FROM analysis_period)
                                  AND (SELECT max_date FROM analysis_period)
    GROUP BY e.employee_id, e.employee_name
    HAVING SUM(ms.total_value) > 0
)

-- Итоговый рейтинг продавцов
SELECT 
    ROW_NUMBER() OVER (ORDER BY total_revenue DESC) as rank_position,
    employee_name,
    ROUND(total_revenue, 2) as total_revenue,
    transaction_count,
    total_items_sold,
    -- Средняя сумма продажи
    ROUND(total_revenue * 1.0 / transaction_count, 2) as avg_sale_amount,
    -- Доля в общей выручке (%)
    ROUND(100.0 * total_revenue / SUM(total_revenue) OVER (), 2) as percent_of_total_revenue
FROM employee_sales_data
ORDER BY total_revenue DESC;
```

**Решение на Postgre:**
```sql
-- Рейтинг продавцов по выручке за последние 6 месяцев

WITH analysis_period AS (
    -- Период: последние 6 месяцев
    SELECT 
        MAX(sales_time::date) as max_date,
        MAX(sales_time::date) - INTERVAL '6 months' as start_date
    FROM main_sales
),

employee_sales_data AS (
    -- Данные по продажам продавцов
    SELECT 
        e.employee_id,
        e.employee_name,
        SUM(ms.total_value) as total_revenue,
        COUNT(DISTINCT ms.sales_transaction_id) as transaction_count,
        SUM(ms.qty) as total_items_sold
    FROM main_sales ms
    JOIN employees e ON ms.employee_id = e.employee_id
    -- Фильтр по периоду
    WHERE ms.sales_time::date BETWEEN (SELECT start_date FROM analysis_period)
                                  AND (SELECT max_date FROM analysis_period)
    GROUP BY e.employee_id, e.employee_name
    HAVING SUM(ms.total_value) > 0
)

-- Итоговый рейтинг продавцов
SELECT 
    ROW_NUMBER() OVER (ORDER BY total_revenue DESC) as rank_position,
    employee_name,
    ROUND(total_revenue::numeric, 2) as total_revenue,
    transaction_count,
    total_items_sold,
    -- Средняя сумма продажи
    CASE 
        WHEN transaction_count > 0 
        THEN ROUND(total_revenue::numeric / transaction_count, 2)
        ELSE 0 
    END as avg_sale_amount,
    -- Доля в общей выручке (%)
    ROUND(100.0 * total_revenue / SUM(total_revenue) OVER (), 2) as percent_of_total_revenue
FROM employee_sales_data
ORDER BY total_revenue DESC;
```

**Как читать результат:**

- rank_position — позиция в рейтинге (1 — лидер);

- employee_name — ФИО продавца;

- total_revenue — выручка за 6 месяцев (руб.);

- transaction_count — количество транзакций за период;

- total_items_sold — общее число проданных товаров;

- avg_sale_amount — средняя сумма одной продажи (руб.);

- percent_of_total_revenue — доля продавца в совокупной выручке компании (%).

**Примеры интерпретации:**

Высокий rank_position + большой percent_of_total_revenue — ключевые продавцы для бизнеса. Рекомендуется:

- премировать за высокие результаты;

- привлекать к обучению менее успешных коллег;

- делегировать сложные задачи и VIP‑клиентов.

Низкий avg_sale_amount при высоком transaction_count — продавец совершает много продаж, но с низкой средней суммой. Возможные действия:

- обучение техникам увеличения чека;

- фокус на продаже высокомаржинальных товаров;

- анализ ассортимента, с которым работает сотрудник.

Высокое total_items_sold при низком total_revenue — продавец продаёт много товаров, но они дешёвые. Стоит:

- пересмотреть распределение товарных групп между сотрудниками;

- мотивировать на продажу премиальных позиций;

- проанализировать, не занижает ли продавец цены.

Резкое падение rank_position по сравнению с прошлыми периодами — сигнал к анализу:

- изменений в нагрузке сотрудника;

- качества обслуживания клиентов;

- конкуренции внутри команды;

- внешних факторов (сезонность, изменения спроса).

**Ключевые выводы:**

1. Топ‑3 продавца генерируют значительную долю выручки — их удержание критично для бизнеса.

2. Продавцы с percent_of_total_revenue <1% — кандидаты на дополнительное обучение.

3. Сравнение avg_sale_amount между сотрудниками помогает выявить лучших практиков и зоны роста.

4. Соотношение transaction_count и total_revenue показывает, за счёт чего достигается результат (объёмы или цена).

5. Рейтинг позволяет справедливо распределять бонусы.
