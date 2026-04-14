# logging — Настройка логирования

## Описание

Модуль `logging` — стандартный инструмент для логирования в Python. Гибкий, настраиваемый, подходит для продакшена.

## Быстрый старт

```python
import logging

# Базовая настройка
logging.basicConfig(level=logging.INFO)

logging.debug('Debug message')      # Не покажется (уровень ниже INFO)
logging.info('Info message')        # INFO:root:Info message
logging.warning('Warning message')  # WARNING:root:Warning message
logging.error('Error message')      # ERROR:root:Error message
logging.critical('Critical message')# CRITICAL:root:Critical message
```

## Уровни логирования

| Уровень | Числовое значение | Когда использовать |
|---------|-------------------|--------------------|
| `DEBUG` | 10 | Детали для отладки |
| `INFO` | 20 | Подтверждение работы |
| `WARNING` | 30 | Предупреждение |
| `ERROR` | 40 | Ошибка выполнения |
| `CRITICAL` | 50 | Критическая ошибка |

## Logger

```python
import logging

# Создание именованного логгера
logger = logging.getLogger(__name__)
logger = logging.getLogger('myapp.module')

# Уровень логгера
logger.setLevel(logging.DEBUG)

# Логирование
logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message', exc_info=True)  # С трейсбеком
logger.critical('Critical message')

# Иерархия логгеров
# root → myapp → myapp.module
# Дочерние наследуют настройки родителя
```

## Handlers — Куда писать

```python
import logging

logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# Console handler
console = logging.StreamHandler()
console.setLevel(logging.INFO)
logger.addHandler(console)

# File handler
file_handler = logging.FileHandler('app.log')
file_handler.setLevel(logging.WARNING)
logger.addHandler(file_handler)

# RotatingFileHandler — ротация по размеру
from logging.handlers import RotatingFileHandler
rotating = RotatingFileHandler(
    'app.log',
    maxBytes=10*1024*1024,  # 10 MB
    backupCount=5
)
logger.addHandler(rotating)

# TimedRotatingFileHandler — ротация по времени
from logging.handlers import TimedRotatingFileHandler
timed = TimedRotatingFileHandler(
    'app.log',
    when='midnight',
    backupCount=30
)
logger.addHandler(timed)
```

## Formatter — Формат сообщений

```python
import logging

# Создание форматтера
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# Применение к хендлеру
handler = logging.StreamHandler()
handler.setFormatter(formatter)

# Доступные переменные:
# %(name)s — имя логгера
# %(levelno)s — числовой уровень
# %(levelname)s — текстовый уровень
# %(pathname)s — полный путь к файлу
# %(filename)s — имя файла
# %(module)s — модуль
# %(lineno)d — номер строки
# %(funcName)s — имя функции
# %(created)f — время создания (timestamp)
# %(message)s — сообщение
```

## dictConfig — Конфигурация из словаря

```python
import logging
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s'
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.FileHandler',
            'level': 'DEBUG',
            'formatter': 'standard',
            'filename': 'app.log'
        }
    },
    'loggers': {
        'myapp': {
            'level': 'DEBUG',
            'handlers': ['console', 'file'],
            'propagate': False
        }
    },
    'root': {
        'level': 'WARNING',
        'handlers': ['console']
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
logger = logging.getLogger('myapp')
```

## Exception logging

```python
import logging

logger = logging.getLogger(__name__)

# Автоматическое добавление трейсбека
try:
    1 / 0
except ZeroDivisionError:
    logger.error('Division failed', exc_info=True)
    # или
    logger.exception('Division failed')  # exc_info=True автоматически

# Без трейсбека
logger.error('Simple error message')
```

## Best Practices

✅ **Используйте** `logging.getLogger(__name__)` для каждого модуля
✅ **Настраивайте** логирование один раз в точке входа (main.py)
✅ **Используйте** `dictConfig()` для сложных конфигураций
✅ **Логируйте** контекст: `logger.info(f"User {user_id} logged in")`
✅ **Используйте** JSON форматер для продакшена (удобно для агрегации)

❌ **Не используйте** `print()` для логирования
❌ **Не логируйте** чувствительные данные (пароли, токены)
❌ **Не логируйте** в цикле на уровне INFO/DEBUG без необходимости
❌ **Не создавайте** логгеры на глобальном уровне без `__name__`

## Ссылки

- [Официальная документация logging](https://docs.python.org/3/library/logging.html)
- [logging.config](https://docs.python.org/3/library/logging.config.html)
- [python-json-logger — JSON форматер](https://github.com/madzak/python-json-logger)
