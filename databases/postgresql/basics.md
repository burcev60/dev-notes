# PostgreSQL — Основы

## Описание

PostgreSQL — мощная open-source реляционная СУБД.

## Подключение

```python
import psycopg

conn = psycopg.connect(
    host="localhost",
    dbname="mydb",
    user="postgres",
    password="secret"
)

cur = conn.cursor()
cur.execute("SELECT version()")
print(cur.fetchone())
```

## SQLAlchemy

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://user:pass@localhost/db")
```

## Основные типы данных

| Тип | Описание |
|-----|----------|
| INTEGER | Целое число |
| VARCHAR(n) | Строка с лимитом |
| TEXT | Неограниченная строка |
| BOOLEAN | True/False |
| TIMESTAMP | Дата и время |
| JSONB | JSON бинарный |
| UUID | Универсальный ID |
| ARRAY | Массив |

## Best Practices

✅ **Используйте** `JSONB` для JSON данных
✅ **Используйте** индексы для частых запросов
✅ **Настройте** `shared_buffers`, `work_mem`

## Ссылки

- [PostgreSQL документация](https://www.postgresql.org/docs/)
