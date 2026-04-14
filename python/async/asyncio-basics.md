# asyncio — Основы асинхронности

## Описание

`asyncio` — стандартная библиотека Python для написания асинхронного кода с использованием `async`/`await`.

## Базовые понятия

```python
import asyncio

# async — объявление корутины
async def greet(name: str) -> str:
    return f"Hello, {name}!"

# await — ожидание выполнения корутины
async def main():
    result = await greet("Alice")
    print(result)  # Hello, Alice!

# Запуск
asyncio.run(main())
```

## Event Loop

```python
import asyncio

# Event Loop — цикл обработки событий
async def main():
    loop = asyncio.get_event_loop()
    print(f"Running loop: {loop}")

asyncio.run(main())

# Ручное управление (редко нужно)
loop = asyncio.new_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

## Sleep — Асинхронная задержка

```python
import asyncio

async def delayed_greet(delay: float, name: str) -> None:
    await asyncio.sleep(delay)  # Не блокирует!
    print(f"Hello, {name}!")

async def main():
    # Параллельное выполнение
    await asyncio.gather(
        delayed_greet(1, "Alice"),
        delayed_greet(2, "Bob"),
        delayed_greet(0.5, "Charlie"),
    )
    # Charlie через 0.5s
    # Alice через 1s
    # Bob через 2s

asyncio.run(main())
```

## asyncio.gather — Параллельное выполнение

```python
import asyncio

async def fetch_data(url: str) -> str:
    print(f"Fetching {url}")
    await asyncio.sleep(1)
    return f"Data from {url}"

async def main():
    # Несколько задач параллельно
    results = await asyncio.gather(
        fetch_data("http://api1.com"),
        fetch_data("http://api2.com"),
        fetch_data("http://api3.com"),
    )
    print(results)
    # ['Data from http://api1.com', 'Data from http://api2.com', ...]

asyncio.run(main())
```

## create_task — Создание задачи

```python
import asyncio

async def background_task():
    while True:
        print("Working...")
        await asyncio.sleep(1)

async def main():
    # Запуск фоновой задачи
    task = asyncio.create_task(background_task())

    # Основная работа
    await asyncio.sleep(3)

    # Отмена задачи
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled")

asyncio.run(main())
```

## Task — Управление задачами

```python
import asyncio

async def slow_task():
    await asyncio.sleep(5)
    return "Done!"

async def main():
    task = asyncio.create_task(slow_task())

    # Проверка статуса
    print(task.done())  # False

    # Таймаут
    try:
        result = await asyncio.wait_for(task, timeout=2)
    except asyncio.TimeoutError:
        print("Timeout!")
        task.cancel()

    # Результат
    if task.done():
        print(task.result())

asyncio.run(main())
```

## asyncio.wait — Ожидание группы

```python
import asyncio

async def task(name: str, delay: float):
    await asyncio.sleep(delay)
    return f"{name} done"

async def main():
    tasks = [
        asyncio.create_task(task("A", 1)),
        asyncio.create_task(task("B", 2)),
        asyncio.create_task(task("C", 3)),
    ]

    # Ожидание всех
    done, pending = await asyncio.wait(tasks)
    for t in done:
        print(t.result())

    # FIRST_COMPLETED — ждать первого
    tasks = [
        asyncio.create_task(task("Fast", 0.5)),
        asyncio.create_task(task("Slow", 2)),
    ]
    done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
    print(f"First done: {done.pop().result()}")

asyncio.run(main())
```

## Асинхронные контекстные менеджеры

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource():
    print("Acquiring")
    resource = {"status": "open"}
    try:
        yield resource
    finally:
        print("Releasing")
        resource["status"] = "closed"

async def main():
    async with async_resource() as resource:
        print(f"Using: {resource['status']}")

asyncio.run(main())
```

## Best Practices

✅ **Используйте** `asyncio.run()` для запуска (Python 3.7+)
✅ **Используйте** `asyncio.gather()` для параллельного выполнения
✅ **Используйте** `asyncio.sleep()` вместо `time.sleep()`
✅ **Обрабатывайте** `asyncio.CancelledError` при отмене задач
✅ **Используйте** `create_task()` для фоновых задач

❌ **Не используйте** `time.sleep()` в async коде (блокирует!)
❌ **Не игнорируйте** `CancelledError` — это может привести к утечкам
❌ **Не запускайте** `asyncio.run()` внутри уже running event loop

## Ссылки

- [Официальная документация asyncio](https://docs.python.org/3/library/asyncio.html)
- [PEP 492 — async/await](https://peps.python.org/pep-0492/)
- [aiohttp — async HTTP клиент](https://docs.aiohttp.org/)
