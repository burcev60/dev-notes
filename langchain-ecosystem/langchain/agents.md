# LangChain — Agents

## Описание

Агенты — LLM, которые используют инструменты (tools) для выполнения задач. В langchain 1.x / langgraph 1.x агенты создаются через `create_react_agent` (deprecated) или `create_agent`.

## Инструменты (Tools)

```python
from langchain_core.tools import tool, BaseTool, StructuredTool

# @tool декоратор (рекомендуемый способ)
@tool
def search(query: str, max_results: int = 5) -> str:
    """Search the web for information.
    
    Args:
        query: The search query
        max_results: Maximum number of results (default: 5)
    """
    # ... implementation
    return f"Found {max_results} results for '{query}'"

# StructuredTool — с явной схемой
from pydantic import BaseModel

class SearchInput(BaseModel):
    query: str
    max_results: int = 5

from langchain_core.tools import StructuredTool

tool = StructuredTool.from_function(
    func=search_fn,
    name="search",
    description="Search the web",
    args_schema=SearchInput,
)

# Из Runnable (convert_to_tool)
from langchain_core.tools import convert_runnable_to_tool

retriever_tool = convert_runnable_to_tool(
    retriever,
    name="retriever",
    description="Search documents"
)
```

## Создание агента (langgraph prebuilt → langchain.agents)

```python
# ⚠️ create_react_agent из langgraph.prebuilt — deprecated
# ✅ Используйте create_agent из langchain.agents

from langgraph.prebuilt import create_react_agent  # deprecated, но работает

tools = [search_tool, calculator_tool]
model = ChatOpenAI(model="gpt-4o-mini")

agent = create_react_agent(
    model=model,
    tools=tools,
    prompt="You are a helpful assistant.",  # опционально
    checkpointer=checkpointer,              # для сохранения истории
    version="v2",  # v2 — распределяет tool calls через Send API
)

result = agent.invoke({
    "messages": [("human", "What is the capital of France?")]
})
print(result["messages"][-1].content)
```

## ToolNode и tools_condition

```python
from langgraph.prebuilt import ToolNode, tools_condition
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import MessagesState

builder = StateGraph(MessagesState)

# Узел с инструментами
tool_node = ToolNode(tools)
builder.add_node("tools", tool_node)

# Модель
def call_model(state: MessagesState):
    model_with_tools = model.bind_tools(tools)
    response = model_with_tools.invoke(state["messages"])
    return {"messages": [response]}

builder.add_node("assistant", call_model)
builder.add_edge(START, "assistant")

# Условное ребро: если есть tool_calls → "tools", иначе → END
builder.add_conditional_edges("assistant", tools_condition)
builder.add_edge("tools", "assistant")

graph = builder.compile()
```

## Injected аргументы в инструментах (langgraph)

```python
from langchain.tools import InjectedState, InjectedStore, ToolRuntime
from langgraph.store.base import BaseStore

@tool
def update_user(
    user_id: str,
    name: str,
    state: InjectedState,          # Доступ к состоянию графа
    store: InjectedStore,          # Доступ к persistent store
    runtime: ToolRuntime,          # Runtime контекст
) -> str:
    """Update user information."""
    # state["messages"] — история сообщений
    # store.put(("users",), user_id, {"name": name})
    # runtime.stream_writer("Custom event")
    return f"Updated user {user_id}"
```

## AgentState

```python
from langchain.agents import AgentState
# или
from langgraph.prebuilt import AgentState  # deprecated

# AgentState включает:
# messages: Annotated[list, add_messages]
```

## Best Practices

✅ **Описывайте** инструменты подробно (docstring = описание для LLM)
✅ **Используйте** `version="v2"` для параллельного выполнения tool calls
✅ **Используйте** `checkpointer` для сохранения истории агента
✅ **Типизируйте** аргументы инструментов (LLM лучше понимает схемы)

❌ **Не используйте** `create_react_agent` из `langgraph.prebuilt` (deprecated)
❌ **Не создавайте** слишком много инструментов — LLM путается

## Ссылки

- [LangChain Tools](https://python.langchain.com/docs/how_to/#tools)
- [LangGraph Prebuilt](https://langchain-ai.github.io/langgraph/reference/prebuilt/)
