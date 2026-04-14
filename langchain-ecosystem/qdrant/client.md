# Qdrant — Клиент и CRUD

## Описание

Qdrant — векторная база данных для хранения и поиска эмбеддингов.

## Подключение

```python
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)
```

## Создание коллекции

```python
client.create_collection(
    collection_name="documents",
    vectors_config=models.VectorParams(size=384, distance=models.Distance.COSINE)
)
```

## Добавление точек

```python
client.upsert(
    collection_name="documents",
    points=[
        models.PointStruct(
            id=1,
            vector=[0.1, 0.2, ...],
            payload={"text": "Hello"}
        )
    ]
)
```

## Best Practices

✅ **Используйте** батчи для вставки
✅ **Индексируйте** payload поля

## Ссылки

- [Qdrant документация](https://qdrant.tech/documentation/)
