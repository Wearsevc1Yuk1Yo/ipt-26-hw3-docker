# Домашняя работа №3 — Docker
Учебный HTTP-сервис со счётчиком.

## Шаблон для третьего ДЗ по ИПР

Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

## Структура проекта

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

Dockerfile выполняет следующие действия:
1. Выбирает базовый образ Python.
2. Создаёт рабочую директорию `/app`.
3. Копирует `app/requirements.txt`.
4. Устанавливает зависимости через `pip`.
5. Копирует исходный код приложения.
6. Указывает внутренний порт 8000.
7. Запускает приложение командой `python app.py`.

Сборка образа:

```powershell
docker build -t ipt-counter .
```

---

## 1. Увеличение счётчика (один контейнер, 4 балла)

### Запуск контейнера

```powershell
docker run -d --name ipt-counter-test -p 8080:8000 -e "GREETING=Hello from my Docker!" ipt-counter
```

### Проверка работы приложения

Выполняем GET-запрос:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Увеличиваем счётчик четыре раза:

```powershell
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
```

Проверяем итоговое значение:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

После четырёх запросов `POST /increment` значение счётчика равно 4.

Результат:

```json
{"greeting": "Hello from my Docker!", "count": 4, "hostname": "959efb23b555", "storage": "sqlite"}
```

### Логи контейнера

Получаем логи:

```powershell
docker logs ipt-counter-test
```

Результат:

```text
2026-11-15 10:25:00,415 listening port=8000 storage=sqlite
2026-11-15 13:25:45,346 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-15 13:26:04,555 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:18,416 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:19,279 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:26:26,873 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-15 13:39:04,008 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-15 13:40:20,498 http 172.17.0.1 "GET / HTTP/1.1" 200 -
```

---

## 2. Docker Compose и SQLite (данные, 6 баллов)

Для сохранения данных между пересозданиями контейнера используется именованный volume `sqlite_data`, подключённый к `/data`.

### Запуск приложения

Настройки приложения задаются через `.env`. Пример настроек находится в `.env.example`.

Если локальный `.env` ещё не создан, копируем пример:

```powershell
Copy-Item .env.example .env
```

Запускаем сompose:

```powershell
docker compose -p counter-sqlite -f compose.sqlite.yaml up -d --build
```

Проверяем состояние контейнера:

```powershell
docker compose -p counter-sqlite -f compose.sqlite.yaml ps
```

### Проверка счётчика

После первого вызова:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 0, "hostname": "e145f046ca11", "storage": "sqlite"}
```

Увеличиваем счётчик трижды:

```powershell
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
```

Проверяем состояние счётчика:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 3, "hostname": "e145f046ca11", "storage": "sqlite"}
```

### Проверка сохранения данных

Удаляем контейнеры:

```powershell
docker compose -p counter-sqlite -f compose.sqlite.yaml down
```

Запускаем проект заново:

```powershell
docker compose -p counter-sqlite -f compose.sqlite.yaml up -d
```

Проверяем счётчик:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 3, "hostname": "4ef3423d6375", "storage": "sqlite"}
```

```Hostname``` изменился, поскольку контейнер был создан заново, но значение счётчика осталось равным 3.

**Итого:** счётчик сохраняется после удаления и повторного создания контейнера благодаря именованному volume.\
При проверке использовалась команда `docker compose down` без параметра `-v`, поэтому volume не удалялся.

---

## 3. Redis (отдельное хранилище, 8 баллов)

В `compose.yaml` определены два сервиса:

- `app` — Python-приложение;
- `redis` — сервер Redis, хранящий значение счётчика.


### Проверка готовности сервисов

Для Redis используется healthcheck с командой:

```yaml
test: ["CMD", "redis-cli", "ping"]
```

Для приложения проверяется HTTP-адрес `/healthz` с помощью Python.

В `depends_on` приложения указано условие `service_healthy`, поэтому при первоначальном запуске `сompose ожидает` готовности Redis.

### Запуск Redis и приложения

Проверяем конфигурацию:

```powershell
docker compose -p counter-redis config
```

Запускаем оба сервиса:

```powershell
docker compose -p counter-redis up -d --build
```

Проверяем состояние контейнеров:

```powershell
docker compose -p counter-redis ps
```

### 3.1. Проверка сохранения данных

Проверяем изначальное значение счётчика:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

На момент проверки новое хранилище Redis содержало:

```text
count = 0
```

Выполняем пять запросов `POST /increment`:

```powershell
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
```

Проверяем счётчик:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "bb8146edf1da", "storage": "redis"}
```

Теперь проверим, что данные хранятся независимо от контейнеров. Удаляем контейнеры и сеть:

```powershell
docker compose -p counter-redis down
```

Создаём контейнеры заново:

```powershell
docker compose -p counter-redis up -d
```

Проверяем состояние сервисов:

```powershell
docker compose -p counter-redis ps
```

После запуска снова проверяем счётчик:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}
```

Счётчик по-прежнему равен 5.

Итого:

- Начальное значение: 0.
- Выполнено POST /increment: 5 раз.
- Значение до удаления контейнеров: 5.
- Значение после повторного запуска: 5.

**Результат: счётчик сохранился. Проверка пройдена.**

### 3.2. Проверка при отключении Redis

Запоминаем текущее значение счётчика:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}
```

Запоминаем `hostname`: `eeb40cae816e`.

Останавливаем Redis:

```powershell
docker compose -p counter-redis stop redis
```

Проверяем работу сервера:

```powershell
curl.exe -i http://127.0.0.1:8080/healthz
```

Получаем:

```text
HTTP/1.0 200 OK
```

Проверяем готовность хранилища:

```powershell
curl.exe -i http://127.0.0.1:8080/readyz
```

Получаем:

```text
HTTP/1.0 503 Service Unavailable
```

Проверяем получение счётчика:

```powershell
curl.exe -i http://127.0.0.1:8080/
```

Получаем:

```text
HTTP/1.0 503 Service Unavailable
```

Проверяем увеличение счётчика:

```powershell
curl.exe -i -X POST http://127.0.0.1:8080/increment
```

Получаем:

```text
HTTP/1.0 503 Service Unavailable
```

**Вывод:** при остановке Redis HTTP-сервер приложения продолжает работать, но операции, требующие обращения к Redis, возвращают ошибку 503.

### 3.3. Проверка восстановления Redis

Запускаем Redis обратно:

```powershell
docker compose -p counter-redis start redis
```

Проверяем состояние сервисов:

```powershell
docker compose -p counter-redis ps
```

После того как Redis станет `healthy`, проверяем готовность приложения:

```powershell
curl.exe -i http://127.0.0.1:8080/readyz
```

Получаем:

```text
HTTP/1.0 200 OK
```

Проверяем сохранённый счётчик:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose!~", "count": 5, "hostname": "eeb40cae816e", "storage": "redis"}
```

Счётчик сохранился, `hostname` остался прежним: `eeb40cae816e`.

Это подтверждает, что контейнер приложения не пересоздавался при восстановлении Redis.

Далее увеличиваем счётчик:

```powershell
curl.exe -fsS -X POST http://127.0.0.1:8080/increment
```

Проверяем новое значение:

```powershell
curl.exe -fsS http://127.0.0.1:8080/
```

Результат:

```json
{"greeting": "Hello from Docker Compose!~", "count": 6, "hostname": "eeb40cae816e", "storage": "redis"}
```

**Вывод:** после восстановления Redis приложение продолжило работать без ручного перезапуска. GET / снова возвращает сохранённое значение счётчика (count = 5), следующий POST увеличивает его до count + 1 = 6.

#### Итоги проверок

- После `docker compose down` и повторного `docker compose up -d` значение счётчика сохранилось.
- После остановки Redis `GET /healthz` возвращал 200.
- При недоступном Redis `GET /readyz`, `GET /` и `POST /increment` возвращали 503.
- После запуска Redis `GET /readyz` снова возвращал 200.
- Счётчик восстановился до предыдущего значения без перезапуска приложения.
- Следующий POST-запрос увеличил счётчик на единицу.