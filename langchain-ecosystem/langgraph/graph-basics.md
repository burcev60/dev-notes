# LangGraph — Основы StateGraph

## Описание

LangGraph — библиотека для создания stateful, многошаговых приложений с LLM.

## StateGraph

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class GraphState(TypedDict):
    question: str
    answer: str

def generate(state: GraphState):
    answer = llm.invoke(state["question"])
    return {"answer": answer}

workflow = StateGraph(GraphState)
workflow.add_node("generate", generate)
workflow.add_edge("generate", END)
workflow.set_entry_point("generate")

app = workflow.compile()
result = app.invoke({"question": "Что такое AI?"})
```

## Best Practices

✅ **Используйте** StateGraph для сложных workflows
✅ **Определяйте** чёткую схему состояния

## Ссылки

- [LangGraph документация](https://langchain-ai.github.io/langgraph/)
