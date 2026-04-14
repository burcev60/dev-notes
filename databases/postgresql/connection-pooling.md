# PostgreSQL — Connection Pooling

## Описание

Connection pool — переиспользование соединений с БД вместо создания новых для каждого запроса. Создание соединения — дорогая операция (TCP handshake, аутентификация, инициализация).

## asyncpg — Native async driver

### Базовый pool

```python
import asyncpg

# Создание пула
pool = await asyncpg.create_pool(
    dsn="postgresql://user:pass@localhost/mydb",
    min_size=5,       # Минимум соединений
    max_size=20,      # Максимум соединений
    max_queries=50000,  # Пересоздание после N запросов
    max_inactive_connection_lifetime=300.0,  # Закрытие неиспользуемых
    command_timeout=60,  # Таймаут запросов
)

# Использование
async with pool.acquire() as conn:
    row = await conn.fetchrow(
        "SELECT id, name FROM users WHERE email = $1",
        "alice@example.com"
    )
    print(row["id"], row["name"])

# Закрытие пула
await pool.close()
```

### Context manager для приложения

```python
class Database:
    def __init__(self, dsn: str, min_size: int = 5, max_size: int = 20):
        self._dsn = dsn
        self._pool: asyncpg.Pool | None = None
        self._min_size = min_size
        self._max_size = max_size

    async def connect(self):
        self._pool = await asyncpg.create_pool(
            dsn=self._dsn,
            min_size=self._min_size,
            max_size=self._max_size,
            max_inactive_connection_lifetime=300.0,
        )

    async def close(self):
        if self._pool:
            await self._pool.close()

    async def fetchrow(self, query: str, *args):
        async with self._pool.acquire() as conn:
            return await conn.fetchrow(query, *args)

    async def fetch(self, query: str, *args):
        async with self._pool.acquire() as conn:
            return await conn.fetch(query, *args)

    async def execute(self, query: str, *args):
        async with self._pool.acquire() as conn:
            return await conn.execute(query, *args)

    async def transaction(self):
        return self._pool.acquire()

# Использование
db = Database("postgresql://user:pass@localhost/mydb")
await db.connect()

# Запрос
user = await db.fetchrow(
    "SELECT * FROM users WHERE id = $1",
    1
)

# Транзакция
async with db.transaction() as conn:
    await conn.execute("INSERT INTO logs (user_id) VALUES ($1)", 1)
```

## psycopg (psycopg3) — Современный PostgreSQL драйвер

### Синхронный pool

```python
import psycopg
from psycopg_pool import ConnectionPool

# Пул соединений
pool = ConnectionPool(
    conninfo="postgresql://user:pass@localhost/mydb",
    min_size=5,
    max_size=20,
    timeout=30,  # Таймаут ожидания свободного соединения
    max_idle=300,  # Закрытие неиспользуемых
    max_lifetime=1800,  # Пересоздание соединений
)

# Использование
with pool.connection() as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT id, name FROM users WHERE id = %s", (1,))
        row = cur.fetchone()
        print(row)
```

### Асинхронный pool

```python
import psycopg
from psycopg_pool import AsyncConnectionPool

# Асинхронный пул
pool = AsyncConnectionPool(
    conninfo="postgresql://user:pass@localhost/mydb",
    min_size=5,
    max_size=20,
    timeout=30,
)

# Использование
async with pool.connection() as conn:
    async with conn.cursor() as cur:
        await cur.execute(
            "SELECT id, name FROM users WHERE id = %s",
            (1,)
        )
        row = await cur.fetchone()
        print(row)

# Закрытие
await pool.close()
```

## SQLAlchemy + asyncpg

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

# SQLAlchemy уже управляет пулом
engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/mydb",
    pool_size=20,           # Размер пула
    max_overflow=10,        # Дополнительные соединения
    pool_timeout=30,        # Ожидание свободного соединения
    pool_recycle=1800,      # Пересоздание (секунды)
    pool_pre_ping=True,     # Проверка соединения перед использованием
)

AsyncSession = async_sessionmaker(engine)

# Использование
async with AsyncSession() as session:
    result = await session.execute(
        select(User).where(User.id == 1)
    )
    user = result.scalar_one()
```

## Настройка пула

### Формула для выбора размера пула

```
pool_size = (CPU_cores * 2) + disk_spindles

# Пример для 4-ядерного сервера с SSD
pool_size = (4 * 2) + 1 = 9 → округляем до 10
```

### Рекомендации

| Нагрузка | min_size | max_size |
|----------|----------|----------|
| Малое приложение | 2 | 10 |
| Среднее API | 5 | 20 |
| Высоконагруженное | 10 | 50 |
| Микросервис (много инстансов) | 2 | 5 |

### Мониторинг пула (asyncpg)

```python
# Статистика пула
print(f"Pool size: {pool.get_size()}")
print(f"Free connections: {pool.get_idle_size()}")
print(f"Active connections: {pool.get_size() - pool.get_idle_size()}")

# Ожидание в очереди
print(f"Queue size: {pool._queue.qsize()}")  # Внутренний
```

## Best Practices

✅ **Используйте** `pool_pre_ping=True` для проверки соединений
✅ **Настраивайте** `pool_recycle` (меньше чем `wait_timeout` в PostgreSQL)
✅ **Используйте** `pool_size` на основе формулы (CPU * 2 + 1)
✅ **Мониторьте** размер пула и время ожидания
✅ **Закрывайте** пул при завершении приложения

❌ **Не создавайте** новое соединение на каждый запрос
❌ **Не ставьте** `max_size` слишком высоким (лимит БД ~100)
❌ **Не забывайте** `await pool.close()` при shutdown
❌ **Не используйте** один пул для разных БД

## Ссылки

- [asyncpg документация](https://magicstack.github.io/asyncpg/current/)
- [psycopg_pool](https://www.psycopg.org/psycopg3/docs/pool.html)
- [SQLAlchemy Engine Configuration](https://docs.sqlalchemy.org/en/20/core/engines.html)
