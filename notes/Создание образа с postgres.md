### Основные параметры для создания Docker-образа с базой данных:

1. **Базовый образ**:
   Используются официальные образы PostgreSQL с Docker Hub, например, `postgres:latest` или конкретная версия (например, `postgres:15`).

2. **Экологические переменные (environment variables)**:
   Эти переменные задают основные параметры конфигурации базы данных:
   - `POSTGRES_DB`: Имя базы данных (по умолчанию — `postgres`).
   - `POSTGRES_USER`: Имя пользователя (по умолчанию — `postgres`).
   - `POSTGRES_PASSWORD`: Пароль пользователя (обязательно).
   - `PGDATA`: Директория для хранения данных (по умолчанию — `/var/lib/postgresql/data`).

3. **Том (volume)**:
   Чтобы данные оставались постоянными между перезапуском контейнера, рекомендуется использовать том (volume) для хранения данных базы данных.

4. **Порты**:
   Необходимо открыть порт для доступа к базе данных снаружи контейнера. Для PostgreSQL это обычно порт 5432.

5. **Скрипты инициализации**:
   Если требуется предварительная инициализация базы данных (например, создание схем, таблиц или загрузка данных), это можно сделать с помощью скриптов, расположенных в директории `/docker-entrypoint-initdb.d/`.

### Пример Dockerfile для PostgreSQL:

```dockerfile
FROM postgres:latest

ENV POSTGRES_DB mydatabase
ENV POSTGRES_USER myuser
ENV POSTGRES_PASSWORD mypassword

VOLUME ["/var/lib/postgresql/data"]

COPY init.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
```

### Пример файла `init.sql` для первичной инициализации:

```sql
CREATE DATABASE mydatabase;
GRANT ALL PRIVILEGES ON DATABASE mydatabase TO myuser;
```

### Пример запуска контейнера:

```bash
docker run -d \
  --name postgres-container \
  -e POSTGRES_DB=mydatabase \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -v /host/path/to/dbdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:latest
```

### Итог:

- Образ Docker с базой данных должен быть построен на официальном образе PostgreSQL.
- Используйте экологические переменные для базовой настройки (имя базы данных, пользователь, пароль).
- Используйте том для постоянной сохранности данных.
- Если нужно, добавляйте скрипты инициализации для создания схем и данных.
- Укажите необходимые порты для доступа к базе данных.

Эти параметры помогут вам построить работоспособный Docker-образ с полноценной базой данных.

### Создание образа через docker-compose.yaml

В `docker-compose.yaml` мы не используем инструкцию `FROM`, так как это специфичная команда Dockerfile, которая указывает базовый образ. Вместо этого, в `docker-compose.yaml` мы будем явно указывать образ, который хотим использовать, и его параметры.

Вот как перенести вышеупомянутые настройки в `docker-compose.yaml`:

```python
version: '3.8'
services:
  db:
    image: postgres:latest  # Используем официальный образ Postgres последней версии
    container_name: postgres_container
    restart: always
    env_file:
      - database.env  # Если вы предпочитаете держать переменные окружения в отдельном файле
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - pgdata:/var/lib/postgresql/data  # Постоянное хранение данных в volume
    ports:
      - "5432:5432"  # Экспонируем порт контейнера 5432 на хостовый порт 5432
    networks:
      - backend_net

volumes:
  pgdata:  # Том для хранения данных

networks:
  backend_net:  # Сеть для коммуникаций между сервисами
```

### Объяснение:

- **image**: Указываем официальный образ PostgreSQL.
- **container_name**: Имя контейнера (можете назвать по-своему).
- **restart**: Политика перезапуска контейнера (всегда перезапускать).
- **env_file**: Если хотите хранить переменные окружения в отдельном файле (например, `database.env`).
- **environment**: Передаем переменные окружения для базы данных.
- **volumes**: Используем volume для постоянного хранения данных PostgreSQL.
- **ports**: Открываем порт контейнера (5432) на хостовом порту (также 5432).
- **networks**: Определяем сеть для коммуникации между сервисами (если нужно).

### Пример файла `database.env` (если решили использовать его):

```python
POSTGRES_DB=mydb
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
```

### Итог:

Таким образом, `docker-compose.yaml` полностью заменяет собой инструкции Dockerfile 
для базового образа PostgreSQL, кроме инструкции `COPY`. Если вам нужно скопировать 
скрипт инициализации (например, `init.sql`), это делается в `Dockerfile`, а не 
в `docker-compose.yaml`.