# uv — Источники зависимостей

## Описание

uv поддерживает различные источники зависимостей: PyPI, Git, URL, локальные пути, workspace.

## PyPI (по умолчанию)

```toml
[project]
dependencies = [
    "fastapi>=0.110.0",
    "pydantic>=2.0",
]
```

## Git зависимости

```toml
[tool.uv.sources]
fastapi = { git = "https://github.com/tiangolo/fastapi", branch = "main" }
pydantic = { git = "https://github.com/pydantic/pydantic", tag = "v2.5.0" }
my-lib = { git = "https://github.com/user/repo", rev = "abc123" }

# Git LFS
my-lib = { git = "https://github.com/user/repo", rev = "main", lfs = true }
```

```bash
# Из командной строки
uv add "fastapi @ git+https://github.com/tiangolo/fastapi.git@main"
uv add "my-lib @ git+https://github.com/user/repo.git"
```

## URL зависимости

```toml
[tool.uv.sources]
my-package = { url = "https://example.com/my-package-1.0.0.tar.gz" }
wheel-pkg = { url = "https://example.com/pkg-1.0.0-py3-none-any.whl" }
```

## Локальная зависимость

```toml
[tool.uv.sources]
my-local-pkg = { path = "../my-local-pkg", editable = true }
my-local-pkg = { path = "./libs/my-local-pkg" }
```

```bash
# Из командной строки
uv add --editable ../my-local-pkg
uv add ../my-local-pkg
```

## Workspace зависимость

```toml
[tool.uv.sources]
my-core = { workspace = true }
```

## Index (альтернативные репозитории)

```toml
[[tool.uv.index]]
name = "company"
url = "https://pypi.company.com/simple/"
default = true  # Использовать вместо PyPI

[[tool.uv.index]]
name = "private"
url = "https://pypi.private.com/simple/"
explicit = true  # Только для конкретных пакетов
```

```bash
# Из командной строки
uv add --index-url https://pypi.company.com/simple/ package
uv add --extra-index-url https://pypi.private.com/simple/ package
```

```toml
# Конкретный пакет из конкретного индекса
[tool.uv.sources]
private-pkg = { index = "private" }
```

## Find-links

```toml
[tool.uv]
find-links = ["./wheels/", "https://example.com/wheels/"]
```

## Комбинирование источников

```toml
[project]
dependencies = [
    "fastapi>=0.110",
    "my-core",
    "my-utils",
    "internal-pkg",
]

[tool.uv.sources]
my-core = { workspace = true }
my-utils = { path = "../utils", editable = true }
internal-pkg = { index = "company" }
```

## No sources (для публикации)

```bash
# Игнорировать sources при lock — только publishable metadata
uv lock --no-sources

# Игнорировать для конкретного пакета
uv lock --no-sources-package my-local-pkg
```

## Best Practices

✅ **Используйте** `editable = true` для локальной разработки
✅ **Фиксируйте** `rev` или `tag` для Git зависимостей
✅ **Используйте** `workspace = true` для монорепозиториев
✅ **Используйте** `--frozen` в CI

❌ **Не используйте** `branch` для продакшен зависимостей
❌ **Не публикуйте** пакеты с local/git sources

## Ссылки

- [uv Sources](https://docs.astral.sh/uv/concepts/projects/dependencies/#dependency-sources)
- [PEP 508](https://peps.python.org/pep-0508/)
