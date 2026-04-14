# PostgreSQL — JSONB: Операторы и индексы

## Описание

JSONB — бинарный формат JSON в PostgreSQL. Поддерживает индексирование и мощные операторы запросов.

## JSON vs JSONB

| Характеристика | JSON | JSONB |
|---------------|------|-------|
| Хранение | Текст | Бинарный |
| Запись | Быстрая | Медленнее (парсинг) |
| Чтение | Медленнее | Быстрая |
| Индексы | ❌ | ✅ |
| Порядок ключей | Сохраняется | Не гарантируется |
| Дубликаты ключей | Сохраняются | Удаляются |

**Рекомендация:** Всегда используйте `JSONB`.

## Создание таблицы

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Вставка
INSERT INTO users (name, email, metadata) VALUES
('Alice', 'alice@example.com', '{"role": "admin", "age": 30, "tags": ["python", "fastapi"]}'),
('Bob', 'bob@example.com', '{"role": "user", "age": 25, "tags": ["javascript"]}');
```

## Операторы доступа

```sql
-- ->  — получить значение по ключу (как JSONB)
SELECT metadata->'role' FROM users WHERE name = 'Alice';
-- "admin"

-- ->> — получить значение как текст
SELECT metadata->>'role' FROM users WHERE name = 'Alice';
-- admin

-- #>  — получить по пути (как JSONB)
SELECT metadata#>'{address, city}' FROM users;

-- #>> — получить по пути как текст
SELECT metadata#>>'{address, city}' FROM users;
```

### Примеры

```sql
-- Доступ к вложенному объекту
SELECT metadata->'preferences'->>'theme' FROM users;

-- Доступ к массиву по индексу
SELECT metadata->'tags'->0 FROM users;

-- Кастинг к типу
SELECT (metadata->>'age')::int FROM users;
```

## Операторы проверки

```sql
-- @> — содержит (левый содержит правый)
SELECT * FROM users WHERE metadata @> '{"role": "admin"}';

-- <@ — содержится в (левый содержится в правом)
SELECT * FROM users WHERE '{"role": "admin"}' <@ metadata;

-- ? — ключ существует
SELECT * FROM users WHERE metadata ? 'email';

-- ?| — любой ключ из списка
SELECT * FROM users WHERE metadata ?| ARRAY['phone', 'email'];

-- ?& — все ключи существуют
SELECT * FROM users WHERE metadata ?& ARRAY['name', 'email'];
```

### Комбинация с JSON операторами

```sql
-- Поиск по значению в массиве
SELECT * FROM users WHERE metadata->'tags' ? 'python';

-- Проверка вложенного значения
SELECT * FROM users WHERE metadata->'preferences' @> '{"notifications": true}';
```

## Индексы для JSONB

### GIN индекс (рекомендуется)

```sql
-- GIN индекс для всех ключей
CREATE INDEX idx_users_metadata ON users USING GIN(metadata);

-- Использование с @>
SELECT * FROM users WHERE metadata @> '{"role": "admin"}';
-- Bitmap Index Scan

-- Использование с ?
SELECT * FROM users WHERE metadata ? 'email';
-- Bitmap Index Scan
```

### Индекс для конкретного ключа

```sql
-- Индекс только для одного ключа
CREATE INDEX idx_users_role ON users((metadata->>'role'));

-- Использование
SELECT * FROM users WHERE metadata->>'role' = 'admin';
-- Index Scan
```

### Составной индекс

```sql
-- Индекс для нескольких ключей
CREATE INDEX idx_users_metadata_keys ON users USING GIN(
    metadata->'role',
    metadata->'tags'
);
```

## Модификация JSONB

```sql
-- || — объединение
UPDATE users
SET metadata = metadata || '{"verified": true}'
WHERE id = 1;

-- - — удаление ключа
UPDATE users
SET metadata = metadata - 'temp_field'
WHERE id = 1;

-- #- — удаление по пути
UPDATE users
SET metadata = metadata #- '{address, city}'
WHERE id = 1;

-- Замена значения
UPDATE users
SET metadata = jsonb_set(metadata, '{role}', '"superadmin"')
WHERE id = 1;

-- Добавление в массив
UPDATE users
SET metadata = jsonb_set(
    metadata,
    '{tags}',
    (metadata->'tags') || '"django"'
)
WHERE id = 1;
```

## Агрегация JSONB

```sql
-- jsonb_agg — агрегация в массив
SELECT
    u.name,
    jsonb_agg(p.title) AS posts
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.name;

-- jsonb_object_agg — агрегация в объект
SELECT
    u.name,
    jsonb_object_agg(p.title, p.status) AS posts
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.name;

-- jsonb_build_object — создание объекта
SELECT jsonb_build_object(
    'user', u.name,
    'posts', jsonb_agg(p.title)
) AS result
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.name;
```

## Распаковка JSONB

```sql
-- jsonb_each — ключи и значения
SELECT key, value
FROM users, jsonb_each(metadata)
WHERE id = 1;

-- jsonb_array_elements — элементы массива
SELECT elem
FROM users, jsonb_array_elements(metadata->'tags') AS elem
WHERE id = 1;

-- С фильтрацией
SELECT elem->>'name'
FROM posts, jsonb_array_elements(metadata->'comments') AS elem
WHERE elem->>'rating' > '3';
```

## Проверка схемы (JSON Schema)

```sql
-- Проверка типа значения
SELECT * FROM users
WHERE jsonb_typeof(metadata->'age') = 'number';

-- Проверка что поле объект
SELECT * FROM users
WHERE jsonb_typeof(metadata->'address') = 'object';

-- Проверка что поле массив
SELECT * FROM users
WHERE jsonb_typeof(metadata->'tags') = 'array';
```

## Практические примеры

### Поиск пользователей по тегам

```sql
SELECT name, email
FROM users
WHERE metadata->'tags' @> '["python", "fastapi"]'::jsonb;
```

### Обновление вложенных значений

```sql
UPDATE users
SET metadata = jsonb_set(
    metadata,
    '{preferences, notifications, email}',
    'false'
)
WHERE metadata->>'role' = 'user';
```

### Подсчёт по JSONB полю

```sql
SELECT
    metadata->>'role' AS role,
    COUNT(*) AS count
FROM users
GROUP BY metadata->>'role'
ORDER BY count DESC;
```

### Сложный запрос с JSONB

```sql
SELECT
    u.name,
    u.email,
    u.metadata->>'role' AS role,
    COUNT(p.id) AS posts_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.metadata @> '{"is_active": true}'
  AND u.metadata->'tags' ? 'python'
GROUP BY u.id, u.name, u.email, u.metadata->>'role'
HAVING COUNT(p.id) > 5
ORDER BY posts_count DESC;
```

## Best Practices

✅ **Используйте** `JSONB` вместо `JSON`
✅ **Создавайте** GIN индекс для частых запросов
✅ **Используйте** `@>` для фильтрации (работает с индексом)
✅ **Проверяйте** схему через `jsonb_typeof`
✅ **Используйте** `jsonb_set` для атомарных обновлений

❌ **Не храните** реляционные данные в JSONB
❌ **Не используйте** `metadata->>'key' = 'value'` без индекса
❌ **Не создавайте** JSONB поля для часто обновляемых данных
❌ **Не забывайте** GIN индекс для `@>`, `?` операторов

## Ссылки

- [PostgreSQL JSONB Functions](https://www.postgresql.org/docs/current/functions-json.html)
- [PostgreSQL JSONB Indexing](https://www.postgresql.org/docs/current/datatype-json.html)
