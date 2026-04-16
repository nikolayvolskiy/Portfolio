## Рейтинг магазинов по выручке за последние 6 месяцев

**Цель анализа:**

- Составить ранжированный список торговых точек по ключевым финансовым и операционным показателям за последние 6 месяцев, выявить:

- лидеров и аутсайдеров по объёму выручки;

- различия в эффективности работы магазинов;

- факторы, влияющие на доходность точек.

**Почему это важно:**

- позволяет оценить результативность работы каждого магазина;

- помогает распределять ресурсы (персонал, закупки, маркетинг) в пользу наиболее эффективных точек;

- выявляет точки для оптимизации или закрытия;

- даёт основу для постановки KPI и мотивации персонала;

- показывает влияние локации, формата и направления магазина на его доходность.

**Решение на Postgre:**

```sql
-- Рейтинг магазинов по выручке за последние 6 месяцев

WITH analysis_period AS (
    -- Определяем период анализа: последние 6 месяцев
    SELECT 
        MAX(sales_time::date) as max_date,
        MAX(sales_time::date) - INTERVAL '6 months' as start_date
    FROM main_sales
),

store_sales_data AS (
    -- Собираем данные по продажам за период
    SELECT 
        ms.store_id,
        s.store_name,
        s.store_category,
        s.store_direction,
        SUM(ms.total_value) as total_revenue,
        COUNT(DISTINCT ms.sales_transaction_id) as transaction_count,
        SUM(ms.qty) as total_items_sold,
        COUNT(DISTINCT ms.customer_phone) as unique_customers
    FROM main_sales ms
    JOIN stores s ON ms.store_id = s.store_id
    -- Фильтр по периоду
    WHERE ms.sales_time::date BETWEEN (SELECT start_date FROM analysis_period)
                                  AND (SELECT max_date FROM analysis_period)
    GROUP BY ms.store_id, s.store_name, s.store_category, s.store_direction
    HAVING SUM(ms.total_value) > 0
)

-- Формируем итоговый рейтинг
SELECT 
    ROW_NUMBER() OVER (ORDER BY total_revenue DESC) as rank_position,
    store_name,
    store_category,
    store_direction,
    ROUND(total_revenue::numeric, 2) as total_revenue,
    transaction_count,
    total_items_sold,
    unique_customers,
    -- Средний чек
    ROUND(total_revenue::numeric / transaction_count, 2) as avg_check,
    -- Среднее количество товаров в чеке
    ROUND(total_items_sold::numeric / transaction_count, 2) as avg_items_per_check,
    -- Выручка на клиента
    ROUND(total_revenue::numeric / NULLIF(unique_customers, 0), 2) as revenue_per_customer,
    -- Доля в общей выручке
    ROUND(100.0 * total_revenue / SUM(total_revenue) OVER (), 2) as percent_of_total_revenue,
    -- Отставание от лидера в %
    ROUND(100.0 * (MAX(total_revenue) OVER () - total_revenue) / MAX(total_revenue) OVER (), 2) as gap_from_leader_percent
FROM store_sales_data
ORDER BY total_revenue DESC;
```

**Как читать результат:**

- rank_position — позиция в рейтинге (1 — лидер);

- store_name — название магазина;

- store_category — категория магазина (например, «премиум», «эконом»);

- store_direction — направление (например, «центр города», «спальный район»);

- total_revenue — выручка за 6 месяцев (руб.);

- transaction_count — количество транзакций за период;

- total_items_sold — общее число проданных товаров;

- unique_customers — число уникальных покупателей;

- avg_check — средний чек (руб.);

- avg_items_per_check — среднее число товаров в одном чеке;

- revenue_per_customer — выручка, генерируемая одним клиентом (руб.);

- percent_of_total_revenue — доля магазина в совокупной выручке сети (%);

- gap_from_leader_percent — процент отставания от магазина‑лидера по выручке.

**Примеры интерпретации:**

Высокий rank_position + большой percent_of_total_revenue — ключевые точки для бизнеса. Рекомендуется:

- поддерживать текущий уровень сервиса;

- масштабировать успешные практики на другие магазины;

- рассматривать расширение площади/ассортимента.

Низкий avg_check при высоком transaction_count — магазин привлекает много покупателей, но с низкой средней суммой покупки. Возможные действия:

- внедрение кросс‑продаж;

- акции на товары с высокой маржой;

- обучение персонала техникам увеличения чека.

Большой gap_from_leader_percent — значительное отставание от лидера. Стоит проанализировать:

- локацию и проходимость;

- ассортимент и цены;

- работу персонала;

- конкуренцию в районе.

Высокое revenue_per_customer при малом unique_customers — клиенты совершают крупные покупки, но их мало. Рекомендации:

- усиление маркетинга для привлечения новых покупателей;

- программы лояльности для удержания текущих клиентов.

Низкое avg_items_per_check — в чеке мало товаров. Возможные решения:

- размещение сопутствующих товаров у кассы;

- пакетные предложения;

- промо‑акции «купи 2 — получи скидку».

**Ключевые выводы:**

1. Топ‑3 магазина формируют основную долю выручки — их стабильность критична для бизнеса.

2. Магазины с percent_of_total_revenue <1% — кандидаты на оптимизацию или закрытие.

3. Сравнение avg_check и avg_items_per_check помогает выявить точки для роста среднего чека.

4. revenue_per_customer показывает эффективность работы с клиентской базой.

5. gap_from_leader_percent позволяет оценить потенциал роста для отстающих точек.
