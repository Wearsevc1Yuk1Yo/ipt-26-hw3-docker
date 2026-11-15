## Шаблон для третьего ДЗ по ИПР

Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

## Структура
...
## Dockerfile
...
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
Значение count равняется 4м
```
{"greeting": "Hello from my Docker!", "count": 4, "hostname": "959efb23b555", "storage": "sqlite"}
```
Логи:
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
curl http://127.0.0.1:8080/
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
Установим и удалим контейнеры:
```
docker compose -p counter-sqlite -f compose.sqlite.yaml down
```
Запустим заново:
```
docker compose -p counter-sqlite -f compose.sqlite.yaml up -d
```
Проверим счетчик --- состояние не изменилось, так как мы не удаляли именованный volume.:
```
{"greeting": "Hello from Docker Compose! ᓚ₍ ^. .^₎", "count": 3, "hostname": "4ef3423d6375", "storage": "sqlite"}
```