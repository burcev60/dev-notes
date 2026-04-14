# PostgreSQL — EXPLAIN ANALYZE: Чтение планов запросов

## Описание

`EXPLAIN ANALYZE` показывает план выполнения запроса с реальными метриками времени и строк.

## Базовое использование

```sql
-- План без выполнения
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- План с выполнением (реальные метрики)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';

-- Формат JSON
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM users WHERE email = 'alice@example.com';

-- С буферами (чтение с диска/кэша)
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE email = 'alice@example.com';
```

## Чтение плана

```
Seq Scan on users  (cost=0.00..18.00 rows=1 width=100) (actual time=0.010..0.025 rows=1 loops=1)
  Filter: (email = 'alice@example.com'::text)
  Rows Removed by Filter: 999
Planning Time: 0.050 ms
Execution Time: 0.050 ms
```

### Ключевые метрики

| Метрика | Описание |
|---------|----------|
| `cost=0.00..18.00` | Стоимость: start..total (произвольные единицы) |
| `rows=1` | Ожидаемое количество строк |
| `width=100` | Ожидаемый размер строки (байты) |
| `actual time=0.010..0.025` | Реальное время: first..total (ms) |
| `rows=1` | Реальное количество строк |
| `loops=1` | Количество выполнений узла |
| `Rows Removed by Filter` | Отфильтрованные строки |

## Типы сканирования

### Seq Scan — Полное сканирование таблицы

```
Seq Scan on users  (cost=0.00..18.00 rows=1 width=100)
```

- **Когда**: Нет индекса или таблица очень мала
- **Плохо**: Для больших таблиц
- **Решение**: Создать индекс

### Index Scan — Поиск по индексу + обращение к таблице

```
Index Scan using idx_users_email on users  (cost=0.00..8.00 rows=1 width=100)
  Index Cond: (email = 'alice@example.com'::text)
```

- **Когда**: Есть индекс и условие
- **Хорошо**: Быстрый поиск
- **Может быть лучше**: Index Only Scan

### Index Only Scan — Только по индексу

```
Index Only Scan using idx_users_email_covering on users  (cost=0.00..4.00 rows=1 width=50)
  Index Cond: (email = 'alice@example.com'::text)
```

- **Когда**: Все данные в индексе (INCLUDE)
- **Отлично**: Без обращения к таблице
- **Требует**: VACUUM для visibility map

### Bitmap Index Scan — Множественные индексы

```
Bitmap Heap Scan on posts  (cost=4.50..20.00 rows=10 width=100)
  Recheck Cond: (tags @> ARRAY['python'])
  -> Bitmap Index Scan on idx_posts_tags  (cost=0.00..4.50 rows=10 width=0)
       Index Cond: (tags @> ARRAY['python'])
```

- **Когда**: Несколько условий или GIN индекс
- **Хорошо**: Комбинирует несколько индексов

### Nested Loop — Вложенные циклы

```
Nested Loop  (cost=0.00..20.00 rows=10 width=100)
  -> Index Scan using idx_users_email on users  (cost=0.00..8.00 rows=1)
  -> Index Scan using idx_posts_user_id on posts  (cost=0.00..12.00 rows=10)
       Index Cond: (user_id = users.id)
```

- **Когда**: Маленький результат + индекс
- **Хорошо**: Для малого количества строк
- **Плохо**: Для большого (O(n*m))

### Hash Join — Хэш соединение

```
Hash Join  (cost=10.00..50.00 rows=100 width=100)
  Hash Cond: (posts.user_id = users.id)
  -> Seq Scan on posts  (cost=0.00..30.00 rows=1000)
  -> Hash  (cost=5.00..5.00 rows=100 width=50)
       -> Index Scan using idx_users_email on users  (cost=0.00..5.00 rows=100)
```

- **Когда**: Большие таблицы без индексов
- **Плохо**: Может быть медленным
- **Решение**: Индексы на JOIN колонках

### Merge Join — Сортированное соединение

```
Merge Join  (cost=0.50..40.00 rows=100 width=100)
  Merge Cond: (posts.user_id = users.id)
  -> Index Scan using idx_posts_user_id on posts  (cost=0.00..20.00 rows=1000)
  -> Index Scan using idx_users_id on users  (cost=0.00..10.00 rows=100)
```

- **Когда**: Оба входа отсортированы по JOIN колонке
- **Хорошо**: Для отсортированных данных

## Проблемы и оптимизация

### Seq Scan → Index Scan

```sql
-- Проблема: Seq Scan
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';
-- Seq Scan on users

-- Решение: создать индекс
CREATE INDEX idx_users_email ON users(email);

-- После: Index Scan
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';
-- Index Scan using idx_users_email
```

### Ожидаемые vs реальные строки

```sql
-- planner не знает реальных данных
EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 1;
-- rows=10 actual rows=10000

-- Решение: обновить статистику
ANALYZE posts;

-- Проверить статистику
SELECT
    attname,
    n_distinct,
    most_common_vals
FROM pg_stats
WHERE tablename = 'posts' AND attname = 'user_id';
```

### Вложенные запросы

```sql
-- Проблема: Nested Loop для большого количества
EXPLAIN ANALYZE
SELECT u.*, p.title
FROM users u
JOIN posts p ON u.id = p.user_id
WHERE u.is_active = true;

-- Nested Loop  (actual rows=100000)

-- Решение: индекс + ANALYZE
CREATE INDEX idx_posts_user_id ON posts(user_id);
ANALYZE users;
ANALYZE posts;

-- После: Hash Join или Merge Join
```

## ANALYZE — Обновление статистики

```sql
-- Обновить статистику для таблицы
ANALYZE users;

-- Обновить для колонки
ANALYZE users(email);

-- Обновить для всех таблиц
ANALYZE;

-- Проверить когда последний раз
SELECT
    relname,
    last_analyze,
    last_autoanalyze,
    analyze_count
FROM pg_stat_user_tables
WHERE relname = 'users';
```

## VACUUM — Очистка мёртвых строк

```sql
-- Очистка таблицы
VACUUM users;

-- Анализ после очистки
VACUUM ANALYZE users;

-- Полная очистка (блокирует таблицу!)
VACUUM FULL users;

-- Проверить размер мёртвых строк
SELECT
    relname,
    n_dead_tup,
    n_live_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE relname = 'users';
```

### Autovacuum

```sql
-- Настройки (postgresql.conf)
autovacuum = on
autovacuum_max_workers = 3
autovacuum_naptime = 60  -- Проверка каждые 60 сек

-- Для конкретной таблицы
ALTER TABLE users SET (
    autovacuum_vacuum_threshold = 50,
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_analyze_threshold = 50,
    autovacuum_analyze_scale_factor = 0.05
);
```

## Практический анализ

```sql
-- Полный анализ запроса
EXPLAIN (
    ANALYZE,      -- Выполнить запрос
    BUFFERS,      -- Показать чтение с диска/кэша
    FORMAT TEXT   -- Формат вывода
)
SELECT u.name, COUNT(p.id) AS posts_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.is_active = true
  AND p.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.name
ORDER BY posts_count DESC
LIMIT 10;
```

## Best Practices

✅ **Используйте** `EXPLAIN ANALYZE` для медленных запросов
✅ **Обновляйте** статистику через `ANALYZE`
✅ **Настраивайте** autovacuum для активных таблиц
✅ **Проверяйте** Seq Scan на больших таблицах
✅ **Мониторьте** `rows` vs `actual rows`

❌ **Не используйте** `VACUUM FULL` на продакшене (блокировка!)
❌ **Не игнорируйте** `Rows Removed by Filter`
❌ **Не верьте** cost без ANALYZE (это оценка)
❌ **Не забывайте** про индексы на JOIN колонках

## Ссылки

- [EXPLAIN документация](https://www.postgresql.org/docs/current/using-explain.html)
- [explain.depesz.com — визуализация](https://explain.depesz.com/)
- [PostgreSQL Performance](https://wiki.postgresql.org/wiki/Performance_Optimization)
