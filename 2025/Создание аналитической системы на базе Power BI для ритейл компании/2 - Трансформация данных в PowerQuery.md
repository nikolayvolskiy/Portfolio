# **Трансформации в Power Query**


## **Основные добавленные столбцы:**


**sales_transaction_id** - уникальный идентификатор транзации. Служит для точных расчетов ключевых показателей, так как в изначальной выгрузке данных номер транзации может дублироваться.


```m

sales_transaction_id = Text.Replace(

    Text.Replace(

        Text.Replace(

            Text.From([sales_time], "ru-RU"),

            ":", ""

        ),

        ".", ""

    ),

    " ", ""

) & Text.PadStart(

    Text.From([sales_transaction_number]),

    5,

    "0"

)

```


**sales_date / return_date** - дата для связи с календарем.


```m

sales_date = Date.From([sales_time])

return_date = Date.From([return_time])

```


**customer_phone_fixed** - стандартизация телефонных номеров, так как вносятся они не в едином формате.


```m

customer_phone_fixed = if [customer_phone] = null then

    null

else

let

    CleanPhone = Text.Remove([customer_phone], {" ", "-", "(", ")", "+", ".", "\", "/", ","}),

    PhoneLength = Text.Length(CleanPhone)

in

    if PhoneLength >= 10 and PhoneLength <= 11 then

        if Text.Start(CleanPhone, 1) = "8" then

            "7" & Text.End(CleanPhone, 10)

        else if Text.Start(CleanPhone, 1) = "9" and PhoneLength = 10 then

            "7" & CleanPhone

        else

            CleanPhone

    else

        CleanPhone

```


**traffic_date** - дата для связи с календарем для таблицы трафика.


```m

traffic_date = Date.From([date])

```


**sales_transaction_id применяется к:**


main_sales,

com_sales,

e_com_sales,

main_returns,

com_returns,

e_com_returns


**sales_date применяется к:**


main_sales,

com_sales,

e_com_sales,

b2b_sales,


**return_date применяется к:**


main_returns,

com_returns,

e_com_returns


**customer_phone_fixed применяется к:**


main_sales,

com_sales,

e_com_sales,

main_returns,

com_returns,

e_com_returns


**traffic_date применяется к:**


traffic_butik


## **Примечания:**


Все трансформации создаются через Power Query Editor.


Для таблиц возвратов используется return_date вместо sales_date.


В b2b_sales применяется только столбец с датой, так как другие данные там отсутствуют.


Стандартизация телефонов обеспечивает единый формат данных во всех таблицах.
