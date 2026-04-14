# SQL — JOIN типы

## Описание

JOIN объединяет строки из двух или более таблиц.

## Типы JOIN

```sql
-- INNER JOIN — только совпадающие
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN — все из левой + совпадающие из правой
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN — все из правой + совпадающие из левой
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN — все строки из обеих таблиц
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN — декартово произведение
SELECT u.name, p.product
FROM users u
CROSS JOIN products p;
```

## Визуализация

```
INNER:      A ∩ B
LEFT:       A + (A ∩ B)
RIGHT:      B + (A ∩ B)
FULL:       A ∪ B
CROSS:      A × B
```

## Best Practices

✅ **Используйте** LEFT JOIN когда нужны все записи из основной таблицы
✅ **Индексируйте** JOIN колонки
✅ **Избегайте** CROSS JOIN на больших таблицах

## Ссылки

- [SQL JOIN визуализация](https://sql-joins.leopard.in.ua/)
