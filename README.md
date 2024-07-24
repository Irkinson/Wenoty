import time

# Максимальное количество попыток
RETRIES = 3
# Промежуток времени (60 сек), в течение которого мы будем ретраиться
TIMEOUT = 60
# Частота, c которой будем ретраиться
PERIOD = 5

# Функция-декоратор, которая модифицирует переданный ей метод
# В нашем случае модификация заключается в повторении вызовов переданного метода
def retry(max_retries, timeout, period):
    def outer(func):
        def inner(*args, **kwargs):
            # Задаем время, когда необходимо остановить попытки
            end_time = time.time() + timeout
            # Создаем счетчик, с каждой попыткой будем уменьшать его значение на 1
            retries = max_retries
            # Бесконечно крутимся в цикле до тех пор, пока не случится одно из событий: 
            # 1) метод func() выполнится успешно
            # 2) закончится отведенное на ретраи время
            # 3) исчерпаются попытки
            while True:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f'{e}')
                    if time.time() > end_time:
                        raise 'Timeout has expired!'
                    if retries == 1:
                        raise e
                    else:
                        retries -= 1
                        print(f"Attempts left: {retries}")
                        print(f"Sleeping {period} seconds ..." )
                        time.sleep(period)
        return inner
    return outer

# Протестируем наш декоратор на примере падающей функции
@retry(RETRIES, TIMEOUT, PERIOD)
def send_request():
    raise Exception('Code block has failed. This is expected.')

# Тесты
send_request()
