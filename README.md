## Шаблон для третьего ДЗ по ИПР

Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

## Структура

```text
ipt-26-hw3-docker/
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── README.md
│
├── Dockerfile
├── .dockerignore
├── .gitignore
├── .env.example
│
├── compose.sqlite.yaml
├── compose.yaml
│
└── README.md
```
Сборка образа:

```powershell
docker build -t ipt-counter .
```
## 1. Увеличение счётчика (один контейнер)
Проверка (Powershell)
```
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/
```

После четырёх запросов ```POST /increment``` значение счётчика равно 4.
```
{"greeting": "Hello from my Docker!", "count": 4, "hostname": "959efb23b555", "storage": "sqlite"}
```
__Логи (ответ):__
```
2026-11-15 10:25:00,415 listening port=8000 storage=sqlite
2026-11-15 13:25:45,346 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-15 13:26:04,555 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:18,416 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:19,279 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:26,873 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-15 13:39:04,008 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:40:20,498 http 172.17.0.1 "GET / HTTP/1.1" 200 -
```
## 2. Docker Compose и SQLite (данные)
После первого вызова :
```
curl -fsS http://127.0.0.1:8080/
```
```
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 0, "hostname": "e145f046ca11", "storage": "sqlite"}
```
Увеличим счетчик трижды:
```
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
```
Состояние счётчика:
```
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 3, "hostname": "e145f046ca11", "storage": "sqlite"}
```
----
Установим и удалим контейнеры:
```
docker compose -p counter-sqlite -f compose.sqlite.yaml down
```
Запустим заново:
```
docker compose -p counter-sqlite -f compose.sqlite.yaml up -d
```
Проверим счетчик - состояние не изменилось, так как мы не удаляли именованный volume.:
```
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 3, "hostname": "4ef3423d6375", "storage": "sqlite"}
```
__Так, счётчик сохраняется после удаления-создания контейнеров.__

## 3. Redis (отдельное хранилище)
### Проверка сохранения данных

Проверяем изначальный count: 
```
         curl.exe -fsS http://127.0.0.1:8080/

                           ↓

                 {..., "count": 0, ...}
```
Сделаем 5 запросов POST, проверим счётчик:
 ```
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS http://127.0.0.1:8080/

                           ↓

{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "bb8146edf1da", "storage": "redis"}
```
Теперь проверим, что данные хранятся независимо от контейнеров: удалим и создадим их заново.
```
docker compose -p counter-redis down
docker compose -p counter-redis up -d
```
Далее проверяем счётчик:
 ```
curl.exe -fsS http://127.0.0.1:8080/

                           ↓

{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}
```
Счётчик по-прежнему равен 5 (count остался прежним).\
Итого:

    Начальное значение: 0
    Выполнено POST /increment: 5
    Значение до удаления контейнеров: 5
    Значение после повторного запуска: 5

    Результат: счётчик сохранился.

__Проверка пройдена__

### Проверка после восстановления
Проверка восстановления данных после отключения Redis.

Запоминаем текущий счётчик:
 ```
curl.exe -fsS http://127.0.0.1:8080/

                           ↓

{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}
```
Запомним, что ```hostname``` равен ```eeb40cae816e```. __*'__\
__Останавливаем redis:__
```
docker compose -p counter-redis stop redis
```
При проверке ```curl.exe -i http://127.0.0.1:8080/healthz```
получаем  ```HTTP/1.0 200 OK```

При проверке ```curl.exe -i http://127.0.0.1:8080/readyz``` получаем ```HTTP/1.0 503 Service Unavailable```.

Проверяем GET/:\
```curl.exe -i http://127.0.0.1:8080/``` -> ```HTTP/1.0 503 Service Unavailable```.

Проверяем POST/:\
```curl.exe -i -X POST http://127.0.0.1:8080/increment``` -> ```HTTP/1.0 503 Service Unavailable```

__Запустим redis:__\
```docker compose -p counter-redis start redis```

Теперь:\
```curl.exe -i http://127.0.0.1:8080/readyz``` -> ```HTTP/1.0 200 OK```  (мы снова можем обращаться к Redis).

Текущий счётчик сохранился.\
```curl.exe -fsS http://127.0.0.1:8080/``` -> ```{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}```

__'*__ ```hostame``` по-прежнему ```eeb40cae816e```, как и до ```docker compose stop redis```, значит, действительно, приложение не перезапускалось.

Далее проверим:
```
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS http://127.0.0.1:8080/
```
Результат:
```
{"greeting": "Hello from Docker Compose!~", "count": 6, "hostname": "eeb40cae816e", "storage": "redis"}
```
__Так, пройдены проверки:__
* Без ручного перезапуска приложения GET / снова возвращает ```count```, следующий POST — ```count + 1```.
* После остановки ```docker compose stop redis``` приложение продолжает работать: ```GET /healthz ```возвращает 200, а ```GET /readyz``` и ```POST /increment``` — 503.