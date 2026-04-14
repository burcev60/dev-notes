# LangGraph — Human-in-the-loop

## Описание

Возможность вмешательства человека в процесс выполнения графа через interrupt и approve/reject паттерны.

## interrupt() — Остановка для вмешательства

```python
from langgraph.types import interrupt
from langgraph.checkpoint.memory import InMemorySaver

def review_node(state: State) -> dict:
    # Остановка графа и ожидание ввода
    feedback = interrupt("Please review the response and provide feedback:")
    return {"feedback": feedback, "messages": [HumanMessage(content=feedback)]}

builder = StateGraph(State)
builder.add_node("review", review_node)
builder.add_edge(START, "review")
builder.add_edge("review", END)

# Компиляция с checkpointer (обязательно для interrupt)
graph = builder.compile(checkpointer=InMemorySaver())
```

## Использование interrupt

```python
config = {"configurable": {"thread_id": "review_123"}}

# Первый вызов — доходит до interrupt и останавливается
result = graph.invoke(
    {"messages": [HumanMessage("Generate a response")]},
    config,
)
# result содержит состояние до interrupt

# Проверка на interrupt
if result.get("__interrupt__"):
    for interrupt_info in result["__interrupt__"]:
        print(f"Value: {interrupt_info.value}")
        print(f"ID: {interrupt_info.id}")

# Возобновление с ответом
result = graph.invoke(
    Command(resume="The response needs improvement"),
    config,
)
```

## Command — Управление потоком

```python
from langgraph.types import Command

# Обновление состояния + переход к узлу
def decision_node(state: State) -> Command:
    if state["needs_review"]:
        return Command(
            update={"status": "pending_review"},
            goto="review_node",
        )
    return Command(
        update={"status": "approved"},
        goto=END,
    )

# Переход к родительскому графу (вложенные графы)
return Command(graph=Command.PARENT, goto="parent_node")

# Отправка значения в конкретный узел
from langgraph.types import Send

return Command(goto=Send("process_item", {"item": item}))
```

## interrupt_before / interrupt_after

```python
# Автоматическая остановка до/после узла
graph = builder.compile(
    checkpointer=InMemorySaver(),
    interrupt_before=["review_node"],   # Остановить ДО узла
    interrupt_after=["generate_node"],  # Остановить ПОСЛЕ узла
)

# Проверка состояния
state = graph.get_state(config)
print(state.next)  # ('review_node',) — следующий узел

# Возобновление
graph.invoke(None, config)  # Продолжить с текущей позиции
```

## approve/reject паттерн

```python
from langgraph.types import interrupt, Command

def approval_node(state: State) -> dict:
    # Запрос подтверждения
    decision = interrupt("Approve this response? (yes/no)")
    
    if decision.strip().lower() == "yes":
        return Command(goto=END)
    else:
        return Command(
            update={"needs_revision": True},
            goto="regenerate",
        )
```

## Множественные interrupt

```python
def multi_review(state: State) -> dict:
    # Несколько interrupt в одном узле — resume значения соответствуют по порядку
    title = interrupt("Review title:")
    content = interrupt("Review content:")
    
    return {
        "title": title,
        "content": content,
    }

# При возобновлении передаётся список значений
graph.invoke(Command(resume={"Good title", "Good content"}), config)
```

## Best Practices

✅ **Всегда используйте** checkpointer с interrupt
✅ **Давайте** понятные описания в interrupt()
✅ **Обрабатывайте** `__interrupt__` в результате для UI
✅ **Используйте** `interrupt_before` для пред-одобрения

❌ **Не вызывайте** interrupt без checkpointer — ошибка
❌ **Не передавайте** sensitive данные в interrupt описании
❌ **Не забывайте** обрабатывать timeout при ожидании ответа

## Ссылки

- [LangGraph Human-in-the-loop](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/)
- [LangGraph Command API](https://langchain-ai.github.io/langgraph/reference/types/#command)
