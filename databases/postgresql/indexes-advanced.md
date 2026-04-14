# PostgreSQL — Продвинутые индексы

## Описание

Типы индексов в PostgreSQL и их применение для разных сценариев.

## B-tree (по умолчанию)

```sql
-- Стандартный индекс
CREATE INDEX idx_users_email ON users(email);

-- Уникальный индекс
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Составной индекс (порядок важен!)
CREATE INDEX idx_users_name_email ON users(last_name, first_name);

-- По условию (частичный индекс)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- DESC индекс
CREATE INDEX idx_posts_created ON posts(created_at DESC);

-- NULLS LAST/FIRST
CREATE INDEX idx_products_price ON products(price DESC NULLS LAST);
```

### Когда использовать B-tree

| Операция | Поддержка |
|----------|-----------|
| `=` | ✅ |
| `<`, `<=`, `>`, `>=` | ✅ |
| `BETWEEN` | ✅ |
| `LIKE 'prefix%'` | ✅ |
| `LIKE '%suffix'` | ❌ |
| `IN` | ✅ |
| `ORDER BY` | ✅ |

## GIN (Generalized Inverted Index)

```sql
-- JSONB индекс
CREATE INDEX idx_users_metadata ON users USING GIN(metadata);

-- Массивы
CREATE INDEX idx_posts_tags ON posts USING GIN(tags);

-- Полнотекстовый поиск
CREATE INDEX idx_docs_search ON documents USING GIN(to_tsvector('russian', content));
```

### JSONB операторы с GIN

```sql
-- @> содержит
SELECT * FROM users WHERE metadata @> '{"role": "admin"}';

-- ? ключ существует
SELECT * FROM users WHERE metadata ? 'email';

-- ?| любой ключ из списка
SELECT * FROM users WHERE metadata ?| ARRAY['email', 'phone'];

-- ?& все ключи существуют
SELECT * FROM users WHERE metadata ?& ARRAY['email', 'phone'];

-- Комбинация с обычным WHERE
SELECT * FROM users
WHERE is_active = true
  AND metadata @> '{"role": "admin"}';
```

### Массивы с GIN

```sql
-- Поиск постов с определёнными тегами
SELECT * FROM posts WHERE tags @> ARRAY['python', 'fastapi'];

-- Пересечение
SELECT * FROM posts WHERE tags && ARRAY['python', 'django'];

-- Вложенность
SELECT * FROM posts WHERE tags <@ ARRAY['python', 'fastapi', 'async'];
```

## GiST (Generalized Search Tree)

```sql
-- Полнотекстовый поиск (с ранжированием)
CREATE INDEX idx_docs_search ON documents USING GiST(to_tsvector('russian', content));

-- Geometric данные
CREATE INDEX idx_locations_coords ON locations USING GiST(coords);

-- Range типы
CREATE INDEX idx_events_duration ON events USING GiST(duration);
```

### Полнотекстовый поиск с GiST

```sql
-- Поиск с ранжированием
SELECT
    title,
    ts_rank(to_tsvector('russian', content), query) AS rank
FROM documents,
     plainto_tsquery('russian', 'Python программирование') AS query
WHERE to_tsvector('russian', content) @@ query
ORDER BY rank DESC
LIMIT 10;

-- С подсветкой
SELECT
    ts_headline('russian', content, query, 'StartSel=** StopSel=**')
FROM documents,
     plainto_tsquery('russian', 'Python') AS query
WHERE to_tsvector('russian', content) @@ query;
```

## Покрывающие индексы (INCLUDE)

```sql
-- INCLUDE — данные только для index-only scan
CREATE INDEX idx_users_email_covering
    ON users(email)
    INCLUDE (name, is_active);

-- Query — не нужно обращаться к таблице
SELECT email, name, is_active
FROM users
WHERE email = 'alice@example.com';
-- Index Only Scan!
```

### Когда работает Index Only Scan

```sql
-- Проверка видимости (visibility map)
-- Должен быть VACUUM для актуальности

-- Проверка плана
EXPLAIN (ANALYZE, BUFFERS)
SELECT email, name FROM users WHERE email = 'alice@example.com';

-- Index Only Scan (хорошо)
-- Index Scan + Heap Fetches (нужен VACUUM)
```

## Частичные индексы

```sql
-- Только для активных пользователей
CREATE INDEX idx_active_users_email
    ON users(email)
    WHERE is_active = true;

-- Только для неархивных заказов
CREATE INDEX idx_orders_active
    ON orders(created_at)
    WHERE status NOT IN ('cancelled', 'archived');

-- Только для определённого типа
CREATE INDEX idx_posts_published
    ON posts(published_at)
    WHERE status = 'published';
```

### Преимущества частичных индексов

- Меньше места на диске
- Быстрее обновление (меньше индексов обновлять)
- Быстрее поиск (меньше данных сканировать)

## Выражения в индексах

```sql
-- Индекс по выражению
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Использование
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Индекс по дате
CREATE INDEX idx_orders_month ON orders(DATE_TRUNC('month', created_at));

-- Использование
SELECT * FROM orders WHERE DATE_TRUNC('month', created_at) = '2024-01-01';
```

## Проверка использования индексов

```sql
-- Поиск неиспользуемых индексов
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan AS scans,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(indexrelid) DESC;

-- Статистика по индексам
SELECT
    relname,
    indexrelname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

## Best Practices

✅ **Используйте** GIN для JSONB и массивов
✅ **Используйте** GiST для полнотекстового поиска
✅ **Используйте** INCLUDE для index-only scan
✅ **Используйте** частичные индексы для фильтрации
✅ **Мониторьте** неиспользуемые индексы

❌ **Не создавайте** индекс для каждого столбца
❌ **Не создавайте** частичные индексы для часто меняющихся условий
❌ **Не забывайте** про VACUUM для index-only scan
❌ **Не используйте** GIN для range запросов (только B-tree)

## Ссылки

- [PostgreSQL Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [PostgreSQL JSONB Indexing](https://www.postgresql.org/docs/current/datatype-json.html)
- [PostgreSQL Full Text Search](https://www.postgresql.org/docs/current/textsearch.html)
