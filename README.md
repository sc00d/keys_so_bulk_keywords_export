```import requests
import pandas as pd
import time
from urllib.parse import quote

# Параметры API
API_TOKEN = "ВАШ_КЛЮЧ"
BASE_URL = "https://api.keys.so/report/simple/organic/keywords"
DOMAIN = "blog.domclick.ru" # адрес сайта, который анализируем
BASE = "msk" # база, по которой выгружаются ключи
PER_PAGE = 50000 # сколько ключей получать за 1 запрос
FILTER = "pos%253C11"  # фильтрация, в примере - позиция <11. Фильтр можно собрать в интерфейсе, например, ну или по документации API
SORT = "pos|asc"

# Заголовки запроса
headers = {
    "X-Keyso-TOKEN": API_TOKEN
}

# Функция для получения данных с API
def fetch_data(page):
    params = {
        "base": BASE,
        "domain": DOMAIN,
        "sort": SORT,
        "page": page,
        "per_page": PER_PAGE,
        "filter": FILTER
    }
    
    try:
        response = requests.get(BASE_URL, headers=headers, params=params)
        response.raise_for_status()  # Проверка на ошибки HTTP
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Ошибка при запросе данных на странице {page}: {e}")
        return None

# Основной код для сбора данных
def main():
    all_data = []
    page = 1

    while True:
        print(f"Получение данных со страницы {page}...")
        data = fetch_data(page)
        
        if data is None:
            print("Прерывание из-за ошибки запроса.")
            break
            
        # Проверяем, есть ли данные в ответе
        if "data" in data and data["data"]:
            all_data.extend(data["data"])
            page += 1
            time.sleep(1)  # Задержка для избежания ограничений API
        else:
            print("Данные не получены или пустые. Завершение сбора.")
            break

    # Преобразование в DataFrame
    if all_data:
        df = pd.DataFrame(all_data)
        # Сохранение в CSV
        output_file = f"keys_so_data_{DOMAIN}.csv"
        df.to_csv(output_file, index=False, encoding='utf-8')
        print(f"Данные успешно сохранены в {output_file}. Всего записей: {len(df)}")
    else:
        print("Нет данных для сохранения.")

# Запуск
if __name__ == "__main__":
    main()```
