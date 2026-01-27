## Анализ возвращаемости клиентов (Retention) за 2024 год
**Цель анализа:**

Оценить, насколько успешно компания удерживает клиентов после первой покупки. Метрика Retention Rate показывает процент клиентов, совершивших повторные покупки в последующие месяцы после привлечения.

**Ключевые понятия:**

•	Когорта — группа клиентов, совершивших первую покупку в одном месяце.  
•	Retention Rate — доля клиентов из когорты, вернувшихся в последующие месяцы (в %).  
•	Матрица retention — таблица, где строки — когорты, столбцы — месяцы, значения — процент возвращаемости.  

**Почему это важно:**  

•	Выявляет «слабые» месяцы привлечения (низкий retention).  
•	Помогает оценить эффективность программ лояльности.  
•	Позволяет прогнозировать доход от существующих клиентов.  

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

#Загрузка данных
df_sales = pd.read_csv('C:/Python/t1_main_sales.csv', sep=';', encoding='cp1251', low_memory=False)

#Преобразование даты
df_sales['sales_time'] = pd.to_datetime(df_sales['sales_time'], errors='coerce')

#Фильтрация за 2024 год
sales_2024 = df_sales[df_sales['sales_time'].dt.year == 2024].copy()

#Создаем колонки с месяцем
sales_2024['month'] = sales_2024['sales_time'].dt.month

#Находим первый месяц покупки для каждого покупателя в 2024 году
first_purchase = sales_2024.groupby('customer_phone')['month'].min().reset_index()
first_purchase.columns = ['customer_phone', 'cohort_month']

#Объединяем с исходными данными
sales_with_cohort = sales_2024.merge(first_purchase, on='customer_phone', how='left')

#Создаем когорты (группируем по когортному месяцу и месяцу покупки)
cohort_data = sales_with_cohort.groupby(['cohort_month', 'month']).agg(
    n_customers=('customer_phone', 'nunique')
).reset_index()

#Создаем сводную таблицу
cohort_pivot = cohort_data.pivot_table(
    index='cohort_month',
    columns='month',
    values='n_customers',
    fill_value=0
)

#Заполняем отсутствующие месяцы
all_months = range(1, 13)
cohort_pivot = cohort_pivot.reindex(index=all_months, columns=all_months, fill_value=0)

#Вычисляем retention rate
retention_matrix = pd.DataFrame(index=cohort_pivot.index, columns=cohort_pivot.columns)

for cohort in cohort_pivot.index:
    cohort_size = cohort_pivot.loc[cohort, cohort]  #количество клиентов в месяц когорты
    if cohort_size > 0:
        for month in cohort_pivot.columns:
            if month >= cohort:  #только текущий и последующие месяцы
                retention_value = (cohort_pivot.loc[cohort, month] / cohort_size) * 100
            else:
                retention_value = 0
            retention_matrix.loc[cohort, month] = retention_value
    else:
        for month in cohort_pivot.columns:
            retention_matrix.loc[cohort, month] = 0

#Преобразуем в float и округляем значения
retention_matrix = retention_matrix.astype(float).round(1)

#Преобразуем названия месяцев
month_names = ['Янв', 'Фев', 'Мар', 'Апр', 'Май', 'Июн', 
               'Июл', 'Авг', 'Сен', 'Окт', 'Ноя', 'Дек']

retention_matrix.index = [month_names[i-1] for i in retention_matrix.index]
retention_matrix.columns = [month_names[i-1] for i in retention_matrix.columns]

#Выводим матрицу retention
print("Матрица Retention за 2024 год (в процентах)")
print("Когорта по вертикали, месяц по горизонтали")
print()
print(retention_matrix)

#Визуализация - тепловая карта
plt.figure(figsize=(14, 10))

#Создаем тепловую карту
sns.heatmap(retention_matrix, 
            annot=True, 
            fmt='.1f', 
            cmap='RdYlGn', 
            vmin=0, 
            vmax=100,
            linewidths=0.5,
            linecolor='gray',
            cbar_kws={'label': 'Retention Rate %'})

plt.title('Матрица Retention по месяцам 2024 года. Процент возвращаемости клиентов', 
          fontsize=16, fontweight='bold', pad=20)
plt.xlabel('Месяц', fontsize=12)
plt.ylabel('Когорта (месяц привлечения)', fontsize=12)
plt.xticks(rotation=0)
plt.yticks(rotation=0)
plt.tight_layout()
plt.show()
```

Что делает код:
1.	Подготовка данных
-	Загружает продажи и фильтрует записи за 2024 год.  
-	Извлекает месяц покупки из даты.  
3.	Формирование когорт
-	Для каждого клиента определяет месяц первой покупки (cohort_month).  
-	Связывает каждую покупку с месяцем привлечения клиента.  
4.	Расчет retention
-	Считает уникальных клиентов в каждой когорте по месяцам.  
-	Для каждой когорты вычисляет процент вернувшихся в последующие месяцы.  
-	Заполняет матрицу нулями для месяцев до привлечения.  
5.	Визуализация
-	Строит тепловую карту с цветовой градацией (красный → жёлтый → зелёный).  
-	Подписывает значения в ячейках с точностью до 1 знака после запятой.  
-	Добавляет легенду с диапазоном от 0 % до 100 %.  

Как читать результат:
-	Строки — месяцы привлечения клиентов (когорты).  
-	Столбцы — месяцы, в которых проверялась возвращаемость.  
-	Значение в ячейке — процент клиентов из когорты, совершивших покупку в этом месяце.  
-	Цвет — чем зеленее, тем выше retention.  

Пример интерпретации:
Если в строке «Янв» значение для «Мар» равно 45,0 %, это значит, что 45 % клиентов, впервые купивших в январе, вернулись в марте.
