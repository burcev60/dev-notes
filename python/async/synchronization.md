# asyncio — Примитивы синхронизации

## Описание

Примитивы синхронизации для координации между корутинами: блокировки, семафоры, события, очереди.

## asyncio.Lock — Взаимное исключение

```python
import asyncio

lock = asyncio.Lock()

async def safe_update(shared_data: dict, key: str, value: str) -> None:
    async with lock:  # Автоматическое acquire/release
        current = shared_data.get(key, "")
        await asyncio.sleep(0.01)  # Симуляция I/O
        shared_data[key] = current + value

async def main():
    data = {"counter": ""}
    await asyncio.gather(
        safe_update(data, "counter", "A"),
        safe_update(data, "counter", "B"),
        safe_update(data, "counter", "C"),
    )
    print(data["counter"])  # "ABC" (в каком-то порядке)

asyncio.run(main())
```

### Ручное управление

```python
lock = asyncio.Lock()

async def manual_lock():
    await lock.acquire()
    try:
        # Критическая секция
        print("Inside lock")
    finally:
        lock.release()
```

## asyncio.Semaphore — Ограничение конкурентности

```python
import asyncio

# Максимум 3 одновременых операции
semaphore = asyncio.Semaphore(3)

async def download(url: str) -> str:
    async with semaphore:
        print(f"Downloading {url}")
        await asyncio.sleep(1)  # Симуляция запроса
        return f"Data from {url}"

async def main():
    # Только 3 задачи выполняются одновременно
    urls = [f"http://api.com/{i}" for i in range(10)]
    results = await asyncio.gather(*[download(url) for url in urls])
    print(f"Downloaded {len(results)} files")

asyncio.run(main())
```

### Rate limiting

```python
class RateLimiter:
    def __init__(self, max_per_second: int):
        self.semaphore = asyncio.Semaphore(max_per_second)
        self.interval = 1.0 / max_per_second

    async def acquire(self):
        async with self.semaphore:
            await asyncio.sleep(self.interval)

# Использование
limiter = RateLimiter(5)  # 5 запросов в секунду

async def api_call(url: str):
    async with limiter:
        return await fetch(url)
```

## asyncio.Event — Сигнализация между корутинами

```python
import asyncio

event = asyncio.Event()

async def waiter(name: str):
    print(f"{name} waiting for event...")
    await event.wait()  # Блокирует до event.set()
    print(f"{name} received event!")

async def setter():
    await asyncio.sleep(2)
    print("Setting event!")
    event.set()  # Разблокирует все waiter

async def main():
    await asyncio.gather(
        waiter("A"),
        waiter("B"),
        waiter("C"),
        setter(),
    )

asyncio.run(main())
```

### Практический пример — готовность сервиса

```python
class Service:
    def __init__(self):
        self._ready = asyncio.Event()
        self._data = None

    async def initialize(self):
        # Длительная инициализация
        await asyncio.sleep(2)
        self._data = "Initialized"
        self._ready.set()  # Сигнал готовности

    async def get_data(self):
        await self._ready.wait()  # Ждёт готовности
        return self._data

async def main():
    service = Service()
    asyncio.create_task(service.initialize())

    # Запрос до готовности — будет ждать
    data = await service.get_data()
    print(data)  # "Initialized"

asyncio.run(main())
```

## asyncio.Condition — Условная синхронизация

```python
import asyncio

condition = asyncio.Condition()
queue = []

async def producer():
    for i in range(3):
        async with condition:
            queue.append(i)
            print(f"Produced {i}")
            condition.notify()  # Разбудить одного consumer
        await asyncio.sleep(0.5)

async def consumer():
    for _ in range(3):
        async with condition:
            while not queue:
                await condition.wait()  # Ждать producer
            item = queue.pop(0)
            print(f"Consumed {item}")

async def main():
    await asyncio.gather(producer(), consumer())

asyncio.run(main())
```

## asyncio.Queue — Очередь задач

```python
import asyncio

queue = asyncio.Queue(maxsize=10)  # Ограниченная очередь

async def producer():
    for i in range(5):
        await queue.put(i)
        print(f"Put {i}")
        await asyncio.sleep(0.1)

    # Сигнал остановки
    await queue.put(None)

async def consumer():
    while True:
        item = await queue.get()
        if item is None:  # Сигнал остановки
            queue.task_done()
            break
        print(f"Processing {item}")
        await asyncio.sleep(0.2)  # Обработка
        queue.task_done()  # Подтверждение обработки

async def main():
    await asyncio.gather(producer(), consumer())

asyncio.run(main())
```

## Producer/Consumer паттерн

### Множественные producers и consumers

```python
import asyncio
from typing import Any

class WorkQueue:
    def __init__(self, maxsize: int = 100):
        self.queue: asyncio.Queue[Any] = asyncio.Queue(maxsize=maxsize)
        self._done = False

    async def put(self, item: Any) -> None:
        await self.queue.put(item)

    async def shutdown(self) -> None:
        self._done = True
        await self.queue.put(None)  # Сигнал для каждого consumer

    async def get(self) -> Any:
        return await self.queue.get()

    def task_done(self) -> None:
        self.queue.task_done()

async def worker(queue: WorkQueue, worker_id: int) -> None:
    while True:
        item = await queue.get()
        if item is None:  # Сигнал остановки
            queue.task_done()
            break

        print(f"Worker {worker_id} processing: {item}")
        await asyncio.sleep(0.5)  # Обработка
        queue.task_done()

async def producer(queue: WorkQueue, count: int) -> None:
    for i in range(count):
        await queue.put(f"Task-{i}")
        print(f"Produced Task-{i}")
        await asyncio.sleep(0.1)

async def main():
    work_queue = WorkQueue()
    num_workers = 3

    # Запуск workers
    workers = [
        asyncio.create_task(worker(work_queue, i))
        for i in range(num_workers)
    ]

    # Producer
    await producer(work_queue, 10)

    # Сигнал остановки для всех workers
    for _ in range(num_workers):
        await work_queue.put(None)

    # Ожидание завершения
    await asyncio.gather(*workers)
    print("All tasks completed!")

asyncio.run(main())
```

### Очередь с приоритетами

```python
import asyncio
import heapq

class PriorityAsyncQueue:
    def __init__(self):
        self._queue: list[tuple[int, Any]] = []
        self._counter = 0
        self._event = asyncio.Event()
        self._lock = asyncio.Lock()

    async def put(self, priority: int, item: Any) -> None:
        async with self._lock:
            heapq.heappush(self._queue, (priority, self._counter, item))
            self._counter += 1
            self._event.set()

    async def get(self) -> tuple[int, Any]:
        while True:
            async with self._lock:
                if self._queue:
                    priority, _, item = heapq.heappop(self._queue)
                    if not self._queue:
                        self._event.clear()
                    return priority, item
            await self._event.wait()

async def producer(queue: PriorityAsyncQueue) -> None:
    items = [
        (3, "Low priority"),
        (1, "High priority!"),
        (2, "Medium priority"),
        (1, "Urgent!"),
    ]
    for priority, item in items:
        await queue.put(priority, item)
        print(f"Produced: [{priority}] {item}")
        await asyncio.sleep(0.1)

async def consumer(queue: PriorityAsyncQueue, name: str) -> None:
    for _ in range(4):
        priority, item = await queue.get()
        print(f"{name}: Got [{priority}] {item}")
        await asyncio.sleep(0.3)

async def main():
    queue = PriorityAsyncQueue()
    await asyncio.gather(
        producer(queue),
        consumer(queue, "Worker-A"),
    )

asyncio.run(main())
```

### Barrier — Синхронизация на контрольной точке

```python
import asyncio

async def barrier_example():
    barrier = asyncio.Barrier(3)  # Ждём 3 корутины

    async def worker(name: str):
        print(f"{name} starting...")
        await asyncio.sleep(1)
        print(f"{name} waiting at barrier...")
        await barrier.wait()
        print(f"{name} passed barrier!")

    await asyncio.gather(
        worker("A"),
        worker("B"),
        worker("C"),
    )

asyncio.run(barrier_example())
```

## Best Practices

✅ **Используйте** `asyncio.Queue` для producer/consumer паттернов
✅ **Используйте** `Semaphore` для ограничения конкурентности
✅ **Используйте** `Lock` для защиты общего состояния
✅ **Используйте** `Event` для сигнализации между корутинами
✅ **Всегда вызывайте** `task_done()` после обработки элемента
✅ **Используйте** `async with` для автоматического release

❌ **Не блокируйте** event loop внутри `async with lock`
❌ **Не забывайте** `task_done()` — `join()` будет ждать вечно
❌ **Не используйте** `Queue` без maxsize — может переполнить память
❌ **Не создавайте** слишком много Semaphore — может привести к deadlock

## Ссылки

- [Официальная документация asyncio primitives](https://docs.python.org/3/library/asyncio-sync.html)
- [asyncio.Queue](https://docs.python.org/3/library/asyncio-queue.html)
