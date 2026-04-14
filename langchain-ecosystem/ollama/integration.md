# Ollama — Интеграция с LangChain

## Описание

Ollama — локальный запуск LLM. Интеграция с LangChain через `ChatOllama`.

## Установка

```bash
# Установка Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Загрузка модели
ollama pull llama3
ollama pull mistral
```

## LangChain интеграция

```python
from langchain_ollama import ChatOllama, OllamaEmbeddings

# Chat модель
llm = ChatOllama(
    model="llama3",
    temperature=0.7,
    base_url="http://localhost:11434",
)

# Вызов
response = llm.invoke("Расскажи о Python")
print(response.content)

# Streaming
for chunk in llm.stream("Tell me a story"):
    print(chunk.content, end="")

# Bind tools
llm_with_tools = llm.bind_tools([search_tool, calculator])
```

## Embeddings

```python
from langchain_ollama import OllamaEmbeddings

embeddings = OllamaEmbeddings(model="nomic-embed-text")

# Векторизация
vectors = embeddings.embed_documents(["text 1", "text 2"])
query_vector = embeddings.embed_query("search query")
```

## Best Practices

✅ **Используйте** для локального тестирования без API ключей
✅ **Настраивайте** `temperature` и `num_ctx` под задачу
✅ **Используйте** `bind_tools` для agent workflows

## Ссылки

- [Ollama сайт](https://ollama.com/)
- [langchain-ollama](https://python.langchain.com/docs/integrations/chat/ollama)
