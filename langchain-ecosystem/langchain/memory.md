# LangChain — Memory

## Описание

Память позволяет LLM запоминать предыдущие сообщения.

## Types of Memory

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()

# С историей
from langchain.memory import ConversationSummaryMemory

memory = ConversationSummaryMemory(llm=llm)

# С окном
from langchain.memory import ConversationTokenBufferMemory

memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=50)
```

## Best Practices

✅ **Используйте** память для диалогов
✅ **Ограничивайте** длину истории

## Ссылки

- [LangChain Memory](https://python.langchain.com/docs/modules/memory/)
