# concurrent-execution — gather, as_completed, TaskGroup

## Описание

Паттерны конкурентного выполнения задач в asyncio.

## asyncio.gather — Групповое ожидание

```python
import asyncio

async def task(name: str, delay: float) -> str:
    print(f"Starting {name}")
    await asyncio.sleep(delay)
    print(f"Finished {name}")
    return f"Result from {name}"

async def main():
    # Параллельный запуск
    results = await asyncio.gather(
        task("A", 1),
        task("B", 2),
        task("C", 0.5),
    )
    print(results)
    # ['Result from A', 'Result from B', 'Result from C']

asyncio.run(main())
```

### gather с динамическим списком

```python
async def fetch_all(urls: list[str]) -> list[dict]:
    # Распаковка списка
    tasks = [fetch_url(url) for url in urls]
    return await asyncio.gather(*tasks)

# С нумерацией
async def fetch_with_index(urls: list[str]) -> list[tuple[int, str]]:
    async def fetch_with_idx(idx, url):
        result = await fetch_url(url)
        return (idx, result)

    tasks = [fetch_with_idx(i, url) for i, url in enumerate(urls)]
    return await asyncio.gather(*tasks)
```

## asyncio.as_completed — По готовности

```python
import asyncio

async def main():
    # Задачи завершаются в разном порядке
    for coro in asyncio.as_completed([
        task("Fast", 0.5),
        task("Medium", 1),
        task("Slow", 2),
    ]):
        result = await coro
        print(f"Got: {result}")  # Fast, Medium, Slow

asyncio.run(main())
```

## asyncio.wait — Гибкое ожидание

```python
import asyncio

async def main():
    tasks = [
        asyncio.create_task(task("A", 1)),
        asyncio.create_task(task("B", 2)),
        asyncio.create_task(task("C", 3)),
    ]

    # Ожидание всех
    done, pending = await asyncio.wait(tasks)
    print(f"Done: {len(done)}, Pending: {len(pending)}")

    # FIRST_COMPLETED
    tasks = [
        asyncio.create_task(task("Fast", 0.1)),
        asyncio.create_task(task("Slow", 5)),
    ]
    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED
    )
    for t in done:
        print(f"First: {t.result()}")

    # Отмена оставшихся
    for t in pending:
        t.cancel()

asyncio.run(main())
```

## TaskGroup — Python 3.11+

```python
import asyncio

async def main():
    # Все задачи выполняются параллельно
    # Если одна падает — остальные отменяются
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_data("url1"))
        task2 = tg.create_task(fetch_data("url2"))
        task3 = tg.create_task(fetch_data("url3"))

    # Результаты доступны после выхода
    print(task1.result(), task2.result(), task3.result())

asyncio.run(main())
```

### TaskGroup обработка ошибок

```python
async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(good_task())
            tg.create_task(bad_task())  # Вызовет исключение
    except* ValueError as eg:
        for e in eg.exceptions:
            print(f"Error: {e}")

asyncio.run(main())
```

## Таймауты

```python
import asyncio

async def with_timeout():
    try:
        result = await asyncio.wait_for(
            slow_operation(),
            timeout=5.0
        )
        return result
    except asyncio.TimeoutError:
        print("Operation timed out")
        return None

async def slow_operation():
    await asyncio.sleep(10)
    return "Done"
```

## Паттерн: Worker Pool

```python
import asyncio
from asyncio import Queue

async def worker(queue: Queue, worker_id: int) -> None:
    while True:
        task = await queue.get()
        if task is None:  # Signal to stop
            queue.task_done()
            break
        print(f"Worker {worker_id} processing {task}")
        await asyncio.sleep(0.1)
        queue.task_done()

async def main():
    queue = Queue()

    # Добавление задач
    for i in range(10):
        await queue.put(f"Task {i}")

    # Запуск воркеров
    workers = [
        asyncio.create_task(worker(queue, i))
        for i in range(3)
    ]

    # Ожидание выполнения
    await queue.join()

    # Сигнал остановки
    for _ in range(3):
        await queue.put(None)

    await asyncio.gather(*workers)

asyncio.run(main())
```

## Семáфор — Ограничение конкурентности

```python
import asyncio

async def limited_fetch(semaphore: asyncio.Semaphore, url: str) -> str:
    async with semaphore:
        print(f"Fetching {url}")
        await asyncio.sleep(0.5)
        return f"Data from {url}"

async def main():
    # Максимум 3 одновременных запроса
    sem = asyncio.Semaphore(3)

    tasks = [
        limited_fetch(sem, f"http://api.com/{i}")
        for i in range(10)
    ]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

## Best Practices

✅ **Используйте** `gather()` когда нужны все результаты
✅ **Используйте** `as_completed()` для обработки по готовности
✅ **Используйте** `TaskGroup` (3.11+) для атомарности
✅ **Используйте** `Semaphore` для ограничения нагрузки
✅ **Используйте** `wait_for()` для таймаутов

❌ **Не создавайте** тысячи задач без лимита (перегрузка)
❌ **Не игнорируйте** `CancelledError` в критичных задачах
❌ **Не используйте** `asyncio.wait()` без `return_when` если не нужно

## Ссылки

- [Официальная документация asyncio](https://docs.python.org/3/library/asyncio-task.html)
- [asyncio.TaskGroup (Python 3.11+)](https://docs.python.org/3/library/asyncio-task.html#task-groups)
