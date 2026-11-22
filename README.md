# ДЗ 3 Docker

Сделал задания на 8 баллов.

## Запуск

```
cp .env.example .env
sudo docker compose up -d --build --wait
```

В `.env` можно менять настройки. Сейчас там:
```
APP_PORT=8080
GREETING="Hello"
```

Проверить работу
```
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
```
Первый запрос показывает счётчик, второй увеличивает его.

Остановка:
```
sudo docker compose down
```

## Проверки

### 6 баллов

Подключил volume к /data, где лежала база SQLite. Увеличил счётчик до 2, затем удалил и заново создал контейнер:
```
sudo docker compose -f compose.sqlite.yaml down
sudo docker compose -f compose.sqlite.yaml up -d
```

В новом контейнере осталось 2 => данные сохранились.

[До удаления](evidence/06-before.txt) и [после](evidence/06-after.txt).

### 8 баллов

Переключил хранилище на Redis. Проверил сохранение даных:

```
curl -X POST http://localhost:8080/increment
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
sudo docker compose down
sudo docker compose up -d --wait
curl http://localhost:8080/
```

Было 2, после пересоздания тоже 2.

[До](evidence/08-before.txt) и [после](evidence/08-after.txt).

Потом остановил Redis:

```
sudo docker compose stop redis
curl -i http://localhost:8080/healthz
curl -i http://localhost:8080/readyz
curl -i -X POST http://localhost:8080/increment
```

Получилось:
- `/healthz` — 200;
- `/readyz` — 503;
- POST `/increment` — 503.

То есть приложение работает но без базы увеличить счётчик нельзя.

Ответы: [healthz](evidence/08-stopped-healthz.txt), [readyz](evidence/08-stopped-readyz.txt), [increment](evidence/08-stopped-increment.txt).

Запустил Redis обратно:
```
sudo docker compose start redis
sudo docker compose ps
```

Когда Redis стал `healthy`, проверил:
```
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
```

GET вернул 2, POST вернул 3. Приложение не перезапускал.

[После восстановления](evidence/08-restored.txt) и [после увеличения](evidence/08-restored-increment.txt).