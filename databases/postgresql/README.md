# PostgreSQL

Заметки по PostgreSQL.

## 📂 Заметки

### Основы

| Файл | Тема |
|------|------|
| [`basics.md`](basics.md) | Основы PostgreSQL, подключение, типы данных |
| [`indexes.md`](indexes.md) | Индексы: B-Tree, уникальные, составные |
| [`query-plans.md`](query-plans.md) | EXPLAIN, анализ планов запросов |

### Продвинутые темы

| Файл | Тема |
|------|------|
| [`connection-pooling.md`](connection-pooling.md) | Connection pooling: asyncpg, psycopg_pool |
| [`transactions.md`](transactions.md) | Транзакции: isolation levels, savepoints |
| [`indexes-advanced.md`](indexes-advanced.md) | GIN, GiST, частичные, покрывающие индексы |
| [`explain-analyze.md`](explain-analyze.md) | Чтение планов, ANALYZE, VACUUM |
| [`optimization.md`](optimization.md) | LIMIT/OFFSET проблемы, N+1 запросы |
| [`jsonb.md`](jsonb.md) | JSONB: операторы, индексы, запросы |
| [`migrations.md`](migrations.md) | Alembic: авто-генерация, ручные правки |
| [`testing.md`](testing.md) | Тестовая БД, fixtures, транзакционные тесты |
