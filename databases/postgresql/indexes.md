# PostgreSQL — Индексы

## Описание

Индексы ускоряют поиск в таблицах.

## Типы индексов

```sql
-- B-Tree (по умолчанию)
CREATE INDEX idx_users_email ON users(email);

-- Уникальный индекс
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Частичный индекс
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- Составной индекс
CREATE INDEX idx_users_name_age ON users(last_name, first_name);

-- GIN индекс (для JSONB, массивов)
CREATE INDEX idx_users_tags ON users USING GIN(tags);

-- GiST индекс (для полнотекстового поиска)
CREATE INDEX idx_docs_search ON documents USING GIST(to_tsvector('english', content));
```

## Когда использовать

| Ситуация | Индекс |
|----------|--------|
| WHERE column = value | B-Tree |
| WHERE column LIKE 'prefix%' | B-Tree |
| WHERE jsonb @> '{"key": "value"}' | GIN |
| WHERE array && [1,2] | GIN |
| Полнотекстовый поиск | GiST/GIN |

## Best Practices

✅ **Создавайте** индексы для WHERE и JOIN колон
✅ **Мониторьте** неиспользуемые индексы
✅ **Избегайте** избыточных индексов (замедляют INSERT)

## Ссылки

- [PostgreSQL Indexes](https://www.postgresql.org/docs/current/indexes.html)
