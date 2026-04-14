# LangChain — Agents

## Описание

Агенты — LLM которые используют инструменты для выполнения задач.

## Создание агента

```python
from langchain.agents import initialize_agent, Tool

tools = [
    Tool(
        name="Search",
        func=search.run,
        description="Полезно для поиска"
    )
]

agent = initialize_agent(
    tools,
    llm,
    agent="zero-shot-react-description",
    verbose=True
)

result = agent.invoke("Найди информацию о Python")
```

## Best Practices

✅ **Описывайте** инструменты подробно
✅ **Используйте** агентов для сложных задач

## Ссылки

- [LangChain Agents](https://python.langchain.com/docs/modules/agents/)
