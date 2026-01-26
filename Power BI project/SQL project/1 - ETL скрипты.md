# **Преобразования данныех перед загрузкой в базу**
Для корректной работы базы данных нам нужно провести некоторые преобразования данных перед загрузкой. Ниже список необходимых преобразований:
1. В таблицах с продажами и возвратами нужно привести номера телефонов к единому формату.
2. В таблицах, где есть даты, привести даты к формату YYYY-MM-DD HH:MM

## Скрипты для таблиц с продажами:
```python
import pandas as pd
import re

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Сырые файлы/t1_com_sales_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/1 - Основная база данных/t1_com_sales.csv'

def clean_phone_number(phone):
    #Проверяем пустые значения
    if pd.isna(phone) or str(phone).strip() == '':
        return None
    
    phone_str = str(phone)
    
    #Удаляем все нецифровые символы (пробелы, дефисы, скобки и т.д.)
    clean_phone = re.sub(r'[^\d]', '', phone_str)
    
    #Проверяем длину номера
    phone_length = len(clean_phone)
    
    #Нормализация номера
    if 10 <= phone_length <= 11:
        if clean_phone.startswith('8'):
            #Номер начинается с 8, заменяем на 7
            return '7' + clean_phone[1:]
        elif clean_phone.startswith('9') and phone_length == 10:
            #Номер начинается с 9 и состоит из 10 цифр, добавляем 7
            return '7' + clean_phone
        else:
            #Оставляем как есть
            return clean_phone
    else:
        #Некорректная длина, возвращаем очищенный номер
        return clean_phone

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    if 'customer_phone' in df.columns:
        df['customer_phone'] = df['customer_phone'].apply(clean_phone_number)
    
    if 'sales_time' in df.columns:
        df['sales_time'] = pd.to_datetime(
            df['sales_time'], 
            format='%d.%m.%Y %H:%M', 
            errors='coerce'
        ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```

```python
import pandas as pd
import re

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Сырые файлы/t1_main_sales_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/1 - Основная база данных/t1_main_sales.csv'

def clean_phone_number(phone):
    #Проверяем пустые значения
    if pd.isna(phone) or str(phone).strip() == '':
        return None
    
    phone_str = str(phone)
    
    #Удаляем все нецифровые символы (пробелы, дефисы, скобки и т.д.)
    clean_phone = re.sub(r'[^\d]', '', phone_str)
    
    #Проверяем длину номера
    phone_length = len(clean_phone)
    
    #Нормализация номера
    if 10 <= phone_length <= 11:
        if clean_phone.startswith('8'):
            #Номер начинается с 8, заменяем на 7
            return '7' + clean_phone[1:]
        elif clean_phone.startswith('9') and phone_length == 10:
            #Номер начинается с 9 и состоит из 10 цифр, добавляем 7
            return '7' + clean_phone
        else:
            #Оставляем как есть
            return clean_phone
    else:
        #Некорректная длина, возвращаем очищенный номер
        return clean_phone

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    if 'customer_phone' in df.columns:
        df['customer_phone'] = df['customer_phone'].apply(clean_phone_number)
    
    df['sales_time'] = pd.to_datetime(
        df['sales_time'], 
        format='%d.%m.%Y %H:%M', 
        errors='coerce'
    ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```

```python
import pandas as pd

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Сырые файлы/t1_b2b_sales_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/1 - Основная база данных/t1_b2b_sales.csv'

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    df['sales_time'] = pd.to_datetime(
        df['sales_time'], 
        format='%d.%m.%Y %H:%M', 
        errors='coerce'
    ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```

## Скрипты для таблиц с возвратами
```python
import pandas as pd
import re

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Сырые файлы/t1_main_returns_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/1 - Основная база данных/t1_main_returns.csv'

def clean_phone_number(phone):
    #Проверяем пустые значения
    if pd.isna(phone) or str(phone).strip() == '':
        return None
    
    phone_str = str(phone)
    
    #Удаляем все нецифровые символы (пробелы, дефисы, скобки и т.д.)
    clean_phone = re.sub(r'[^\d]', '', phone_str)
    
    #Проверяем длину номера
    phone_length = len(clean_phone)
    
    #Нормализация номера
    if 10 <= phone_length <= 11:
        if clean_phone.startswith('8'):
            #Номер начинается с 8, заменяем на 7
            return '7' + clean_phone[1:]
        elif clean_phone.startswith('9') and phone_length == 10:
            #Номер начинается с 9 и состоит из 10 цифр, добавляем 7
            return '7' + clean_phone
        else:
            #Оставляем как есть
            return clean_phone
    else:
        #Некорректная длина, возвращаем очищенный номер
        return clean_phone

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    if 'customer_phone' in df.columns:
        df['customer_phone'] = df['customer_phone'].apply(clean_phone_number)
    
    df['return_time'] = pd.to_datetime(
        df['return_time'], 
        format='%d.%m.%Y %H:%M', 
        errors='coerce'
    ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    df['sales_time'] = pd.to_datetime(
        df['sales_time'], 
        format='%d.%m.%Y %H:%M', 
        errors='coerce'
    ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```

```python
import pandas as pd
import re

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Сырые файлы/t1_com_returns_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/1 - Основная база данных/t1_com_returns.csv'

def clean_phone_number(phone):
    #Проверяем пустые значения
    if pd.isna(phone) or str(phone).strip() == '':
        return None
    
    phone_str = str(phone)
    
    #Удаляем все нецифровые символы (пробелы, дефисы, скобки и т.д.)
    clean_phone = re.sub(r'[^\d]', '', phone_str)
    
    #Проверяем длину номера
    phone_length = len(clean_phone)
    
    #Нормализация номера
    if 10 <= phone_length <= 11:
        if clean_phone.startswith('8'):
            #Номер начинается с 8, заменяем на 7
            return '7' + clean_phone[1:]
        elif clean_phone.startswith('9') and phone_length == 10:
            #Номер начинается с 9 и состоит из 10 цифр, добавляем 7
            return '7' + clean_phone
        else:
            #Оставляем как есть
            return clean_phone
    else:
        #Некорректная длина, возвращаем очищенный номер
        return clean_phone

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    if 'customer_phone' in df.columns:
        df['customer_phone'] = df['customer_phone'].apply(clean_phone_number)
    
    if 'return_time' in df.columns:
        df['return_time'] = pd.to_datetime(
            df['return_time'], 
            format='%d.%m.%Y %H:%M', 
            errors='coerce'
        ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    if 'sales_time' in df.columns:
        df['sales_time'] = pd.to_datetime(
            df['sales_time'], 
            format='%d.%m.%Y %H:%M', 
            errors='coerce'
        ).dt.strftime('%Y-%m-%d %H:%M:%S')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```

## Скрипты для таблицы трафика
```python
import pandas as pd

#Указываем пути к файлам
INPUT_FILE = 'C:/Users/user/Documents/Николай/Тестовая база данных/Сырые файлы/t1_traffic_butik_unclear.csv'
OUTPUT_FILE = 'C:/Users/user/Documents/Николай/Тестовая база данных/1 - Основная база данных/t1_traffic.csv'

def main():
    #Чтение исходного файла
    print(f"Чтение файла: {INPUT_FILE}")
    df = pd.read_csv(INPUT_FILE, sep=';', dtype=str, encoding='cp1251')
    print(f"Загружено строк: {len(df)}")
    
    df['date'] = pd.to_datetime(
        df['date'], 
        format='%d.%m.%Y', 
        errors='coerce'
    ).dt.strftime('%Y-%m-%d')
    
    print(f"Сохранение в: {OUTPUT_FILE}")
    df.to_csv(OUTPUT_FILE, sep=';', index=False, encoding='cp1251')
    
    print(f"Готово. Преобразовано строк: {len(df)}")

if __name__ == "__main__":
    main()
```
