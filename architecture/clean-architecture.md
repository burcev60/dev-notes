# Чистая архитектура

## Описание

Чистая архитектура (Clean Architecture) — подход к организации кода с разделением на слои.

## Слои

```
┌──────────────────────────────────────┐
│         Frameworks & Drivers         │  ← DB, UI, External APIs
├──────────────────────────────────────┤
│          Interface Adapters          │  ← Controllers, Presenters
├──────────────────────────────────────┤
│             Use Cases                │  ← Business Logic
├──────────────────────────────────────┤
│             Entities                 │  ← Domain Models
└──────────────────────────────────────┘
```

## Пример структуры

```
app/
├── domain/           # Entities
│   ├── user.py
│   └── order.py
├── usecases/         # Use Cases
│   ├── create_user.py
│   └── place_order.py
├── adapters/         # Interface Adapters
│   ├── web/
│   └── db/
└── main.py           # Frameworks
```

## Dependency Rule

Зависимости направлены внутрь. Внешние слои зависят от внутренних.

## Best Practices

✅ **Разделяйте** бизнес-логику и инфраструктуру
✅ **Используйте** интерфейсы между слоями

## Ссылки

- [Clean Architecture — Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
