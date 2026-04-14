# SQL — Оконные функции

## Описание

Оконные функции выполняют вычисления над набором строк, связанных с текущей.

## Синтаксис

```sql
function_name() OVER (
    PARTITION BY column
    ORDER BY column
    ROWS BETWEEN ... AND ...
)
```

## Функции

```sql
-- ROW_NUMBER — номер строки
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- RANK — ранг с пропусками
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- DENSE_RANK — ранг без пропусков
SELECT name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- SUM, AVG, COUNT
SELECT name, department, salary,
       SUM(salary) OVER (PARTITION BY department) as dept_total,
       AVG(salary) OVER (PARTITION BY department) as dept_avg
FROM employees;

-- LAG, LEAD — предыдущее/следующее значение
SELECT name, salary,
       LAG(salary, 1) OVER (ORDER BY salary) as prev_salary,
       LEAD(salary, 1) OVER (ORDER BY salary) as next_salary
FROM employees;

-- FIRST_VALUE, LAST_VALUE
SELECT name, salary,
       FIRST_VALUE(name) OVER (ORDER BY salary DESC) as top_earner
FROM employees;
```

## Best Practices

✅ **Используйте** для ранжирования и агрегации
✅ **PARTITION BY** разделяет данные на группы
✅ **Избегайте** оконных функций в WHERE (используйте CTE)

## Ссылки

- [PostgreSQL Window Functions](https://www.postgresql.org/docs/current/functions-window.html)
