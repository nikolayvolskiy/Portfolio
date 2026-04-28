## RFM анализ клиентской базы


**Цель анализа:**


Разделить клиентов на сегменты, чтобы:


•	выявлять самых ценных покупателей;  

•	возвращать «засыпающих» клиентов;  

•	оптимизировать маркетинговые затраты;  

•	персонализировать коммуникации.  


Что показывают метрики:


•	Recency (давность) — сколько дней прошло с последней покупки. Чем меньше значение, тем выше лояльность.  

•	Frequency (частота) — количество заказов за период. Отражает вовлечённость клиента.  

•	Monetary (денежная ценность) — суммарная стоимость покупок. Показывает вклад клиента в выручку.  


Как работает сегментация:


1.	Для каждой метрики присваиваем балл от 1 до 3:

o	3 — лучший показатель (например, покупал вчера, много заказов, большие траты);

o	2 — средний уровень;

o	1 — слабый показатель (давно не покупал, 1 заказ, малые траты).

2.	Объединяем баллы в код вида RFM (например, 321 — покупал недавно, средняя частота, низкая сумма покупок).

3.	Анализируем распределение клиентов по группам, чтобы понять структуру базы.


Результат


Таблица с:

•	идентификатором клиента;

•	значениями recency, frequency, monetary_value;

•	баллами r, f, m;

•	итоговым RFM кодом;

•	долей каждой группы в общей базе (в %).

```python


import pandas as pd

#Загрузка данных
df_sales = pd.read_csv('C:/Python/t1_main_sales.csv', sep=';', encoding='cp1251', low_memory=False)

#Зададим дату анализа
df_sales['sales_time'] = pd.to_datetime(df_sales['sales_time'], errors='coerce')
analysis_date = pd.to_datetime('2024-12-31')

#Рассчитаем количество дней с покупки до даты анализа
df_sales['order_recency'] = analysis_date - df_sales['sales_time']

#Сгруппируем данные для каждого пользователя и проведем расчеты
rfm_analysis = df_sales.groupby('customer_phone').agg(

    #Количество дней с последнего заказа    
    recency = ('order_recency', lambda x: x.min().days),

    #Количество заказов за период времени
    frequency = ('sales_transaction_id', 'nunique'),

    #Сумма стоимости всех заказов
    monetary_value = ('total_value', 'sum')

).reset_index()

print(rfm_analysis.head(5))

#Создадим копию таблицы rfm_analysis для сегментации
rfm_analysis_2 = rfm_analysis.copy()

#Определим группу пользователя по количеству дней с момента последнего заказа:
recency_min = rfm_analysis_2['recency'].min()
recency_max = rfm_analysis_2['recency'].max()
interval = (recency_max - recency_min) / 3
recency_bins = [recency_min - 1, 
                recency_min + interval, 
                recency_min + 2 * interval, 
                recency_max]

rfm_analysis_2['r'] = pd.cut(rfm_analysis_2['recency'], 
                             bins=recency_bins, 
                             labels=[3, 2, 1], 
                             include_lowest=True)

#Определим группу пользователя по количеству заказов:
#Для частоты используем кастомные границы: 1, 2, 3+ заказов
frequency_bins = [0, 1, 2, float('inf')]
rfm_analysis_2['f'] = pd.cut(rfm_analysis_2['frequency'], bins=frequency_bins, labels=[1, 2, 3], right=False)

#Определим группу пользователя по сумме стоимости заказов:
#Делим пользователей на три равные группы по квантилям (33% и 66%)
rfm_analysis_2['m'] = pd.qcut(rfm_analysis_2['monetary_value'], q=3, labels=[1, 2, 3])

#Создадим групповой RFM-индекс:
rfm_analysis_2[['r', 'f', 'm']] = rfm_analysis_2[['r', 'f', 'm']].astype('str')
rfm_analysis_2['rfm_group'] = rfm_analysis_2['r'] + rfm_analysis_2['f'] + rfm_analysis_2['m']

print(rfm_analysis_2)

#Создаем таблицу с процентным соотношением для каждой RFM группы
rfm_analysis_3 = rfm_analysis_2['rfm_group'].value_counts().reset_index()
rfm_analysis_3.columns = ['rfm_group', 'count']

#Добавляем процентное соотношение
total_users = len(rfm_analysis_2)
rfm_analysis_3['percentage'] = round((rfm_analysis_3['count'] / total_users * 100), 2)

#Сортируем по убыванию
rfm_analysis_3 = rfm_analysis_3.sort_values('count', ascending=False).reset_index(drop=True)

print(rfm_analysis_3)
```
