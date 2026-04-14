# LangChain — Chains

## Описание

Chains — последовательность вызовов LLM для выполнения задачи.

## Базовая цепочка

```python
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(
    input_variables=["product"],
    template="Что такое {product}?"
)

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.invoke({"product": "AI"})
```

## Sequential Chains

```python
from langchain.chains import SimpleSequentialChain

overall_chain = SimpleSequentialChain(
    chains=[chain1, chain2],
    verbose=True
)
result = overall_chain.invoke("input")
```

## Best Practices

✅ **Используйте** chains для многошаговых задач
✅ **Комбинируйте** разные типы chains

## Ссылки

- [LangChain Chains](https://python.langchain.com/docs/modules/chains/)
