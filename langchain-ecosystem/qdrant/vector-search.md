# Qdrant — Векторный поиск

## Описание

Поиск похожих векторов в Qdrant с фильтрацией, scoring и гибридным поиском.

## Базовый поиск

```python
from qdrant_client.http import models

results = client.search(
    collection_name="documents",
    query_vector=[0.1, 0.2, 0.3, ...],  # 384 элемента
    limit=5,                             # Топ-5 результатов
    query_filter=None,                   # Опциональный фильтр
    with_payload=True,                   # Вернуть payload
    with_vectors=False,                  # Не возвращать векторы
    score_threshold=0.7,                 # Минимальный порог схожести
)

for hit in results:
    print(f"ID: {hit.id}")
    print(f"Score: {hit.score}")         # 0.0 - 1.0 (для COSINE)
    print(f"Payload: {hit.payload}")
```

## Поиск с фильтрацией

```python
# Filter — комбинация условий
query_filter = models.Filter(
    must=[                                # AND — все должны выполняться
        models.FieldCondition(
            key="category",
            match=models.MatchValue(value="news"),
        ),
        models.FieldCondition(
            key="views",
            range=models.Range(gte=100),
        ),
    ],
    should=[                              # OR — хотя бы одно
        models.FieldCondition(
            key="tags",
            match=models.MatchAny(any=["python", "fastapi"]),
        ),
    ],
    must_not=[                            # NOT — ни одно не должно выполняться
        models.FieldCondition(
            key="status",
            match=models.MatchValue(value="archived"),
        ),
    ],
)

results = client.search(
    collection_name="documents",
    query_vector=query,
    query_filter=query_filter,
    limit=10,
)
```

## Типы Match

```python
# Точное совпадение
models.MatchValue(value="python")

# Любое из списка
models.MatchAny(any=["python", "fastapi", "django"])

# Текст (полнотекстовый поиск)
models.MatchText(text="Python programming language")

# Boolean
models.MatchValue(value=True)
```

## Range фильтры

```python
# Числовые диапазоны
models.Range(gte=0, lte=100)      # [0, 100]
models.Range(gt=0, lt=100)        # (0, 100)
models.Range(gte=18)              # [18, ∞)
models.Range(lte=65)              # (-∞, 65]

# Использование
models.FieldCondition(
    key="age",
    range=models.Range(gte=18, lte=65),
)
```

## Geo фильтры

```python
# Точка в радиусе
models.FieldCondition(
    key="location",
    geo_radius=models.GeoRadius(
        center=models.GeoPoint(lat=55.75, lon=37.62),
        radius=10000,  # 10 км в метрах
    ),
)

# Точка в полигоне
models.FieldCondition(
    key="location",
    geo_bounding_box=models.GeoBoundingBox(
        top_left=models.GeoPoint(lat=55.8, lon=37.5),
        bottom_right=models.GeoPoint(lat=55.7, lon=37.7),
    ),
)
```

## Гибридный поиск (vector + full-text)

```python
# Full-text индекс
client.create_payload_index(
    collection_name="documents",
    field_name="text",
    field_schema=models.TextIndexParams(
        type="text",
        tokenizer="word",
        min_token_len=2,
        max_token_len=20,
        lowercase=True,
    ),
)

# Поиск с full-text
results = client.search(
    collection_name="documents",
    query_vector=query,
    query_filter=models.Filter(
        must=[
            models.FieldCondition(
                key="text",
                match=models.MatchText(text="Python programming"),
            )
        ]
    ),
    limit=10,
)
```

## Batch поиск

```python
from qdrant_client.http.models import SearchRequest

results = client.search_batch(
    collection_name="documents",
    requests=[
        models.SearchRequest(
            vector=[0.1, 0.2, ...],
            limit=5,
            filter=models.Filter(must=[...]),
        ),
        models.SearchRequest(
            vector=[0.3, 0.4, ...],
            limit=3,
        ),
    ],
)
# results — список списков (по одному на запрос)
```

## Поиск с group_by

```python
from qdrant_client.http.models import GroupsResult

results = client.search_groups(
    collection_name="documents",
    query_vector=query,
    group_by="category",      # Группировка по полю
    group_size=3,             # Топ-3 на группу
    limit=10,                 # Топ-10 групп
)

for group in results.groups:
    print(f"Group: {group.id}")
    for hit in group.hits:
        print(f"  Score: {hit.score}, Text: {hit.payload['text']}")
```

## Recommend — поиск похожих

```python
# Рекомендация на основе примеров
results = client.recommend(
    collection_name="documents",
    positive=[1, 2, 3],    # ID точек, которые нравятся
    negative=[4, 5],       # ID точек, которые не нравятся
    limit=5,
    with_payload=True,
)

# С весами
results = client.recommend(
    collection_name="documents",
    positive=[
        models.RecommendExample(id=1, weight=2.0),
        models.RecommendExample(id=2, weight=1.0),
    ],
    negative=[
        models.RecommendExample(id=3, weight=1.5),
    ],
    limit=5,
)
```

## Discover — целенаправленный поиск

```python
# Discover — баланс между positive и negative
results = client.discover(
    collection_name="documents",
    target=[0.1, 0.2, ...],    # Целевой вектор
    context=[
        models.ContextExamplePair(
            positive=models.RecommendExample(id=1),
            negative=models.RecommendExample(id=2),
        ),
    ],
    limit=5,
)
```

## Индексы payload

```python
# Keyword индекс (для фильтрации)
client.create_payload_index(
    collection_name="documents",
    field_name="category",
    field_schema=models.KeywordIndexParams(
        type="keyword",
        is_tenant=False,
    ),
)

# Integer индекс
client.create_payload_index(
    collection_name="documents",
    field_name="views",
    field_schema=models.IntegerIndexParams(
        type="integer",
        lookup=True,
        range=True,
    ),
)

# Удаление индекса
client.delete_payload_index(
    collection_name="documents",
    field_name="category",
)
```

## Best Practices

✅ **Используйте** `score_threshold` для отсечения нерелевантных
✅ **Индексируйте** поля для частой фильтрации
✅ **Используйте** `with_vectors=False` если не нужны векторы
✅ **Используйте** `search_batch` для множественных запросов
✅ **Используйте** `group_by` для разнообразия результатов

❌ **Не фильтруйте** без индексов на больших коллекциях
❌ **Не используйте** full-text search без индекса
❌ **Не забывайте** про `timeout` при поиске

## Ссылки

- [Qdrant Search API](https://qdrant.tech/documentation/concepts/search/)
- [Qdrant Filtering](https://qdrant.tech/documentation/concepts/filtering/)
- [Qdrant Payload Indices](https://qdrant.tech/documentation/concepts/indexing/)
