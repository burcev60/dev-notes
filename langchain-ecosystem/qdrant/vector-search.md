# Qdrant — Векторный поиск

## Описание

Поиск похожих векторов в Qdrant.

## Поиск

```python
results = client.search(
    collection_name="documents",
    query_vector=[0.1, 0.2, ...],
    limit=5
)

for hit in results:
    print(f"Score: {hit.score}, Text: {hit.payload['text']}")
```

## Фильтрация

```python
results = client.search(
    collection_name="documents",
    query_vector=query,
    query_filter=models.Filter(
        must=[models.FieldCondition(key="category", match=models.MatchValue(value="news"))]
    ),
    limit=5
)
```

## Best Practices

✅ **Фильтруйте** до поиска если возможно
✅ **Используйте** payload для метаданных

## Ссылки

- [Qdrant Search](https://qdrant.tech/documentation/concepts/search/)
