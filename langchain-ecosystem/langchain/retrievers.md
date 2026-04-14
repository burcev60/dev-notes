# LangChain — Retrievers и RAG

## Описание

RAG (Retrieval-Augmented Generation) — генерация ответов с поиском по документам.

## Документы

```python
from langchain_core.documents import Document

# Создание документа
doc = Document(
    page_content="Python is a programming language.",
    metadata={"source": "wiki.txt", "section": "intro"},
)
```

## Text Splitters — Разбиение текста

```python
from langchain_text_splitters import (
    CharacterTextSplitter,
    RecursiveCharacterTextSplitter,
    TokenTextSplitter,
    MarkdownHeaderTextSplitter,
    HTMLHeaderTextSplitter,
    SemanticChunker,
)

# RecursiveCharacterTextSplitter (рекомендуемый)
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,       # Размер чанка в символах
    chunk_overlap=200,     # Перекрытие
    length_function=len,   # Функция подсчёта длины
    separators=["\n\n", "\n", ". ", " ", ""],  # Порядок разделителей
)

docs = splitter.create_documents([long_text])

# С метаданными
splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    chunk_size=500,
    chunk_overlap=100,
    model_name="gpt-4o-mini",  # Считает токены через tiktoken
)

# SemanticChunker — разбиение по семантике
from langchain_experimental.text_splitter import SemanticChunker

splitter = SemanticChunker(embeddings, breakpoint_threshold_type="percentile")
```

## Embeddings

```python
from langchain_openai import OpenAIEmbeddings
from langchain.embeddings import init_embeddings

# OpenAI
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Через фабрику
embeddings = init_embeddings("openai:text-embedding-3-small")

# Векторизация
vectors = embeddings.embed_documents(["text 1", "text 2"])
query_vector = embeddings.embed_query("search query")
```

## Vector Store

```python
from langchain_core.vectorstores import InMemoryVectorStore, VectorStore

# In-memory (для тестов)
vectorstore = InMemoryVectorStore(embeddings)
vectorstore.add_documents(docs)

# Поиск
results = vectorstore.similarity_search("Python", k=3)
results = await vectorstore.asimilarity_search("Python", k=3)

# С scores
results = vectorstore.similarity_search_with_score("Python", k=3)

# Максимальное отклонение (MMR — разнообразие результатов)
results = vectorstore.max_marginal_relevance_search("Python", k=3, fetch_k=10)
```

## Retriever

```python
# Retriever из VectorStore
retriever = vectorstore.as_retriever(
    search_type="similarity",       # "similarity" | "mmr" | "similarity_score_threshold"
    search_kwargs={"k": 5},
)

# Вызов
docs = retriever.invoke("Python programming")
docs = await retriever.ainvoke("Python programming")

# Custom Retriever
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document

class MyRetriever(BaseRetriever):
    def _get_relevant_documents(self, query: str, *, run_manager):
        # Ваша логика поиска
        return [Document(page_content=f"Results for {query}")]

# Retriever как инструмент
from langchain_core.tools import create_retriever_tool

retriever_tool = create_retriever_tool(
    retriever,
    name="document_search",
    description="Search documents for information",
)
```

## RAG Pipeline (LCEL)

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

# Промпт с контекстом
prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer based on context:\n{context}\n\nIf unsure, say you don't know."),
    ("human", "{question}"),
])

# RAG цепочка
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

result = rag_chain.invoke("What is Python?")
```

## Context Compression

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import (
    LLMChainExtractor,
    LLMChainFilter,
    EmbeddingsFilter,
)

# LLM выделяет только релевантное
compressor = LLMChainExtractor.from_llm(model)

# Фильтр — убирает нерелевантные
compressor = LLMChainFilter.from_llm(model)

# Embeddings фильтр — по порогу схожести
compressor = EmbeddingsFilter(embeddings=embeddings, similarity_threshold=0.7)

# Retriever с компрессией
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=retriever,
)
```

## Best Practices

✅ **Используйте** `RecursiveCharacterTextSplitter` с overlap
✅ **Считайте токены** через `from_tiktoken_encoder`
✅ **Используйте** `ContextualCompressionRetriever` для точных ответов
✅ **Добавляйте** источник в metadata документов

❌ **Не создавайте** слишком маленькие чанки (< 200 tokens)
❌ **Не забывайте** overlap для контекста между чанками
❌ **Не используйте** similarity_search без фильтрации для больших баз

## Ссылки

- [LangChain RAG](https://python.langchain.com/docs/how_to/#retrievers)
- [LangChain Vector Stores](https://python.langchain.com/docs/how_to/#vector-stores)
- [LangChain Text Splitters](https://python.langchain.com/docs/concepts/text_splitters/)
