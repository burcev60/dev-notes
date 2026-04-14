# LangChain — Retrievers (RAG)

## Описание

RAG — Retrieval-Augmented Generation, генерация с поиском по документам.

## Базовый RAG

```python
from langchain.text_splitter import CharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

# Разбиение
text_splitter = CharacterTextSplitter(chunk_size=1000)
docs = text_splitter.create_documents([text])

# Векторизация
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(docs, embeddings)

# Retriever
retriever = vectorstore.as_retriever()
relevant_docs = retriever.get_relevant_documents("query")
```

## Best Practices

✅ **Разбивайте** текст на чанки
✅ **Используйте** подходящие embeddings

## Ссылки

- [LangChain RAG](https://python.langchain.com/docs/modules/data_connection/)
