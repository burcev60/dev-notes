# Alembic — Операции

## Описание

Операции для изменения схемы БД в миграциях.

## Основные операции

```python
def upgrade():
    # Создание таблицы
    op.create_table(
        'users',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('email', sa.String(100)),
    )

    # Добавление колонки
    op.add_column('users', sa.Column('age', sa.Integer))

    # Изменение колонки
    op.alter_column('users', 'name', existing_type=sa.String(50), new_type=sa.String(100))

    # Удаление колонки
    op.drop_column('users', 'age')

    # Создание индекса
    op.create_index('idx_user_email', 'users', ['email'], unique=True)

def downgrade():
    op.drop_table('users')
```

## Best Practices

✅ **Всегда реализуйте** `downgrade()`
✅ **Тестируйте** миграции на staging

## Ссылки

- [Alembic Operations](https://alembic.sqlalchemy.org/en/latest/ops.html)
