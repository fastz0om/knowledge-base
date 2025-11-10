---
created: 2025-01-27
tags:
  - database
  - clickhouse
  - sql
  - queries
category: infrastructure
---

# ClickHouse Database

## Обзор

Операции с базой данных ClickHouse, полезные запросы и работа с системными таблицами.

## Запросы к системным настройкам

### Список системных настроек

Список всех системных настроек, соответствующих паттерну:

```sql
SELECT *
FROM system.settings
WHERE name LIKE '%_live_%'
```

## Системные таблицы

**Основные системные таблицы:**
- `system.settings` - Системные настройки
- `system.processes` - Выполняющиеся запросы
- `system.tables` - Таблицы базы данных
- `system.columns` - Столбцы таблиц
- `system.parts` - Части таблиц
- `system.merges` - Операции слияния

## Полезные запросы

### Проверка производительности запросов

```sql
SELECT 
    query,
    elapsed,
    memory_usage,
    read_rows,
    read_bytes
FROM system.processes
ORDER BY elapsed DESC;
```

### Поиск больших таблиц

```sql
SELECT 
    database,
    table,
    formatReadableSize(sum(bytes)) as size
FROM system.parts
GROUP BY database, table
ORDER BY sum(bytes) DESC;
```

### Информация о таблицах

```sql
-- Список всех таблиц
SELECT 
    database,
    name,
    engine,
    total_rows
FROM system.tables
WHERE database != 'system'
ORDER BY database, name;

-- Столбцы таблицы
SELECT 
    database,
    table,
    name,
    type,
    default_kind
FROM system.columns
WHERE database = 'default' AND table = 'my_table';
```

### Мониторинг операций

```sql
-- Текущие операции слияния
SELECT 
    database,
    table,
    progress,
    merge_type
FROM system.merges
ORDER BY create_time DESC;
```

## Рекомендации

1. **Индексы** - Используйте индексы для улучшения производительности запросов
2. **Партиционирование** - Партиционируйте большие таблицы по дате или другим критериям
3. **Мониторинг** - Регулярно проверяйте `system.processes` для долгих запросов
4. **Очистка** - Регулярно удаляйте старые данные для экономии места

## Связанные заметки

- [[../Jenkins/Pipeline Scripts]]

