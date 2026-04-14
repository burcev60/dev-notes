# FastAPI — Background Tasks

## Описание

Выполнение фоновых задач в FastAPI: встроенные `BackgroundTasks` и интеграция с Celery.

## BackgroundTasks (встроенные)

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

def send_email(to: str, subject: str, body: str):
    """Фоновая задача — выполняется после ответа клиенту"""
    time.sleep(1)  # Симуляция отправки
    print(f"Email sent to {to}: {subject}")

@app.post("/users/")
async def create_user(
    data: UserCreate,
    background_tasks: BackgroundTasks,
):
    user = await db.create_user(data)

    # Добавление фоновой задачи
    background_tasks.add_task(send_email, user.email, "Welcome!", "Hello!")

    return user  # Ответ отправится сразу, email — в фоне
```

## Несколько фоновых задач

```python
def write_log(message: str):
    with open("logs.txt", "a") as f:
        f.write(f"{message}\n")

def cleanup_temp_files():
    import os
    for file in os.listdir("/tmp"):
        if file.endswith(".tmp"):
            os.remove(f"/tmp/{file}")

@app.post("/upload/")
async def upload_file(background_tasks: BackgroundTasks):
    # ... обработка файла ...

    background_tasks.add_task(write_log, "File uploaded")
    background_tasks.add_task(cleanup_temp_files)

    return {"status": "uploaded"}
```

## Фоновые задачи с зависимостями

```python
from fastapi import Depends, BackgroundTasks

async def get_notification_service():
    return NotificationService()

async def send_notification(
    user_id: int,
    message: str,
    notification_service: NotificationService = Depends(get_notification_service),
):
    await notification_service.send(user_id, message)

@app.post("/posts/")
async def create_post(
    data: PostCreate,
    background_tasks: BackgroundTasks,
    notification_service: NotificationService = Depends(get_notification_service),
):
    post = await post_service.create(data)

    background_tasks.add_task(
        send_notification,
        post.author_id,
        f"Post '{post.title}' created",
    )

    return post
```

## Ограничения BackgroundTasks

| Характеристика | Описание |
|---------------|----------|
| **Жизненный цикл** | Выполняются в процессе сервера |
| **Отслеживание** | Нет встроенного отслеживания |
| **Повторные попытки** | Нет retry при ошибке |
| **Масштабирование** | Не работают с несколькими workers |
| **Планирование** | Нет отложенного запуска |

## Celery — для сложных фоновых задач

### Установка

```bash
pip install celery[redis]
```

### Конфигурация Celery

```python
# app/celery_app.py
from celery import Celery

celery_app = Celery(
    "worker",
    broker="redis://localhost:6379/0",
    backend="redis://localhost:6379/1",
)

celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_routes={
        "app.tasks.email.*": {"queue": "emails"},
        "app.tasks.analytics.*": {"queue": "analytics"},
    },
    # Retry
    task_acks_late=True,
    task_reject_on_worker_lost=True,
    # Rate limiting
    worker_prefetch_multiplier=1,
)
```

### Создание задач

```python
# app/tasks/email.py
from app.celery_app import celery_app
import smtplib

@celery_app.task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    queue="emails",
)
def send_email_task(self, to: str, subject: str, body: str):
    try:
        # Отправка email
        with smtplib.SMTP("smtp.example.com") as server:
            server.sendmail("noreply@example.com", to, f"Subject: {subject}\n{body}")
    except Exception as e:
        # Retry с экспоненциальным backoff
        raise self.retry(exc=e, countdown=2 ** self.request.retries * 60)
```

### Запуск из FastAPI

```python
from fastapi import FastAPI
from app.tasks.email import send_email_task

app = FastAPI()

@app.post("/users/")
async def create_user(data: UserCreate):
    user = await db.create_user(data)

    # Отправка задачи в Celery
    send_email_task.delay(
        to=user.email,
        subject="Welcome!",
        body="Hello and welcome!",
    )

    return user
```

### Проверка статуса задачи

```python
from celery.result import AsyncResult

@app.get("/tasks/{task_id}")
async def get_task_status(task_id: str):
    result = AsyncResult(task_id)
    return {
        "task_id": task_id,
        "status": result.status,  # PENDING, STARTED, SUCCESS, FAILURE
        "result": result.result if result.ready() else None,
    }

# С результатом
@app.post("/export/")
async def export_data(background_tasks: BackgroundTasks):
    task = export_task.delay()
    return {"task_id": task.id, "status": "started"}
```

### Celery команды

```bash
# Запуск worker
celery -A app.celery_app worker --loglevel=info -Q emails,analytics

# Запуск beat (периодические задачи)
celery -A app.celery_app beat --loglevel=info

# Flower (мониторинг)
celery -A app.celery_app flower --port=5555
```

## Периодические задачи (Celery Beat)

```python
# app/celery_app.py
from celery.schedules import crontab

celery_app.conf.beat_schedule = {
    "cleanup-every-hour": {
        "task": "app.tasks.cleanup.run_cleanup",
        "schedule": crontab(minute=0),  # Каждый час
    },
    "daily-report": {
        "task": "app.tasks.reports.generate_daily_report",
        "schedule": crontab(hour=8, minute=0),  # 8:00 UTC
    },
}
```

## ARQ — async альтернатива Celery

```python
# pip install arq
from arq import create_pool
from arq.connections import RedisSettings
from arq import cron

async def send_email(ctx, to: str, subject: str, body: str):
    # Асинхронная отправка
    await asyncio.sleep(1)
    print(f"Email sent to {to}")

class WorkerSettings:
    functions = [send_email]
    redis_settings = RedisSettings()
    cron_jobs = [
        cron(cleanup_job, hour=0, minute=0),  # Каждый день в полночь
    ]
```

## Best Practices

✅ **Используйте** `BackgroundTasks` для простых задач (email, логирование)
✅ **Используйте** Celery для сложных задач (retry, масштабирование)
✅ **Разделяйте** очереди по типу задач (emails, analytics)
✅ **Настраивайте** retry для надёжности
✅ **Мониторьте** через Flower

❌ **Не используйте** `BackgroundTasks` для критичных операций
❌ **Не забывайте** про обработку ошибок
❌ **Не блокируйте** event loop в фоновых задачах
❌ **Не используйте** in-memory данные в фоновых задачах

## Ссылки

- [FastAPI BackgroundTasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [Celery документация](https://docs.celeryq.dev/)
- [ARQ](https://arq-docs.helpmanual.io/)
