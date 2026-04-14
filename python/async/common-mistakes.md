# asyncio — Частые ошибки и антипаттерны

## Описание

Распространённые ошибки при работе с asyncio и способы их избежания.

## 🚫 Блокирующий I/O в async функции

```python
# ❌ ПЛОХО — time.sleep блокирует весь event loop
import time

async def bad_sleep():
    time.sleep(1)  # Блокирует все корутины!
    return "done"

# ✅ ХОРОШО — asyncio.sleep неблокирующий
import asyncio

async def good_sleep():
    await asyncio.sleep(1)  # Не блокирует
    return "done"
```

### Блокирующие операции и их замены

| ❌ Блокирующий | ✅ Асинхронная замена |
|---------------|---------------------|
| `time.sleep()` | `asyncio.sleep()` |
| `requests.get()` | `httpx.AsyncClient().get()` |
| `open().read()` | `aiofiles.open().read()` |
| `subprocess.run()` | `asyncio.create_subprocess_exec()` |
| `os.system()` | `asyncio.create_subprocess_shell()` |
| `input()` | `aioconsole.ainput()` |

### asyncio.to_thread() — Запуск blocking кода

```python
import asyncio
import time

# Блокирующая функция (нельзя изменить)
def cpu_intensive_task(n: int) -> int:
    result = 0
    for i in range(n):
        result += i
    return result

def blocking_io() -> str:
    with open("large_file.txt", "r") as f:
        return f.read()

# ✅ Запуск в отдельном потоке
async def main():
    # CPU-bound задача
    result = await asyncio.to_thread(cpu_intensive_task, 10_000_000)
    print(f"Result: {result}")

    # I/O blocking операция
    content = await asyncio.to_thread(blocking_io)
    print(f"Read {len(content)} characters")

asyncio.run(main())
```

### aiofiles — Асинхронная работа с файлами

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

# Построчное чтение
async def read_lines(path: str) -> list[str]:
    lines = []
    async with aiofiles.open(path, "r") as f:
        async for line in f:
            lines.append(line.strip())
    return lines
```

## 🚫 await внутри цикла вместо gather

```python
# ❌ ПЛОХО — последовательное выполнение
async def fetch_sequential(urls: list[str]) -> list[str]:
    results = []
    for url in urls:
        result = await fetch_url(url)  # Ждём каждый запрос
        results.append(result)
    return results  # Общее время = сумма всех запросов

# ✅ ХОРОШО — параллельное выполнение
async def fetch_parallel(urls: list[str]) -> list[str]:
    tasks = [fetch_url(url) for url in urls]
    return await asyncio.gather(*tasks)  # Все запросы одновременно

# ✅ Альтернатива — TaskGroup (Python 3.11+)
async def fetch_with_taskgroup(urls: list[str]) -> list[str]:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_url(url)) for url in urls]
    return [t.result() for t in tasks]
```

### С ограничением конкурентности

```python
# ✅ Semaphore — не больше N одновременных запросов
async def fetch_with_limit(urls: list[str], limit: int = 5) -> list[str]:
    semaphore = asyncio.Semaphore(limit)

    async def fetch_with_semaphore(url: str) -> str:
        async with semaphore:
            return await fetch_url(url)

    tasks = [fetch_with_semaphore(url) for url in urls]
    return await asyncio.gather(*tasks)
```

## 🚫 Утечки задач (неотменённые create_task)

```python
# ❌ ПЛОХО — задача теряется
async def leaky_background():
    asyncio.create_task(background_work())  # Никто не ждёт результат
    return "done"  # Функция завершилась, задача может быть отменена

# ✅ ХОРОШО — ожидание задачи
async def safe_background():
    task = asyncio.create_task(background_work())
    # ... другая работа ...
    await task  # Ждём завершения
    return "done"

# ✅ ХОРОШО — обработка ошибок
async def safe_background_with_error_handling():
    task = asyncio.create_task(background_work())
    task.add_done_callback(lambda t: t.exception() if t.exception() else None)
    return "done"
```

### Правильный паттерн для фоновых задач

```python
import asyncio

class BackgroundService:
    def __init__(self):
        self._tasks: list[asyncio.Task] = []

    def start(self):
        self._tasks = [
            asyncio.create_task(self._worker_1()),
            asyncio.create_task(self._worker_2()),
        ]

    async def stop(self):
        for task in self._tasks:
            task.cancel()
        # Ожидание завершения с обработкой CancelledError
        results = await asyncio.gather(*self._tasks, return_exceptions=True)
        for result in results:
            if isinstance(result, Exception) and not isinstance(result, asyncio.CancelledError):
                print(f"Task error: {result}")
        self._tasks.clear()

    async def _worker_1(self):
        try:
            while True:
                await asyncio.sleep(1)
                # Работа
        except asyncio.CancelledError:
            # Очистка ресурсов
            raise

    async def _worker_2(self):
        try:
            while True:
                await asyncio.sleep(2)
                # Работа
        except asyncio.CancelledError:
            # Очистка ресурсов
            raise

async def main():
    service = BackgroundService()
    service.start()

    try:
        await asyncio.sleep(10)  # Основная работа
    finally:
        await service.stop()  # Graceful shutdown

asyncio.run(main())
```

## 🚫 Игнорирование return_exceptions в gather

```python
# ❌ ПЛОХО — одна ошибка убивает все
async def bad_gather():
    try:
        results = await asyncio.gather(
            success_task(),
            failure_task(),  # Уронит всё
            success_task(),
        )
    except Exception:
        # Потеряем результаты успешных задач
        pass

# ✅ ХОРОШО — собираем все результаты
async def good_gather():
    results = await asyncio.gather(
        success_task(),
        failure_task(),
        success_task(),
        return_exceptions=True,
    )

    successes = []
    errors = []

    for result in results:
        if isinstance(result, Exception):
            errors.append(result)
        else:
            successes.append(result)

    print(f"Success: {len(successes)}, Errors: {len(errors)}")
```

## 🚫 Забытый await

```python
# ❌ ПЛОХО — вызов корутины без await
async def missing_await():
    coro = some_async_function()  # Ничего не выполнится!
    # Coroutine was never awaited — RuntimeWarning

# ✅ ХОРОШО
async def correct_await():
    result = await some_async_function()

# ✅ Или через create_task если нужно позже
async def await_later():
    task = asyncio.create_task(some_async_function())
    # ... другая работа ...
    result = await task
```

## 🚫 Смешивание sync и async

```python
# ❌ ПЛОХО — синхронная функция вызывает async
def sync_calling_async():
    result = asyncio.run(async_function())  # Может вызвать RuntimeError
    return result

# ✅ ХОРОШО — вся цепочка async
async def async_chain():
    result = await async_function()
    return result

# Если нужно вызвать из sync кода
import asyncio

def sync_wrapper():
    loop = asyncio.new_event_loop()
    try:
        return loop.run_until_complete(async_function())
    finally:
        loop.close()
```

## 🚫 Неправильная обработка CancelledError

```python
# ❌ ПЛОХО — игнорирование CancelledError
async def bad_cancellation():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        pass  # Скрыли отмену — задача "зависнет"

# ✅ ХОРОШО — пробрасывание дальше
async def good_cancellation():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        # Очистка ресурсов
        cleanup()
        raise  # Обязательно пробрасываем!

# ✅ С обработкой в Task
async def task_with_handler():
    task = asyncio.create_task(long_running())

    def handle_done(t: asyncio.Task):
        try:
            t.result()
        except asyncio.CancelledError:
            print("Task was cancelled")
        except Exception as e:
            print(f"Task failed: {e}")

    task.add_done_callback(handle_done)
```

## 🚫 Глобальное состояние без синхронизации

```python
# ❌ ПЛОХО — гонка данных
shared_counter = 0

async def increment():
    global shared_counter
    for _ in range(1000):
        current = shared_counter
        await asyncio.sleep(0)  # Переключение контекста
        shared_counter = current + 1

async def main():
    await asyncio.gather(increment(), increment())
    print(shared_counter)  # Будет меньше 2000!

# ✅ ХОРОШО — с блокировкой
shared_counter = 0
lock = asyncio.Lock()

async def safe_increment():
    global shared_counter
    for _ in range(1000):
        async with lock:
            current = shared_counter
            await asyncio.sleep(0)
            shared_counter = current + 1

async def main():
    await asyncio.gather(safe_increment(), safe_increment())
    print(shared_counter)  # 2000
```

## 🚫 Создание клиента на каждый запрос

```python
# ❌ ПЛОХО — новая сессия каждый раз
async def bad_http():
    for url in urls:
        async with httpx.AsyncClient() as client:  # Дорого!
            await client.get(url)

# ✅ ХОРОШО — переиспользование клиента
async def good_http():
    async with httpx.AsyncClient() as client:
        for url in urls:
            await client.get(url)

# ✅ ИЛИ — параллельно
async def parallel_http(urls: list[str]):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        await asyncio.gather(*tasks)
```

## Диагностика проблем

```python
# Включение debug mode
import asyncio

asyncio.run(main(), debug=True)

# Или через переменную окружения
# PYTHONASYNCIODEBUG=1 python script.py

# Получение всех задач
tasks = asyncio.all_tasks()
for task in tasks:
    print(f"Task: {task.get_name()}, Done: {task.done()}")

# Отладка медленных операций
import asyncio

async def slow_operation():
    await asyncio.sleep(5)

# debug покажет предупреждение если задача выполняется долго
```

## Best Practices

✅ **Всегда используйте** `await asyncio.sleep()` вместо `time.sleep()`
✅ **Всегда используйте** `gather()` для параллельного выполнения
✅ **Всегда обрабатывайте** `return_exceptions=True` в gather
✅ **Всегда пробрасывайте** `CancelledError` после cleanup
✅ **Всегда закрывайте** ресурсы (клиенты, соединения)
✅ **Используйте** `asyncio.Lock` для общего состояния
✅ **Используйте** `asyncio.to_thread()` для blocking I/O
✅ **Используйте** `aiofiles` для работы с файлами
✅ **Храните** ссылки на фоновые задачи и отменяйте их

❌ **Не блокируйте** event loop
❌ **Не теряйте** задачи созданные через `create_task`
❌ **Не игнорируйте** `CancelledError`
❌ **Не создавайте** клиенты/соединения в цикле
❌ **Не смешивайте** sync и async без необходимости

## Ссылки

- [asyncio最佳实践](https://docs.python.org/3/library/asyncio-task.html)
- [Common asyncio Mistakes](https://docs.astral.sh/uv/)
