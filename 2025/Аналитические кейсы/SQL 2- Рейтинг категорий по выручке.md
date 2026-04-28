## Рейтинг категорий товаров по выручке (за последние 6 месяцев)


**Цель анализа:**


Составить ранжированный список товарных категорий по объёму выручки, выявить:


- лидеры и аутсайдеры продаж;


- вклад каждой категории в общий доход;


- объёмы продаж в натуральном выражении;


- широту ассортимента внутри категорий.


**Почему это важно:**


- позволяет оптимизировать ассортиментную политику;


- помогает распределять маркетинговые бюджеты в пользу наиболее прибыльных категорий;


- выявляет потенциальные ниши для расширения/сокращения предложения;


- даёт основу для ценообразования и промо‑активностей;


- показывает структуру спроса на уровне товарных групп.


**Решение на Postgre:**

```sql
-- Топ категорий товаров по выручке за последние 6 месяцев

WITH analysis_period AS (
    -- Период: последние 6 месяцев
    SELECT 
        MAX(sales_time::date) as max_date,
        MAX(sales_time::date) - INTERVAL '6 months' as start_date
    FROM main_sales
),

category_sales_data AS (
    -- Данные по продажам категорий
    SELECT 
        c.category_id,
        c.category_name,
       SUM(ms.total_value) as total_revenue,
        SUM(ms.qty) as total_items_sold,
        COUNT(DISTINCT ms.item_id) as unique_items_sold
    FROM main_sales ms
    JOIN items i ON ms.item_id = i.item_id
    JOIN categories c ON i.category_id = c.category_id
    -- Фильтр по периоду
    WHERE ms.sales_time::date BETWEEN (SELECT start_date FROM analysis_period)
                                  AND (SELECT max_date FROM analysis_period)
    GROUP BY c.category_id, c.category_name
    HAVING SUM(ms.total_value) > 0
)

-- Итоговый рейтинг категорий
SELECT 
    ROW_NUMBER() OVER (ORDER BY total_revenue DESC) as rank_position,
    category_name,
    ROUND(total_revenue::numeric, 2) as total_revenue,
    total_items_sold,
    unique_items_sold,
    -- Доля в общей выручке (%)
    ROUND(100.0 * total_revenue / SUM(total_revenue) OVER (), 2) as percent_of_total_revenue
FROM category_sales_data
ORDER BY total_revenue DESC;
```

**Как читать результат:**


- rank_position — позиция в рейтинге (1 — лидер);


- category_name — название категории;


- total_revenue — выручка категории за 6 месяцев (руб.);


- total_items_sold — общее количество проданных товаров в категории;


- unique_items_sold — число уникальных артикулов в продажах;


- percent_of_total_revenue — доля категории в совокупной выручке сети (%).


**Примеры интерпретации:**


- Высокий rank_position + большой percent_of_total_revenue — ключевые категории для бизнеса. Рекомендуется:


  - поддерживать стабильный ассортимент;


  - усиливать маркетинговую поддержку;


  - контролировать маржинальность.


- Низкий percent_of_total_revenue при высоком total_items_sold — категория с высокой оборачиваемостью, но низкой средней ценой. Возможные действия:


  - анализ наценок;


  - поиск способов увеличить средний чек;


**Ключевые выводы:**


1. Топ‑5 категорий формируют основную долю выручки — их стабильность критична для бизнеса.


2. Категории с долей <1% могут быть кандидатами на оптимизацию ассортимента.


3. Соотношение total_items_sold и total_revenue показывает, за счёт чего генерируется доход (объёмы или цены).
