## Шаблон для третьего ДЗ по ИПР

Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)

## Увеличение счётчика (один контейнер)
Проверка (Powershell)
```
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
curl -X POST http://127.0.0.1:8080/increment
curl -X POST http://127.0.0.1:8080/increment
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
curl http://127.0.0.1:8080/
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