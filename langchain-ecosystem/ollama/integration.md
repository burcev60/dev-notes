# Ollama — Интеграция с LangChain

## Описание

Подключение Ollama к LangChain.

## Использование

```python
from langchain_community.llms import Ollama

llm = Ollama(model="llama3", base_url="http://localhost:11434")

response = llm.invoke("Расскажи о Python")
print(response)
```

## Best Practices

✅ **Используйте** для локальной разработки
✅ **Настраивайте** параметры модели

## Ссылки

- [LangChain Ollama](https://python.langchain.com/docs/integrations/llms/ollama)
