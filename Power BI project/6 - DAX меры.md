1. Расчёт средней цены покупки
```dax
avg_price = [total_sales_revenue_2]/[total_sales_qty_2]
```

2. Расчёт среднего чека
```dax
avg_spend = [total_sales_revenue_2]/[total_transactions] 
```

3. Расчёт средней корзины
```dax
avg_unit = [total_sales_qty_2]/[total_transactions]
```

4. Определение размера квартальной когорты
```dax
cohort_size = 
CALCULATE(
    DISTINCTCOUNT(cohort_analysis_base[customer_phone_fixed]),
    cohort_analysis_base[sales_quarter] = cohort_analysis_base[first_purchase_quarter]
) 
```

5. Расчёт конверсии
```dax
conversion = [total_transaction_0001]/SUM(traffic_0001[traffic_average])
```

6. Расчёт показателя LTV
```dax
ltv = 
CALCULATE(
    SUM(ltv_analysis[revenue]),
    FILTER(
        ALL(ltv_analysis[quarter]),
        ltv_analysis[quarter] <= MAX(ltv_analysis[quarter])
    ),
    NOT(ISBLANK(ltv_analysis[customer]))
) / CALCULATE(
    DISTINCTCOUNT(ltv_analysis[customer]),
    ltv_analysis[quarter] = MAX(ltv_analysis[cohort]),
    NOT(ISBLANK(ltv_analysis[customer]))
) 
```

7. Расчёт количества новых клиентов
```dax
new_customers_count = CALCULATE(
    DISTINCTCOUNT(all_sales_2[customer_phone_fixed]),
    all_sales_2[is_first_purchase] = "Новый клиент",
    all_sales_2[customer_phone_fixed] <> BLANK()
) 
```

8. Расчёт доли новых клиентов
```dax
new_customers_ratio = DIVIDE(
    [new_customers_count],
    [total_customers],
    0
) 
```

9. Расчёт показателя Retention rate
```dax
retention_rate = 
VAR cohort_q = SELECTEDVALUE(cohort_analysis_base[first_purchase_quarter])
VAR activity_q = SELECTEDVALUE(cohort_analysis_base[sales_quarter])
VAR base_size = 
    CALCULATE(
        DISTINCTCOUNT(cohort_analysis_base[customer_phone_fixed]),
        cohort_analysis_base[first_purchase_quarter] = cohort_q,
        cohort_analysis_base[sales_quarter] = cohort_q
    )
VAR retained = 
    CALCULATE(
        DISTINCTCOUNT(cohort_analysis_base[customer_phone_fixed]),
        cohort_analysis_base[first_purchase_quarter] = cohort_q,
        cohort_analysis_base[sales_quarter] = activity_q
    )
RETURN
IF(
    base_size > 0 && cohort_q <> activity_q,
    DIVIDE(retained, base_size, 0)
)
``` 
10. Расчёт пропорции возвратов относительно выручки
```dax
return_ratio = [total_returns_amount]/[total_sales_revenue]
```

11. Расчёт количества вернувшихся покупателей
```dax
returning_customers_count = CALCULATE(
    DISTINCTCOUNT(all_sales_2[customer_phone_fixed]),
    all_sales_2[is_first_purchase] = "Старый клиент",
    all_sales_2[customer_phone_fixed] <> BLANK()
) 
```

12. Расчёт общего количества покупателей (важно считать отдельно, так как нельзя просто суммировать старых и новых)
```dax
total_customers = CALCULATE(
    DISTINCTCOUNT(all_sales_2[customer_phone_fixed]),
    all_sales_2[customer_phone_fixed] <> BLANK()
)
```

13. Расчёт суммы возвратов
```dax
total_returns_amount = SUM(main_returns[total_value])+SUM(com_returns[total_value])+SUM(e_com_returns[total_value]) 
//Направление b2b_sales не участвует так как в ней нем бывает возвратов
```

14. Расчёт количества возвращенных товаров
```dax
total_returns_qty = 
CALCULATE(
    SUM(main_returns[qty]) +
    SUM(com_returns[qty]) +
    SUM(e_com_returns[qty]),
    FILTER(
        items,
        RELATED(categories[category_name]) <> "Услуги" &&
        RELATED(categories[category_name]) <> "Упаковка"
    )
//Вспомогательная мера для расчетов средних показателей, чтобы услуги и упаковка, а так же продажи b2b не искажали общие данные
```
```dax
total_returns_qty_3 = SUM(main_returns[qty]) + SUM(com_returns[qty]) + SUM(e_com_returns[qty])
//Для подсчета количество товаров по категориям и артикулам, чтобы так же можно было смотреть услуги и упаковку (вспомогательная формула)
//Направление b2b_sales не участвует так как в ней нем бывает возвратов
```

15. Расчёт продаж по формуле (выручка - возвраты)
```dax
total_sales_amount = [total_sales_revenue]-[total_returns_amount]
```

16. Расчёт проданного количества товаров по всем каналам для общей таблицы
```dax
total_sales_qty = 
CALCULATE(
    SUM(main_sales[qty]) +
    SUM(com_sales[qty]) +
    SUM(e_com_sales[qty]) +
    SUM(b2b_sales[qty]),
    FILTER(
        items,
        RELATED(categories[category_name]) <> "Услуги" &&
        RELATED(categories[category_name]) <> "Упаковка"
    )
)
//Для отображения в общей таблице
```
```dax
total_sales_qty_2 = CALCULATE(SUM(main_sales[qty])+
                            SUM(com_sales[qty])+
                            SUM(e_com_sales[qty]),
                            FILTER(
                            items,
                            RELATED(categories[category_name]) <> "Услуги" &&
                            RELATED(categories[category_name]) <> "Упаковка"
    )
)
//Вспомогательная мера для расчетов средних показателей, чтобы услуги и упаковка, а так же продажи b2b не искажали общие данные            
```
```dax
total_sales_qty_3 = 
SUM(main_sales[qty]) +
SUM(com_sales[qty]) +
SUM(e_com_sales[qty]) +
SUM(b2b_sales[qty])
// Для отображения количества товаров по категориям и артикулам, чтобы так же отражались услуги и упаковка
```

17. Расчёт количества проданных товаров за вычетом возвратов для общей таблицы
```dax
total_sales_qty_with_returns = [total_sales_qty]-[total_returns_qty]
```

18. Расчёт количества проданных товаров для детализованной таблицы (с категориями)
```dax
total_sales_qty_with_returns_3 = [total_sales_qty_3]-main_returns[total_returns_qty_3]
```

19. Расчёт общей выручки по всем каналам: для основной таблицы
```dax
total_sales_revenue = SUM(main_sales[total_value])+SUM(com_sales[total_value])+SUM(e_com_sales[total_value])+SUM(b2b_sales[total_value])
```

20. Расчёт общей выручки (вспомогательная мера для подсчёта средних показателей)
```dax
total_sales_revenue_2 = SUM(main_sales[total_value])+SUM(com_sales[total_value])+SUM(e_com_sales[total_value])
```

21. Расчёт количества чеков только в одной точке продаж (по которой доступны данные по трафику
```dax
total_transaction_0001 = CALCULATE(DISTINCTCOUNT(main_sales[sales_transaction_id]),
                          stores[store_name] = "0001"
```

22. Количество транзакций
```dax
total_transactions = CALCULATE(DISTINCTCOUNT(main_sales[sales_transaction_id]) + 
                               DISTINCTCOUNT(com_sales[sales_transaction_id]) +
                               DISTINCTCOUNT(e_com_sales[sales_transaction_id])
) -- количество транзакций (чеков)   
```
