# datetime — Работа с датой и временем

## Описание

Модуль `datetime` предоставляет классы для работы с датами, временем и интервалами.

## Основные классы

```python
from datetime import date, time, datetime, timedelta

# date — только дата
d = date(2024, 1, 15)
print(d.year, d.month, d.day)  # 2024 1 15

# time — только время
t = time(14, 30, 45)
print(t.hour, t.minute, t.second)  # 14 30 45

# datetime — дата и время
dt = datetime(2024, 1, 15, 14, 30, 45)
print(dt)  # 2024-01-15 14:30:45
```

## Текущая дата и время

```python
from datetime import datetime

# Локальное время
now = datetime.now()
print(now)  # 2024-01-15 14:30:45.123456

# UTC время
utc_now = datetime.utcnow()

# Timestamp → datetime
dt = datetime.fromtimestamp(1705312245)
dt = datetime.fromtimestamp(1705312245, tz=timezone.utc)

# datetime → timestamp
ts = datetime.now().timestamp()
```

## Парсинг и форматирование

```python
from datetime import datetime

# Парсинг из строки
dt = datetime.strptime('2024-01-15 14:30', '%Y-%m-%d %H:%M')
dt = datetime.strptime('15/01/2024', '%d/%m/%Y')

# Форматирование в строку
print(dt.strftime('%Y-%m-%d %H:%M:%S'))
# '2024-01-15 14:30:00'
print(dt.strftime('%d.%m.%Y'))
# '15.01.2024'

# ISO формат
dt = datetime.fromisoformat('2024-01-15T14:30:45')
print(dt.isoformat())  # '2024-01-15T14:30:45'
```

## Директивы форматирования

| Директива | Описание | Пример |
|-----------|----------|--------|
| `%Y` | Год (4 знака) | 2024 |
| `%m` | Месяц (01-12) | 01 |
| `%d` | День (01-31) | 15 |
| `%H` | Час (00-23) | 14 |
| `%M` | Минута (00-59) | 30 |
| `%S` | Секунда (00-59) | 45 |
| `%y` | Год (2 знака) | 24 |
| `%B` | Название месяца | January |
| `%A` | Название дня | Monday |

## timedelta — Интервалы времени

```python
from datetime import datetime, timedelta

# Создание
delta = timedelta(days=7, hours=3, minutes=30)

# Сложение/вычитание
now = datetime.now()
week_later = now + timedelta(days=7)
hour_ago = now - timedelta(hours=1)

# Разница между датами
d1 = datetime(2024, 1, 1)
d2 = datetime(2024, 1, 15)
diff = d2 - d1
print(diff.days)  # 14
print(diff.total_seconds())  # 1209600.0

# Практические примеры
yesterday = datetime.now() - timedelta(days=1)
tomorrow = datetime.now() + timedelta(days=1)
next_week = datetime.now() + timedelta(weeks=1)
```

## Timezone (Python 3.9+)

```python
from datetime import datetime, timezone, timedelta
from zoneinfo import ZoneInfo  # Python 3.9+

# UTC
utc_now = datetime.now(timezone.utc)

# Свой timezone
msk = timezone(timedelta(hours=3))
msk_time = datetime.now(msk)

# ZoneInfo (рекомендуется)
mow = ZoneInfo('Europe/Moscow')
mow_time = datetime.now(mow)
print(mow_time)  # 2024-01-15 17:30:00+03:00

# Конвертация
utc_dt = datetime.now(timezone.utc)
mow_dt = utc_dt.astimezone(ZoneInfo('Europe/Moscow'))
```

## Практические рецепты

```python
from datetime import datetime, timedelta

# Начало дня
start_of_day = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)

# Конец дня
end_of_day = datetime.now().replace(hour=23, minute=59, second=59)

# Первый день месяца
first_day = datetime.now().replace(day=1)

# Последний день месяца
import calendar
year, month = 2024, 2
last_day = calendar.monthrange(year, month)[1]

# Проверка високосного года
import calendar
calendar.isleap(2024)  # True

# Генерация диапазона дат
def date_range(start, end):
    for i in range((end - start).days + 1):
        yield start + timedelta(days=i)

for d in date_range(datetime(2024, 1, 1), datetime(2024, 1, 5)):
    print(d.strftime('%Y-%m-%d'))
```

## Best Practices

✅ **Используйте** `datetime.now(timezone.utc)` для серверного времени
✅ **Используйте** `zoneinfo.ZoneInfo` (Python 3.9+) для часовых поясов
✅ **Храните** даты в UTC, конвертируйте только при отображении
✅ **Используйте** `isoformat()` для сериализации

❌ **Не используйте** `datetime.utcnow()` — она naive (без timezone)
❌ **Не смешивайте** aware и naive datetime объекты

## Ссылки

- [Официальная документация datetime](https://docs.python.org/3/library/datetime.html)
- [zoneinfo](https://docs.python.org/3/library/zoneinfo.html)
- [dateutil — расширения для datetime](https://dateutil.readthedocs.io/)
