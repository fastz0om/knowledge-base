---
created: 2025-01-27
tags:
  - jenkins
  - active-choice-parameter
  - logging
  - troubleshooting
  - plugin
category: automation
---

# Настройка логгирования Active Choice Parameter

## Обзор

Инструкция по включению логгирования для плагина Active Choice Parameter (org.biouno.unochoice) в Jenkins для отладки и диагностики проблем.

## Настройка логгера

### Шаги настройки

1. **Открыть настройки логгеров Jenkins:**
   - Manage Jenkins → System Log → Add new log recorder

2. **Создать новый логгер:**
   - Нажать "Add new log recorder"
   - Ввести название логгера (например: "Active Choice Parameter Logger")

3. **Настроить логгер:**
   - Нажать "Add" для добавления записи
   - В поле "Logger" указать:
     ```
     org.biouno.unochoice
     ```
   - Выбрать уровень логгирования:
     - **FINEST** - максимальная детализация (все события)
     - **FINER** - подробное логирование
     - **FINE** - детальное логирование
     - **INFO** - информационные сообщения (рекомендуется для начала)
     - **WARNING** - только предупреждения и ошибки
     - **SEVERE** - только критические ошибки

4. **Сохранить настройки**

### Рекомендуемые уровни

- **Для отладки:** FINE или FINER
- **Для production:** INFO или WARNING
- **Для диагностики проблем:** FINEST (с осторожностью, много логов)

## Просмотр логов

После настройки логи будут доступны:

- Manage Jenkins → System Log → [Название вашего логгера]
- Или напрямую через URL: `http://jenkins-url/log/[log-recorder-name]/`

## Использование

Логгер поможет диагностировать:
- Проблемы с выполнением скриптов в Active Choice Parameter
- Ошибки при генерации параметров
- Проблемы с загрузкой данных
- Ошибки вычисления выражений

## Дополнительные логгеры для отладки

Если нужно более детальное логирование, можно добавить:

```
org.biouno.unochoice.model.AbstractActiveChoiceParameter
org.biouno.unochoice.model.ChoiceParameter
org.biouno.unochoice.model.DynamicReferenceParameter
```

## Best Practices

1. **Используйте INFO для начала** - для базовой диагностики
2. **Увеличивайте уровень при необходимости** - только если нужно больше деталей
3. **Ограничивайте время логирования** - не держите FINEST постоянно
4. **Проверяйте производительность** - высокий уровень логирования может замедлить Jenkins

## Связанные заметки

- [[Pipeline Scripts]] - Работа с Jenkins pipelines
- [[Git Timeout Fix]] - Решение проблем с Jenkins

