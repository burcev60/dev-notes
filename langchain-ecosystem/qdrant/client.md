# Qdrant — Клиент и CRUD

## Описание

Qdrant — векторная база данных для хранения и поиска эмбеддингов. Поддерживает точный и приближённый поиск, фильтрацию по payload, и гибридный поиск.

**Версия:** qdrant-client 1.17.1

## Установка

```bash
pip install qdrant-client
```

## Подключение

```python
from qdrant_client import QdrantClient
from qdrant_client.http import models

# Локальный режим (in-memory, без сервера)
client = QdrantClient(":memory:")

# Локальный файл
client = QdrantClient(path="my_qdrant.db")

# Remote сервер
client = QdrantClient(
    url="http://localhost:6333",
    api_key="your_api_key",  # опционально
    timeout=60,
)

# gRPC (быстрее)
client = QdrantClient(
    url="http://localhost:6333",
    grpc_port=6334,
    prefer_grpc=True,  # использовать gRPC вместо HTTP
)
```

## Создание коллекции

```python
from qdrant_client.http import models

client.create_collection(
    collection_name="documents",
    vectors_config=models.VectorParams(
        size=384,                        # Размерность вектора
        distance=models.Distance.COSINE, # COSINE | EUCLID | DOT
    ),
    optimizers_config=models.OptimizersConfigDiff(
        indexing_threshold=10000,        # Порог индексации
    ),
    hnsw_config=models.HnswConfigDiff(
        m=16,                            # Размер графа
        ef_construct=100,                # Качество построения
    ),
)
```

## Добавление точек (upsert)

```python
# Одна точка
client.upsert(
    collection_name="documents",
    points=[
        models.PointStruct(
            id=1,
            vector=[0.1, 0.2, 0.3, ...],  # 384 элемента
            payload={
                "text": "Hello world",
                "source": "wiki.txt",
                "category": "greeting",
                "views": 100,
            }
        )
    ]
)

# Массовая вставка
points = []
for i, (text, vector) in enumerate(zip(texts, vectors)):
    points.append(models.PointStruct(
        id=i,
        vector=vector,
        payload={"text": text, "source": "doc.txt"}
    ))

client.upsert(collection_name="documents", points=points)

# Batch размер до 256 точек для оптимальной производительности
```

## Получение точек

```python
# По ID
point = client.retrieve(
    collection_name="documents",
    ids=[1, 2, 3],
    with_payload=True,
    with_vectors=False,  # Не загружать векторы (экономия памяти)
)

# Все точки с фильтром
points = client.scroll(
    collection_name="documents",
    scroll_filter=models.Filter(
        must=[
            models.FieldCondition(
                key="category",
                match=models.MatchValue(value="greeting"),
            )
        ]
    ),
    limit=100,
    with_payload=True,
)
```

## Удаление точек

```python
# По ID
client.delete(
    collection_name="documents",
    points_selector=models.PointIdsList(
        points=[1, 2, 3]
    ),
)

# По фильтру
client.delete(
    collection_name="documents",
    points_selector=models.FilterSelector(
        filter=models.Filter(
            must=[
                models.FieldCondition(
                    key="category",
                    match=models.MatchValue(value="old"),
                )
            ]
        )
    ),
)

# Удаление коллекции
client.delete_collection("documents")
```

## Обновление payload

```python
# Обновление по ID
client.set_payload(
    collection_name="documents",
    payload={"views": 150, "updated": True},
    points=[1, 2, 3],
)

# Обновление по фильтру
client.set_payload(
    collection_name="documents",
    payload={"archived": True},
    points=models.FilterSelector(
        filter=models.Filter(
            must=[
                models.FieldCondition(
                    key="category",
                    match=models.MatchValue(value="old"),
                )
            ]
        )
    ),
)

# Удаление payload ключей
client.delete_payload(
    collection_name="documents",
    keys=["views", "updated"],
    points=[1, 2, 3],
)

# Очистка всего payload
client.clear_payload(
    collection_name="documents",
    points_selector=models.PointIdsList(points=[1, 2, 3]),
)
```

## Async клиент

```python
from qdrant_client import AsyncQdrantClient

async with AsyncQdrantClient(url="http://localhost:6333") as client:
    await client.create_collection(
        collection_name="documents",
        vectors_config=models.VectorParams(size=384, distance=models.Distance.COSINE),
    )
    await client.upsert(collection_name="documents", points=points)
```

## Best Practices

✅ **Используйте** `prefer_grpc=True` для производительности
✅ **Используйте** `with_vectors=False` если не нужны векторы
✅ **Батчите** upsert до 256 точек
✅ **Индексируйте** payload поля для фильтрации

❌ **Не загружайте** векторы без необходимости
❌ **Не используйте** `:memory:` для продакшена
❌ **Не забывайте** про `timeout` при больших операциях

## Ссылки

- [Qdrant документация](https://qdrant.tech/documentation/)
- [Qdrant Python Client](https://qdrant.github.io/qdrant/redoc/index.html)
