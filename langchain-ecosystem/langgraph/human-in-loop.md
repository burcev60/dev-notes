# LangGraph — Human-in-the-loop

## Описание

Возможность вмешательства человека в процесс выполнения графа.

## Interrupt

```python
from langgraph.graph import StateGraph

workflow.add_node("human_review", human_review)
workflow.add_edge("human_review", "continue")

# Точка остановки
app = workflow.compile(
    interrupt_before=["human_review"]
)

# Продолжение после ревью
result = app.invoke(None, config)
```

## Best Practices

✅ **Используйте** interrupt для критичных шагов
✅ **Давайте** человеку контекст для решения

## Ссылки

- [LangGraph Human-in-the-loop](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/)
