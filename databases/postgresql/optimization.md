# PostgreSQL — Оптимизация: LIMIT/OFFSET, N+1

## Описание

Распространённые проблемы производительности и способы их решения.

## LIMIT / OFFSET проблема

```sql
-- ❌ ПЛОХО — OFFSET сканирует и отбрасывает строки
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 10 OFFSET 10000;
-- Сканирует 10010 строк, отбрасывает 10000, возвращает 10

-- ✅ ХОРОШО — Keyset pagination (cursor-based)
SELECT * FROM posts
WHERE created_at < '2024-01-01'  -- Последнее значение с предыдущей страницы
ORDER BY created_at DESC
LIMIT 10;
-- Сканирует только 10 строк через индекс
```

### Keyset Pagination реализация

```python
# API эндпоинт
@app.get("/posts/")
async def get_posts(
    limit: int = 10,
    cursor: str | None = None,  # created_at последней записи
):
    query = select(Post).order_by(Post.created_at.desc()).limit(limit)

    if cursor:
        query = query.where(Post.created_at < cursor)

    posts = await session.execute(query)
    posts = posts.scalars().all()

    next_cursor = posts[-1].created_at if posts else None

    return {
        "items": posts,
        "next_cursor": next_cursor,
        "has_more": len(posts) == limit,
    }
```

### Сравшение подходов

| Метод | Скорость | Сложность | Смешение страниц |
|-------|----------|-----------|-----------------|
| OFFSET | Медленно (O(n)) | Просто | Нет |
| Keyset | Быстро (O(log n)) | Средне | Нет |
| Cursor (id) | Быстро | Просто | Да (если вставка) |

## N+1 запросы

### Проблема

```python
# ❌ ПЛОХО — N+1 запросов
posts = await session.execute(select(Post).limit(10))
for post in posts.scalars():
    # Запрос для каждого поста!
    author = await session.execute(
        select(User).where(User.id == post.author_id)
    )
    print(post.title, author.scalar_one().name)
# 1 запрос для posts + 10 запросов для authors = 11 запросов
```

### Решение: Eager Loading

```python
# ✅ ХОРОШО — 1 запрос с JOIN
from sqlalchemy.orm import selectinload

posts = await session.execute(
    select(Post)
    .options(selectinload(Post.author))  # Подгрузить авторов
    .limit(10)
)
for post in posts.scalars():
    print(post.title, post.author.name)  # Без дополнительного запроса
# 2 запроса: один для posts, один для всех authors

# joinedload — через JOIN
from sqlalchemy.orm import joinedload

posts = await session.execute(
    select(Post)
    .options(joinedload(Post.author))
    .limit(10)
)

# Вложенные отношения
posts = await session.execute(
    select(Post)
    .options(
        selectinload(Post.author),
        selectinload(Post.comments).selectinload(Comment.author),
    )
    .limit(10)
)
```

### Когда использовать

| Метод | Когда |
|-------|-------|
| `selectinload` | Коллекции (one-to-many, many-to-many) |
| `joinedload` | One-to-one, many-to-one |
| `subqueryload` | Альтернатива selectinload (иногда быстрее) |

## Индексы для сортировки

```sql
-- Сортировка без индекса
SELECT * FROM posts ORDER BY created_at DESC;
-- Sort (top-N heapsort)

-- С индексом
CREATE INDEX idx_posts_created_desc ON posts(created_at DESC);
-- Index Scan Backward

-- Составной индекс для сортировки + фильтрации
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);

-- Использование
SELECT * FROM posts
WHERE user_id = 1
ORDER BY created_at DESC;
-- Index Scan (не нужно дополнительной сортировки!)
```

## Покрытие индексов

```sql
-- Index Only Scan — все данные в индексе
CREATE INDEX idx_posts_user_title ON posts(user_id) INCLUDE (title, created_at);

-- Запрос не обращается к таблице
SELECT user_id, title, created_at
FROM posts
WHERE user_id = 1;
-- Index Only Scan!
```

## Избегание SELECT *

```sql
-- ❌ ПЛОХО
SELECT * FROM users WHERE id = 1;
-- Загружает все колонки (включая большие)

-- ✅ ХОРОШО
SELECT id, name, email FROM users WHERE id = 1;
-- Загружает только нужные колонки

-- В SQLAlchemy
stmt = select(User.id, User.name, User.email).where(User.id == 1)
# или
stmt = select(User).options(load_only(User.id, User.name, User.email))
```

## Batch операции

```sql
-- ❌ ПЛОХО — N отдельных INSERT
INSERT INTO users (name, email) VALUES ('Alice', 'a@x.com');
INSERT INTO users (name, email) VALUES ('Bob', 'b@x.com');
...

-- ✅ ХОРОШО — Один INSERT с множеством значений
INSERT INTO users (name, email) VALUES
    ('Alice', 'a@x.com'),
    ('Bob', 'b@x.com'),
    ('Charlie', 'c@x.com');

-- Или COPY для больших объёмов
COPY users(name, email) FROM '/tmp/users.csv' WITH CSV HEADER;
```

### В SQLAlchemy

```python
# ❌ ПЛОХО
for user_data in users_data:
    user = User(**user_data)
    session.add(user)
    await session.commit()  # COMMIT для каждой записи!

# ✅ ХОРОШО — Bulk insert
users = [User(**data) for data in users_data]
session.add_all(users)
await session.commit()

# ✅ ЕЩЁ ЛУЧШЕ — executemany
await session.execute(
    insert(User),
    users_data  # list[dict]
)
await session.commit()
```

## Подзапроси vs JOIN

```sql
-- ❌ ПЛОХО — Подзапрос в SELECT (выполняется для каждой строки)
SELECT
    u.name,
    (SELECT COUNT(*) FROM posts WHERE user_id = u.id) AS posts_count
FROM users u;

-- ✅ ХОРОШО — JOIN с GROUP BY
SELECT
    u.name,
    COUNT(p.id) AS posts_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- ✅ ИЛИ — оконная функция
SELECT DISTINCT
    u.name,
    COUNT(p.id) OVER (PARTITION BY u.id) AS posts_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

## Материализованные представления

```sql
-- Создание
CREATE MATERIALIZED VIEW user_stats AS
SELECT
    u.id,
    u.name,
    COUNT(p.id) AS posts_count,
    MAX(p.created_at) AS last_post_at
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- Индекс для быстрого поиска
CREATE UNIQUE INDEX idx_user_stats_id ON user_stats(id);

-- Обновление
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;

-- Использование
SELECT * FROM user_stats WHERE posts_count > 10;
```

## Best Practices

✅ **Используйте** keyset pagination вместо OFFSET
✅ **Используйте** `selectinload`/`joinedload` для N+1
✅ **Создавайте** индексы для ORDER BY колонок
✅ **Используйте** `SELECT col1, col2` вместо `SELECT *`
✅ **Используйте** bulk insert для множества записей
✅ **Используйте** материализованные представления для сложных агрегаций

❌ **Не используйте** OFFSET > 1000 — деградирует
❌ **Не забывайте** про eager loading
❌ **Не используйте** подзапросы в SELECT для каждой строки
❌ **Не создавайте** индексы без анализа EXPLAIN

## Ссылки

- [PostgreSQL Pagination](https://www.citusdata.com/blog/2016/03/30/five-ways-to-paginate/)
- [SQLAlchemy Loading Strategies](https://docs.sqlalchemy.org/en/20/orm/loading_relationships.html)
