# re — Регулярные выражения

## Описание

Модуль `re` предоставляет поддержку регулярных выражений для поиска, замены и разбора текста.

## Базовое использование

```python
import re

# Поиск первого совпадения
match = re.search(r'\d+', 'Order 123 shipped')
if match:
    print(match.group())  # '123'

# Все совпадения
matches = re.findall(r'\d+', 'a1 b2 c33')
print(matches)  # ['1', '2', '33']

# Итератор (для больших данных)
for match in re.finditer(r'\d+', 'a1 b2 c33'):
    print(f'{match.group()} at {match.start()}')
# 1 at 1, 2 at 4, 33 at 7

# Замена
result = re.sub(r'\d+', 'X', 'a1 b2 c3')
print(result)  # 'aX bX cX'

# Разделение
parts = re.split(r'[,\s]+', 'a,b c  d')
print(parts)  # ['a', 'b', 'c', 'd']
```

## Компиляция паттерна

```python
import re

# Предкомпиляция для повторного использования
pattern = re.compile(r'\d{3}-\d{2}-\d{4}')

# Использование
result = pattern.search('SSN: 123-45-6789')
print(result.group())  # '123-45-6789'

# Проверка полного совпадения
print(pattern.fullmatch('123-45-6789'))  # Match object
print(pattern.fullmatch('123-45-678'))   # None
```

## Флаги

```python
import re

# IGNORECASE — регистронезависимость
re.findall(r'hello', 'Hello HELLO hello', re.IGNORECASE)
# ['Hello', 'HELLO', 'hello']

# MULTILINE — ^ и $ для каждой строки
text = 'first\nsecond\nthird'
re.findall(r'^\w+', text, re.MULTILINE)
# ['first', 'second', 'third']

# DOTALL — . включает newline
re.search(r'.*', 'line1\nline2', re.DOTALL)

# VERBOSE — комментарии в паттерне
pattern = re.compile(r'''
    \d{3}   # area code
    -       # separator
    \d{2}   # first part
    -       # separator
    \d{4}   # second part
''', re.VERBOSE)
```

## Паттерны

```python
# Основные метасимволы
.       # Любой символ кроме \n
\d      # Цифра [0-9]
\D      # Не цифра
\w      # Слово [a-zA-Z0-9_]
\W      # Не слово
\s      # Пробельный символ
\S      # Не пробел
^       # Начало строки
$       # Конец строки
\b      # Граница слова

# Квантификаторы
*       # 0 или больше
+       # 1 или больше
?       # 0 или 1
{n}     # Ровно n
{n,}    # n или больше
{n,m}   # от n до m

# Группы и альтнативы
(...)   # Захватывающая группа
(?:...) # Не захватывающая
(?P<name>...)  # Именованная группа
a|b     # a или b
[abc]   # Символ из набора
[^abc]  # Не из набора
[a-z]   # Диапазон

# Lookaround
(?=...)   # Lookahead (после)
(?!...)   # Negative lookahead
(?<=...)  # Lookbehind (до)
(?<!...)  # Negative lookbehind
```

## Группы

```python
import re

# Захватывающие группы
match = re.search(r'(\d{4})-(\d{2})-(\d{2})', '2024-01-15')
print(match.group(0))  # '2024-01-15' (весь матч)
print(match.group(1))  # '2024'
print(match.group(2))  # '01'
print(match.groups())  # ('2024', '01', '15')

# Именованные группы
pattern = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
match = pattern.search('2024-01-15')
print(match.group('year'))   # '2024'
print(match.groupdict())     # {'year': '2024', 'month': '01', 'day': '15'}

# Не захватывающие группы
re.findall(r'(?:https?://)?(\w+\.\w+)', 'https://google.com')
# ['google.com']
```

## Практические примеры

```python
import re

# Email
email_pattern = re.compile(r'^[\w\.-]+@[\w\.-]+\.\w+$')
bool(email_pattern.match('user@example.com'))  # True

# URL
url_pattern = re.compile(r'https?://[^\s]+')
urls = url_pattern.findall('Visit https://example.com and http://test.org')

# Phone (Russian)
phone_pattern = re.compile(r'(?:\+7|8)[\s\(]*(\d{3})[\s\)]*(\d{3})[\s\-]*(\d{2})[\s\-]*(\d{2})')
match = phone_pattern.search('+7 (495) 123-45-67')
print(f'+7 ({match.group(1)}) {match.group(2)}-{match.group(3)}-{match.group(4)}')

# Извлечение JSON из текста
json_pattern = re.compile(r'\{.*\}', re.DOTALL)
json_str = json_pattern.search('Response: {"key": "value"}').group()

# Валидация IP
ip_pattern = re.compile(r'^(?:[0-9]{1,3}\.){3}[0-9]{1,3}$')
bool(ip_pattern.match('192.168.1.1'))  # True
```

## Best Practices

✅ **Компилируйте** паттерны через `re.compile()` для повторного использования
✅ **Используйте** `re.finditer()` для больших текстов (экономия памяти)
✅ **Используйте** именованные группы для читаемости
✅ **Тестируйте** паттерны на [regex101.com](https://regex101.com/)

❌ **Не используйте** regex для парсинга HTML/XML — используйте BeautifulSoup/lxml
❌ **Избегайте** сложных nested паттернов — они unreadable
❌ **Не забывайте** про `re.escape()` для литеральных строк

## Ссылки

- [Официальная документация re](https://docs.python.org/3/library/re.html)
- [regex101 — тестирование паттернов](https://regex101.com/)
- [regex — расширенный модуль](https://pypi.org/project/regex/)
