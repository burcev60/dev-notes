# PostgreSQL — Планы запросов (EXPLAIN)

## Описание

EXPLAIN показывает план выполнения запроса для оптимизации.

## Использование

```sql
-- План запроса
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- С выполнением (реальные метрики)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';

-- Формат JSON
EXPLAIN (FORMAT JSON) SELECT * FROM users;
```

## Чтение плана

```
Seq Scan on users  (cost=0.00..18.00 rows=1 width=100)
  Filter: (email = 'alice@example.com'::text)

Index Scan using idx_users_email on users  (cost=0.00..8.00 rows=1 width=100)
  Index Cond: (email = 'alice@example.com'::text)
```

| Тип | Описание |
|-----|----------|
| Seq Scan | Полное сканирование таблицы (медленно) |
| Index Scan | Поиск по индексу (быстро) |
| Index Only Scan | Только по индексу (очень быстро) |

## Оптимизация

```sql
-- Seq Scan → Index Scan
CREATE INDEX idx_users_email ON users(email);

-- Анализ статистики
ANALYZE users;

-- Переиндексация
REINDEX INDEX idx_users_email;
```

## Best Practices

✅ **Запускайте** `EXPLAIN ANALYZE` для медленных запросов
✅ **Избегайте** Seq Scan на больших таблицах
✅ **Обновляйте** статистику через `ANALYZE`

## Ссылки

- [EXPLAIN документация](https://www.postgresql.org/docs/current/using-explain.html)
- [explain.depesz.com — визуализация планов](https://explain.depesz.com/)
