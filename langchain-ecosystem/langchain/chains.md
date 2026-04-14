# LangChain — Chains и LCEL

## Описание

LangChain Expression Language (LCEL) — декларативный способ построения цепочек через композицию Runnable-объектов.

**Версии:** langchain 1.2.15, langchain-core 1.2.28

## Установка

```bash
pip install langchain langchain-core
```

## Базовый паттерн: prompt | model | parser

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

# Модель
model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# Промпт
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {topic} expert. Answer concisely."),
    ("human", "{question}"),
])

# Цепочка через LCEL
chain = prompt | model | StrOutputParser()

# Вызов
result = chain.invoke({
    "topic": "Python",
    "question": "What are decorators?"
})
print(result)
```

## Runnable-объекты

```python
from langchain_core.runnables import (
    RunnablePassthrough,   # Пропускает input без изменений
    RunnableLambda,        # Оборачивает функцию в Runnable
    RunnableParallel,      # Параллельное выполнение веток
    RunnableBranch,        # Условное ветвление
    RunnableAssign,        # Добавляет поле к dict-состоянию
    RunnablePick,          # Извлекает поле из dict
)

# RunnablePassthrough — передаёт input дальше
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# RunnableLambda — обёртка обычной функции
def extract_json(text: str) -> dict:
    import json
    return json.loads(text)

chain = model | StrOutputParser() | RunnableLambda(extract_json)

# RunnableParallel — несколько веток параллельно
chain = RunnableParallel({
    "summary": summary_chain,
    "entities": entity_chain,
    "sentiment": sentiment_chain,
})
result = chain.invoke({"text": "Some article..."})
# {'summary': '...', 'entities': [...], 'sentiment': 'positive'}
```

## RunnableBranch — условная логика

```python
from langchain_core.runnables import RunnableBranch

def is_math_question(x):
    return "calculate" in x.get("question", "").lower()

math_chain = math_prompt | math_model | StrOutputParser()
general_chain = general_prompt | general_model | StrOutputParser()

branch = RunnableBranch(
    (is_math_question, math_chain),
    general_chain,  # default
)

result = branch.invoke({"question": "Calculate 2+2"})
```

## Конфигурация на лету

```python
from langchain_core.runnables import ConfigurableField

model = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0
).configurable_fields(
    temperature=ConfigurableField(
        id="temperature",
        name="Temperature",
        description="Controls randomness",
    ),
    model=ConfigurableField(
        id="model_name",
        name="Model",
    ),
)

# Использование с конфигурацией
result = chain.invoke(
    {"question": "What is AI?"},
    config={"configurable": {"temperature": 0.7, "model_name": "gpt-4o"}}
)
```

## Streaming

```python
chain = prompt | model | StrOutputParser()

# Потоковый вывод
for chunk in chain.stream({"question": "Tell me a story"}):
    print(chunk, end="", flush=True)

# Асинхронный стриминг
async for chunk in chain.astream({"question": "Tell me a story"}):
    print(chunk, end="", flush=True)
```

## Init-фабрики (langchain 1.x)

```python
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings

# Единая фабрика — ленивая инициализация
model = init_chat_model("openai:gpt-4o-mini")
model = init_chat_model("anthropic:claude-sonnet-4-20250514")

embeddings = init_embeddings("openai:text-embedding-3-small")
```

## Batch

```python
# Параллельный вызов нескольких input
results = chain.batch([
    {"question": "What is Python?"},
    {"question": "What is AI?"},
    {"question": "What is ML?"},
])

# Асинхронный batch
results = await chain.abatch([...])
```

## Best Practices

✅ **Используйте** LCEL (`|`) вместо устаревших `LLMChain`
✅ **Используйте** `RunnableParallel` для параллельных веток
✅ **Используйте** `stream()` для лучшего UX
✅ **Используйте** `init_chat_model()` для абстрагирования от провайдера

❌ **Не используйте** `LLMChain` — он устарел в пользу LCEL
❌ **Не блокируйте** стриминг там, где он доступен

## Ссылки

- [LangChain LCEL Docs](https://python.langchain.com/docs/how_to/#langchain-expression-language-lcel)
- [LangChain API Reference](https://python.langchain.com/api_reference/langchain/)
