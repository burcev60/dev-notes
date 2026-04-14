# asyncio — Обработка ошибок

## Описание

Паттерны обработки ошибок при конкурентном выполнении корутин.

## asyncio.gather — return_exceptions

```python
import asyncio

async def success_task(name: str):
    await asyncio.sleep(0.1)
    return f"Result from {name}"

async def failure_task(name: str, raise_exception: bool):
    await asyncio.sleep(0.1)
    if raise_exception:
        raise ValueError(f"Error in {name}")
    return f"Result from {name}"

async def main():
    # ❌ Без return_exceptions — падает на первой ошибке
    try:
        results = await asyncio.gather(
            success_task("A"),
            failure_task("B", True),
            success_task("C"),
        )
    except ValueError as e:
        print(f"Caught: {e}")  # Только первая ошибка

    # ✅ С return_exceptions — все результаты
    results = await asyncio.gather(
        success_task("A"),
        failure_task("B", True),
        success_task("C"),
        return_exceptions=True,
    )

    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Task {i} failed: {result}")
        else:
            print(f"Task {i}: {result}")

asyncio.run(main())
```

## Фильтрация результатов

```python
async def process_with_errors():
    tasks = [fetch_data(i) for i in range(10)]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    successes = []
    errors = []

    for i, result in enumerate(results):
        if isinstance(result, Exception):
            errors.append((i, result))
            print(f"Task {i} failed: {type(result).__name__}: {result}")
        else:
            successes.append((i, result))

    print(f"Success: {len(successes)}, Errors: {len(errors)}")
    return successes, errors
```

## asyncio.TaskGroup — Структурированная обработка (Python 3.11+)

```python
import asyncio

async def task_group_example():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(success_task("A"))
            tg.create_task(failure_task("B", True))
            tg.create_task(success_task("C"))
    except* ValueError as eg:
        # except* — обработка группы исключений
        for e in eg.exceptions:
            print(f"Error: {e}")
    except* Exception as eg:
        for e in eg.exceptions:
            print(f"Unexpected error: {e}")

asyncio.run(task_group_example())
```

### TaskGroup — отмена при первой ошибке

```python
async def task_group_cancellation():
    async def slow_task():
        try:
            await asyncio.sleep(10)
            return "Done"
        except asyncio.CancelledError:
            print("Task was cancelled!")
            raise

    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(slow_task())
            tg.create_task(failure_task("B", True))  # Уронит группу
    except* ValueError:
        print("Group failed, slow_task was cancelled")

asyncio.run(task_group_cancellation())
```

## Retry паттерн

```python
import asyncio
from functools import wraps

def async_retry(max_attempts: int = 3, delay: float = 1.0, backoff: float = 2.0):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            current_delay = delay
            last_exception = None

            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}, retrying in {current_delay}s")
                    await asyncio.sleep(current_delay)
                    current_delay *= backoff

            raise last_exception  # На случай если цикл завершился
        return wrapper
    return decorator

@async_retry(max_attempts=3, delay=0.5, backoff=2.0)
async def unstable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("API unavailable")
    return "Success!"

async def main():
    try:
        result = await unstable_api_call()
        print(result)
    except ConnectionError as e:
        print(f"All retries failed: {e}")

asyncio.run(main())
```

## Retry с классом

```python
class AsyncRetry:
    def __init__(
        self,
        max_attempts: int = 3,
        base_delay: float = 1.0,
        max_delay: float = 60.0,
        retry_on: type | tuple[type, ...] = Exception,
    ):
        self.max_attempts = max_attempts
        self.base_delay = base_delay
        self.max_delay = max_delay
        self.retry_on = retry_on

    def __call__(self, func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            delay = self.base_delay
            for attempt in range(self.max_attempts):
                try:
                    return await func(*args, **kwargs)
                except self.retry_on as e:
                    if attempt == self.max_attempts - 1:
                        raise
                    print(f"Retry {attempt + 1}/{self.max_attempts}: {e}")
                    await asyncio.sleep(delay)
                    delay = min(delay * 2, self.max_delay)
        return wrapper

@AsyncRetry(
    max_attempts=5,
    base_delay=0.5,
    max_delay=30.0,
    retry_on=(ConnectionError, TimeoutError),
)
async def fetch_with_retry(url: str) -> str:
    # ... HTTP запрос
    pass
```

## Timeout с обработкой ошибок

```python
async def fetch_with_timeout(url: str, timeout: float = 5.0):
    try:
        result = await asyncio.wait_for(fetch_data(url), timeout=timeout)
        return result
    except asyncio.TimeoutError:
        print(f"Timeout fetching {url}")
        return None
    except ConnectionError as e:
        print(f"Connection error for {url}: {e}")
        return None

# Multiple timeouts
async def fetch_with_retries(url: str, timeouts: list[float] = [1.0, 3.0, 10.0]):
    for i, timeout in enumerate(timeouts):
        try:
            result = await asyncio.wait_for(fetch_data(url), timeout=timeout)
            return result
        except asyncio.TimeoutError:
            print(f"Attempt {i + 1} timed out ({timeout}s)")
            if i == len(timeouts) - 1:
                raise
    return None
```

## Graceful shutdown

```python
import signal
import asyncio

async def graceful_shutdown_example():
    shutdown_event = asyncio.Event()

    def handle_signal():
        print("Received shutdown signal")
        shutdown_event.set()

    # Регистрация обработчиков сигналов
    loop = asyncio.get_event_loop()
    for sig in (signal.SIGINT, signal.SIGTERM):
        loop.add_signal_handler(sig, handle_signal)

    # Фоновые задачи
    async def worker():
        while not shutdown_event.is_set():
            print("Working...")
            await asyncio.sleep(1)
        print("Worker shutting down")

    async def another_worker():
        while not shutdown_event.is_set():
            print("Another working...")
            await asyncio.sleep(2)
        print("Another worker shutting down")

    # Запуск
    tasks = [
        asyncio.create_task(worker()),
        asyncio.create_task(another_worker()),
    ]

    # Ожидание сигнала
    await shutdown_event.wait()

    # Отмена задач
    for task in tasks:
        task.cancel()

    # Ожидание завершения
    results = await asyncio.gather(*tasks, return_exceptions=True)
    for result in results:
        if isinstance(result, asyncio.CancelledError):
            print("Task cancelled successfully")
        elif isinstance(result, Exception):
            print(f"Task error: {result}")

asyncio.run(graceful_shutdown_example())
```

## Обработка ошибок в Task

```python
async def task_error_handling():
    async def failing_task():
        await asyncio.sleep(0.1)
        raise ValueError("Something went wrong")

    task = asyncio.create_task(failing_task())

    # Добавление callback для обработки ошибки
    def handle_exception(t: asyncio.Task):
        try:
            t.result()
        except Exception as e:
            print(f"Task failed: {e}")

    task.add_done_callback(handle_exception)

    # Или явное ожидание
    try:
        await task
    except ValueError as e:
        print(f"Caught: {e}")
```

## Best Practices

✅ **Используйте** `return_exceptions=True` когда нужны все результаты
✅ **Используйте** `TaskGroup` для атомарных операций (Python 3.11+)
✅ **Реализуйте** retry с экспоненциальным backoff
✅ **Обрабатывайте** `CancelledError` для graceful shutdown
✅ **Используйте** `add_done_callback` для фоновой обработки ошибок

❌ **Не игнорируйте** ошибки из `gather()` без `return_exceptions=True`
❌ **Не оставляйте** задачи без обработки ошибок (silent failures)
❌ **Не используйте** `except: pass` — логируйте ошибки
❌ **Не забывайте** про `CancelledError` — это не обычное исключение

## Ссылки

- [asyncio.gather](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather)
- [asyncio.TaskGroup](https://docs.python.org/3/library/asyncio-task.html#task-groups)
- [Exception groups (Python 3.11+)](https://docs.python.org/3/tutorial/errors.html#exception-groups)
