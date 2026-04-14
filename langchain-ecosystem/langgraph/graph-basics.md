# LangGraph — Основы StateGraph

## Описание

LangGraph — библиотека для создания stateful многошаговых приложений с LLM на основе графов. Построена поверх langchain-core.

**Версия:** langgraph 1.1.6

## Установка

```bash
pip install langgraph
```

## Базовый StateGraph

```python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END

# Определение состояния
class State(TypedDict):
    question: str
    answer: str
    messages: list  # история сообщений

# Узлы графа
def generate(state: State) -> dict:
    """Генерация ответа"""
    answer = llm.invoke(state["question"])
    return {"answer": answer.content, "messages": [HumanMessage(state["question"]), AIMessage(answer.content)]}

# Построение графа
builder = StateGraph(State)
builder.add_node("generate", generate)
builder.add_edge(START, "generate")
builder.add_edge("generate", END)

# Компиляция
graph = builder.compile()

# Вызов
result = graph.invoke({"question": "What is AI?"})
print(result["answer"])
```

## Контекст и схемы ввода/вывода

```python
from typing_extensions import TypedDict

# Разделение схем (langgraph 1.x)
class InputState(TypedDict):
    question: str  # Только вход

class OutputState(TypedDict):
    answer: str    # Только выход

class FullState(TypedDict):
    question: str
    answer: str
    internal_data: dict  # Внутреннее состояние

# Context schema — неизменный контекст на весь run
class Context(TypedDict):
    user_id: str
    model: str

builder = StateGraph(
    state_schema=FullState,
    context_schema=Context,       # Run-scoped context
    input_schema=InputState,      # Что принимает invoke()
    output_schema=OutputState,    # Что возвращает invoke()
)

# Доступ к контексту в узле
from langgraph.runtime import Runtime

def generate(state: FullState, runtime: Runtime[Context]) -> dict:
    user_id = runtime.context["user_id"]
    # runtime.store — persistent store
    # runtime.stream_writer — кастомные события
    return {"answer": "..."}
```

## Автоматическое именование узлов

```python
# Автоматическое имя из функции
builder.add_node(generate)  # Узел называется "generate"

# Явное имя
builder.add_node("assistant", generate)

# Узел из Runnable
builder.add_node("model", prompt | model | StrOutputParser())
```

## Условные рёбра

```python
def route(state: State) -> str:
    """Маршрутизация на основе состояния"""
    if "error" in state:
        return "error_handler"
    elif state.get("needs_clarification"):
        return "clarify"
    else:
        return "respond"

builder.add_conditional_edges("assistant", route, {
    "error_handler": "error_handler",
    "clarify": "clarify",
    "respond": "respond",
})  # map_to — для визуализации в графе
```

## Параллельные узлы

```python
# Параллельное выполнение — несколько рёбер к одному узлу
builder.add_edge("analyze_sentiment", "summarize")
builder.add_edge("analyze_entities", "summarize")
# summarize будет запущен после завершения обоих
```

## Stream режимы

```python
# values — полное состояние после каждого узла
for event in graph.stream(input, stream_mode="values"):
    print(event)

# updates — только обновления от текущего узла
for event in graph.stream(input, stream_mode="updates"):
    for node_name, update in event.items():
        print(f"{node_name}: {update}")

# messages — LLM токены (для стриминга ответов)
for event in graph.stream(input, stream_mode="messages"):
    print(event[0].content, end="")  # сообщение + метаданные

# custom — кастомные данные через runtime.stream_writer
for event in graph.stream(input, stream_mode="custom"):
    print(event)

# debug — отладочные события (checkpoint + tasks)
for event in graph.stream(input, stream_mode="debug"):
    print(event)

# tasks — события запуска/завершения задач
for event in graph.stream(input, stream_mode="tasks"):
    print(event)
```

## Functional API

```python
from langgraph.func import task, entrypoint
from langgraph.checkpoint.memory import InMemorySaver

# @task — отдельная задача в графе
@task
def analyze(text: str) -> dict:
    # Выполняется как отдельный узел
    return {"sentiment": "positive", "entities": []}

# @entrypoint — точка входа
@entrypoint(checkpointer=InMemorySaver())
def analyze_workflow(text: str, *, previous=None) -> dict:
    """Workflow как функцияа"""
    # previous — предыдущее состояние (из checkpoint)
    result = analyze(text).result()
    return {"analysis": result}

# Вызов
result = analyze_workflow.invoke("I love Python!")

# entrypoint.final — разделение сохраняемого и возвращаемого значения
@entrypoint(checkpointer=InMemorySaver())
def counter(x: int, *, previous=None) -> entrypoint.final[int, int]:
    prev = previous or 0
    new_value = prev + x
    return entrypoint.final(value=new_value, save=new_value)
```

## RetryPolicy и CachePolicy

```python
from langgraph.types import RetryPolicy, CachePolicy

# Retry с экспоненциальным backoff
retry_policy = RetryPolicy(
    initial_interval=0.5,
    backoff_factor=2.0,
    max_interval=128.0,
    max_attempts=3,
    jitter=True,
    retry_on=(ConnectionError, TimeoutError),  # Типы исключений
)

builder.add_node("api_call", call_api, retry_policy=retry_policy)

# Кэширование результатов
def cache_key(state):
    return state["question"]

builder.add_node(
    "generate",
    generate,
    cache_policy=CachePolicy(key_func=cache_key, ttl=3600),
)
```

## Best Practices

✅ **Используйте** `TypedDict` для определения состояния
✅ **Используйте** `context_schema` для user-scoped данных
✅ **Возвращайте** partial state update из узлов (не всё состояние)
✅ **Используйте** `stream_mode="messages"` для стриминга LLM

❌ **Не храните** всё состояние в одном узле — разбивайте на шаги
❌ **Не забывайте** `retry_policy` для внешних API
❌ **Не используйте** глобальные переменные вместо context

## Ссылки

- [LangGraph документация](https://langchain-ai.github.io/langgraph/)
- [LangGraph StateGraph API](https://langchain-ai.github.io/langgraph/reference/graphs/)
- [LangGraph Functional API](https://langchain-ai.github.io/langgraph/reference/func/)
