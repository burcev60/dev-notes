# Ollama — Установка и запуск

## Описание

Ollama — инструмент для локального запуска LLM.

## Установка

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

## Использование

```bash
# Запуск модели
ollama run llama3

# Список моделей
ollama list

# Загрузка модели
ollama pull llama3

# API
curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Hello"
}'
```

## Best Practices

✅ **Используйте** для локального тестирования
✅ **Выбирайте** модель под задачу

## Ссылки

- [Ollama документация](https://ollama.com/)
