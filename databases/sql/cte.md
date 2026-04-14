# SQL — CTE (Common Table Expressions)

## Описание

CTE — именованные временные результаты запросов, повышающие читаемость.

## Базовый CTE

```sql
WITH active_users AS (
    SELECT id, name FROM users WHERE is_active = true
)
SELECT * FROM active_users
WHERE name LIKE 'A%';
```

## Каскадные CTE

```sql
WITH
active_users AS (
    SELECT id, name FROM users WHERE is_active = true
),
user_orders AS (
    SELECT u.name, o.total
    FROM active_users u
    JOIN orders o ON u.id = o.user_id
)
SELECT name, SUM(total) as total_spent
FROM user_orders
GROUP BY name
ORDER BY total_spent DESC;
```

## Рекурсивный CTE

```sql
WITH RECURSIVE hierarchy AS (
    -- Базовый случай
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Рекурсивный случай
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.id
)
SELECT * FROM hierarchy ORDER BY level, name;
```

## Best Practices

✅ **Используйте** CTE для читаемости сложных запросов
✅ **Используйте** рекурсивные CTE для иерархий
✅ **Проверяйте** производительность (CTE может материализоваться)

## Ссылки

- [PostgreSQL WITH Queries](https://www.postgresql.org/docs/current/queries-with.html)
