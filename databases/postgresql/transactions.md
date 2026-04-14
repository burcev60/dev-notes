# PostgreSQL — Транзакции: Isolation Levels, Savepoints

## Описание

Транзакции гарантируют ACID свойства: Atomicity (атомарность), Consistency (согласованность), Isolation (изоляция), Durability (долговечность).

## ACID свойства

| Свойство | Описание |
|----------|----------|
| **Atomicity** | Все или ничего — либо все операции выполнены, ни одна |
| **Consistency** | БД переходит из одного согласованного состояния в другое |
| **Isolation** | Параллельные транзакции не влияют друг на друга |
| **Durability** | После COMMIT данные сохраняются даже при сбое |

## BEGIN / COMMIT / ROLLBACK

```sql
-- Начало транзакции
BEGIN;

-- Операции
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Фиксация
COMMIT;

-- Или откат
ROLLBACK;
```

### В Python (asyncpg)

```python
import asyncpg

async with pool.acquire() as conn:
    async with conn.transaction():
        await conn.execute(
            "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
            100, 1
        )
        await conn.execute(
            "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
            100, 2
        )
    # Автоматический COMMIT при выходе
    # Автоматический ROLLBACK при исключении
```

### В Python (SQLAlchemy)

```python
async with AsyncSession() as session:
    try:
        # Операции
        account1.balance -= 100
        account2.balance += 100

        session.add(account1)
        session.add(account2)

        await session.commit()
    except Exception:
        await session.rollback()
        raise
```

## Isolation Levels

| Уровень | Dirty Read | Non-Repeatable Read | Phantom Read |
|---------|-----------|-------------------|--------------|
| **READ UNCOMMITTED** | Возможно (в PG = READ COMMITTED) | Возможно | Возможно |
| **READ COMMITTED** (default) | ❌ | Возможно | Возможно |
| **REPEATABLE READ** | ❌ | ❌ | Возможно (в PG = ❌) |
| **SERIALIZABLE** | ❌ | ❌ | ❌ |

### Установка уровня изоляции

```sql
-- Для текущей транзакции
BEGIN ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Или
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Для сессии
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### READ COMMITTED (по умолчанию)

```sql
-- Сессия A
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 1000

-- Сессия B (пока A ещё в транзакции)
UPDATE accounts SET balance = 2000 WHERE id = 1;
COMMIT;

-- Сессия A
SELECT balance FROM accounts WHERE id = 1;  -- 2000 (видит изменения B!)
COMMIT;
```

### REPEATABLE READ

```sql
-- Сессия A
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- 1000

-- Сессия B
UPDATE accounts SET balance = 2000 WHERE id = 1;
COMMIT;

-- Сессия A
SELECT balance FROM accounts WHERE id = 1;  -- 1000 (снимок на начало транзакции)
COMMIT;
```

### SERIALIZABLE — строгая изоляция

```sql
-- Сессия A
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT balance FROM accounts WHERE id = 1;  -- 1000

-- Сессия B
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT balance FROM accounts WHERE id = 1;  -- 1000

-- Сессия A
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;  -- OK

-- Сессия B
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
COMMIT;  -- ERROR: could not serialize access
-- (conflict detected — нужно retry)
```

### Retry для SERIALIZABLE

```python
import asyncpg
import asyncio

async def serializable_transaction(conn, func, max_retries=3):
    for attempt in range(max_retries):
        try:
            async with conn.transaction(isolation="serializable"):
                return await func(conn)
        except asyncpg.SerializationFailure:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(0.1 * (2 ** attempt))  # Exponential backoff

# Использование
async def transfer(conn):
    await conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    await conn.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")

async with pool.acquire() as conn:
    await serializable_transaction(conn, transfer)
```

## Savepoints — Частичный откат

```sql
BEGIN;

-- Первая операция
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- Savepoint
SAVEPOINT sp1;

-- Вторая операция (может ошибиться)
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Если ошибка — откат к savepoint
ROLLBACK TO sp1;

-- Или подтверждение
RELEASE SAVEPOINT sp1;

COMMIT;
```

### В Python

```python
async with pool.acquire() as conn:
    async with conn.transaction():
        await conn.execute(
            "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
            100, 1
        )

        # Savepoint
        async with conn.transaction():
            try:
                await conn.execute(
                    "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
                    100, 2
                )
            except Exception:
                # Откат к savepoint, основная транзакция продолжается
                raise

        # Продолжение основной транзакции
        await conn.execute(
            "INSERT INTO logs (user_id, amount) VALUES (1, 100)"
        )
```

### В SQLAlchemy

```python
async with AsyncSession() as session:
    async with session.begin():
        # Основная операция
        session.add(log1)

        # Savepoint
        async with session.begin_nested():
            try:
                session.add(log2)
                await session.flush()
            except Exception:
                # log2 отменён, log1 остаётся
                pass

        # Продолжение
        session.add(log3)
```

## Long-running транзакции

```sql
-- Проверка долгих транзакций
SELECT
    pid,
    now() - xact_start AS duration,
    query,
    state
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY duration DESC;

-- Отмена долгой транзакции
SELECT pg_terminate_backend(pid);
```

### Предотвращение

```sql
-- Таймаут транзакции (в postgresql.conf)
statement_timeout = 30000        -- 30 секунд
idle_in_transaction_session_timeout = 60000  -- 60 секунд

-- Или для сессии
SET statement_timeout = '30s';
SET idle_in_transaction_session_timeout = '60s';
```

## Best Practices

✅ **Используйте** `READ COMMITTED` по умолчанию
✅ **Используйте** `REPEATABLE READ` для сложных вычислений
✅ **Используйте** `SERIALIZABLE` только когда нужна строгая консистентность
✅ **Используйте** savepoints для частичного отката
✅ **Реализуйте** retry для `SERIALIZABLE` транзакций
✅ **Устанавливайте** `statement_timeout`

❌ **Не оставляйте** транзакции открытыми надолго
❌ **Не используйте** `SERIALIZABLE` без retry логики
❌ **Не блокируйте** соединения вне транзакции
❌ **Не игнорируйте** `idle_in_transaction_session_timeout`

## Ссылки

- [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL Concurrency Control](https://www.postgresql.org/docs/current/mvcc.html)
