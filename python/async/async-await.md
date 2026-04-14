# async/await — Паттерны

## Описание

Паттерны использования async/await для эффективного асинхронного кода.

## Асинхронная функция vs обычная

```python
# Обычная функция
def sync_greet(name: str) -> str:
    return f"Hello, {name}!"

# Асинхронная функция (корутина)
async def async_greet(name: str) -> str:
    return f"Hello, {name}!"

# Вызов
result = sync_greet("Alice")           # Обычный вызов
result = await async_greet("Alice")    # Только из async контекста
```

## Передача параметров

```python
async def fetch_with_timeout(url: str, timeout: float = 10.0) -> str:
    try:
        return await asyncio.wait_for(
            fetch_url(url),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        return f"Timeout fetching {url}"

async def fetch_url(url: str) -> str:
    await asyncio.sleep(5)  # Симуляция запроса
    return f"Data from {url}"
```

## Цепочка вызовов

```python
async def get_user(user_id: int) -> dict:
    await asyncio.sleep(0.1)
    return {"id": user_id, "name": f"User{user_id}"}

async def get_posts(user: dict) -> list:
    await asyncio.sleep(0.1)
    return [f"Post by {user['name']}"]

async def get_user_with_posts(user_id: int) -> dict:
    user = await get_user(user_id)
    posts = await get_posts(user)
    return {**user, "posts": posts}
```

## Параллельное выполнение

```python
import asyncio

# gather — ждать все
async def fetch_all(urls: list[str]) -> list[str]:
    tasks = [fetch_url(url) for url in urls]
    return await asyncio.gather(*tasks)

# as_completed — получать по мере готовности
async def fetch_as_completed(urls: list[str]) -> list[str]:
    results = []
    for coro in asyncio.as_completed([fetch_url(url) for url in urls]):
        result = await coro
        results.append(result)
        print(f"Got: {result}")
    return results
```

## Ошибки в async коде

```python
import asyncio

async def risky_task(task_id: int) -> str:
    await asyncio.sleep(0.1 * task_id)
    if task_id == 2:
        raise ValueError(f"Task {task_id} failed")
    return f"Task {task_id} done"

# gather с обработкой ошибок
async def main():
    # return_exceptions=True — не бросать исключения
    results = await asyncio.gather(
        risky_task(1),
        risky_task(2),
        risky_task(3),
        return_exceptions=True
    )
    for result in results:
        if isinstance(result, Exception):
            print(f"Error: {result}")
        else:
            print(f"Success: {result}")

asyncio.run(main())

# gather без return_exceptions — упадёт на первой ошибке
try:
    results = await asyncio.gather(
        risky_task(1),
        risky_task(2),
        risky_task(3),
    )
except ValueError as e:
    print(f"Caught: {e}")
```

## Async итераторы

```python
class AsyncCounter:
    def __init__(self, max: int) -> None:
        self.max = max

    def __aiter__(self):
        self.current = 0
        return self

    async def __anext__(self):
        if self.current >= self.max:
            raise StopAsyncIteration
        await asyncio.sleep(0.1)
        value = self.current
        self.current += 1
        return value

async def main():
    async for num in AsyncCounter(3):
        print(num)  # 0, 1, 2 (с задержкой)

asyncio.run(main())
```

## Async генераторы

```python
async def async_range(max: int):
    """Асинхронный генератор"""
    for i in range(max):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for num in async_range(3):
        print(num)

asyncio.run(main())
```

## Async with

```python
import aiohttp

async def fetch_page(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    html = await fetch_page("http://example.com")
    print(html[:100])

asyncio.run(main())
```

## asyncio.to_thread() — Запуск blocking кода (Python 3.9+)

```python
import asyncio
import time

# Blocking функция (не async)
def blocking_io() -> str:
    with open("large_file.txt", "r") as f:
        return f.read()

def cpu_intensive(n: int) -> int:
    return sum(range(n))

# Запуск в отдельном потоке
async def main():
    # I/O blocking
    content = await asyncio.to_thread(blocking_io)
    print(f"Read {len(content)} chars")

    # CPU-intensive (но лучше использовать multiprocessing)
    result = await asyncio.to_thread(cpu_intensive, 10_000_000)
    print(f"Sum: {result}")

asyncio.run(main())
```

### to_thread с параметрами

```python
# Передача аргументов
async def main():
    result = await asyncio.to_thread(
        requests.get,
        "https://api.example.com/data",
        timeout=10,
    )
    print(result.json())

asyncio.run(main())
```

### aiofiles — асинхронная работа с файлами

```python
import aiofiles

# Чтение
async def read_file(path: str) -> str:
    async with aiofiles.open(path, "r") as f:
        return await f.read()

# Запись
async def write_file(path: str, content: str) -> None:
    async with aiofiles.open(path, "w") as f:
        await f.write(content)
```

## Best Practices

✅ **Используйте** `async`/`await` для I/O операций
✅ **Параллельте** независимые задачи через `gather()`
✅ **Обрабатывайте** ошибки через `return_exceptions=True`
✅ **Используйте** `async with` для контекстных менеджеров
✅ **Используйте** `asyncio.to_thread()` для blocking I/O
✅ **Используйте** `aiofiles` для асинхронной работы с файлами

❌ **Не смешивайте** sync и async код без необходимости
❌ **Не блокируйте** event loop (`time.sleep`, `requests.get`)
❌ **Не используйте** `async` для CPU-bound задач (используйте multiprocessing)

## Ссылки

- [Официальная документация async/await](https://docs.python.org/3/reference/compound_stmts.html#async-def)
- [PEP 492 — async/await syntax](https://peps.python.org/pep-0492/)
- [asyncio.to_thread](https://docs.python.org/3/library/asyncio-task.html#asyncio.to_thread)
