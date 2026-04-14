# uv — Tool: установка и запуск утилит

## Описание

`uv tool` — аналог pipx. Устанавливает и запускает CLI-утилиты из Python пакетов в изолированных окружениях.

## Установка утилит

```bash
# Установка
uv tool install ruff
uv tool install black
uv tool install "httpie>=3.0"

# С ограничениями
uv tool install ruff --with "ruff-lsp>=0.0.50"

# Переустановка
uv tool install --force ruff

# Из файла требований
uv tool install -r tools.txt
```

## Запуск утилит

```bash
# Запуск установленной утилиты
uv tool run ruff check .
uv tool run black src/

# Одноразовый запуск (без установки)
uv tool run ruff check .

# С временными пакетами
uv tool run --with rich python -c "from rich import print; print('Hello')"

# С файлом требований
uv tool run --with-requirements tools.txt my-cli-tool

# Изолированный запуск
uv tool run --isolated ruff check .

# Из конкретного пакета
uv tool run --from httpie http https://example.com
```

## Управление утилитами

```bash
# Список установленных
uv tool list

# Обновление
uv tool upgrade ruff
uv tool upgrade --all

# Удаление
uv tool uninstall ruff
uv tool uninstall --all

# Директория утилит
uv tool dir
```

## Сравнение с аналогами

| Задача | pipx | uv tool |
|--------|------|---------|
| Установка | `pipx install ruff` | `uv tool install ruff` |
| Запуск | `pipx run ruff` | `uv tool run ruff` |
| Список | `pipx list` | `uv tool list` |
| Обновление | `pipx upgrade-all` | `uv tool upgrade --all` |

## Best Practices

✅ **Используйте** `uv tool install` для глобальных CLI
✅ **Используйте** `uv tool run` для одноразовых команд
✅ **Используйте** `--with` для временных зависимостей
✅ **Не смешивайте** tool install и pip install одного пакета

## Ссылки

- [uv Tools](https://docs.astral.sh/uv/guides/tools/)
