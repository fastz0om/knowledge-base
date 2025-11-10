---
created: 2025-01-27
tags:
  - jenkins
  - build-number
  - reset
  - script-console
  - groovy
category: automation
---

# Сброс индекса номера сборки в Jenkins

## Обзор

Инструкция по откату индекса номера сборки на определенное значение (простыми словами: начать отчет номера билдов с начала или с конкретного индекса).

> **⚠️ Важно!**
> При использовании скрипта существующие билды будут очищены (т.е. фактически информация о них будет удалена)

## Базовый скрипт

```groovy
def pipelineName = "techJobs/deployFrontend"
job = Jenkins.instance.getItemByFullName(pipelineName)
job.getBuilds().each {
    it.delete() 
}
job.updateNextBuildNumber(1)
```

## Параметры

### pipelineName

**Полное название пайплайна (Full Project Name)** - полный путь к пайплайну в Jenkins.

**Как узнать:**
- Перейти в любой пайплайн
- В адресной строке или в информации о проекте будет указан полный путь
- Например: `techJobs/deployFrontend` или `DevOps/Backend/test-publish-conventions-build/master`
### Значение следующего номера сборки

В `job.updateNextBuildNumber(1)` можно передать любое значение:
- `1` - начать с начала (первая сборка будет #1)
- `100` - следующая сборка будет #100
- Любое другое число

## Для Multibranch Pipeline

Если вы используете Multibranch Pipeline, необходимо передать полный путь, **включая ветку**, в которой необходимо сбросить индекс номера сборки.

**Пример:**
```groovy
def pipelineName = "DevOps/Backend/test-publish-conventions-build/master"
job = Jenkins.instance.getItemByFullName(pipelineName)
job.getBuilds().each {
    it.delete() 
}
job.updateNextBuildNumber(1)
```

**Формат:** `<folder>/<folder>/.../<multibranch-name>/<branch-name>`

## Выполнение скрипта

Скрипт необходимо выполнить в **Jenkins → Manage Jenkins → Script Console**.

### Шаги:

1. Открыть: **Manage Jenkins → Script Console**
2. Вставить скрипт с нужным именем пайплайна
3. Нажать **Run**
4. Проверить результат (обычно нет вывода, если все успешно)

## Примеры использования

### Сброс до начала (сборка #1)

```groovy
def pipelineName = "techJobs/deployFrontend"
job = Jenkins.instance.getItemByFullName(pipelineName)
job.getBuilds().each {
    it.delete() 
}
job.updateNextBuildNumber(1)
```

### Начало с определенного номера

```groovy
def pipelineName = "my-project/my-pipeline"
def nextBuildNumber = 50

job = Jenkins.instance.getItemByFullName(pipelineName)
job.getBuilds().each {
    it.delete() 
}
job.updateNextBuildNumber(nextBuildNumber)
```

### Сброс для Multibranch Pipeline

```groovy
def pipelineName = "DevOps/Backend/test-publish-conventions-build/master"
job = Jenkins.instance.getItemByFullName(pipelineName)
job.getBuilds().each {
    it.delete() 
}
job.updateNextBuildNumber(1)
```

### Функция для повторного использования

```groovy
def resetBuildNumber(pipelineName, nextNumber = 1) {
    def job = Jenkins.instance.getItemByFullName(pipelineName)
    if (!job) {
        println "Pipeline '${pipelineName}' not found!"
        return false
    }
    
    job.getBuilds().each {
        it.delete() 
    }
    job.updateNextBuildNumber(nextNumber)
    println "Build number reset to ${nextNumber} for '${pipelineName}'"
    return true
}

// Использование
resetBuildNumber("techJobs/deployFrontend", 1)
```

## Последствия

### Что удаляется:

- ✅ История сборок (builds)
- ✅ Логи сборок
- ✅ Артефакты сборок (если не настроено внешнее хранилище)
- ✅ Ссылки на сборки в других системах

### Что сохраняется:

- ✅ Конфигурация пайплайна
- ✅ Параметры пайплайна
- ✅ Настройки проекта
- ✅ Триггеры и хуки

## Рекомендации

1. **Резервное копирование** - Если важны логи/артефакты, сделайте резервную копию перед сбросом
2. **Проверка зависимостей** - Убедитесь, что нет внешних ссылок на номера сборок
3. **Коммуникация** - Уведомите команду о сбросе номеров
4. **Тестирование** - Сначала проверьте на тестовом проекте
5. **Документация** - Задокументируйте причину сброса

## Script Approval

Для выполнения скрипта могут потребоваться одобренные методы:

- `method hudson.model.Item getItemByFullName`
- `method hudson.model.AbstractProject getBuilds`
- `method hudson.model.Run delete`
- `method hudson.model.AbstractProject updateNextBuildNumber`

Одобрите их в **Manage Jenkins → In-process Script Approval**.

## Troubleshooting

### Pipeline не найден

```
Pipeline 'xxx' not found!
```

**Решение:** Проверьте правильность полного пути к пайплайну. Используйте точный путь как в адресной строке Jenkins.

### Нет прав доступа

**Решение:** Убедитесь, что у вас есть права администратора Jenkins.

### Скрипт не выполняется

**Решение:**
1. Проверьте одобрение методов в Script Approval
2. Проверьте логи Jenkins на наличие ошибок
3. Убедитесь, что пайплайн существует и доступен

## Альтернативный способ (через UI)

Если у вас есть доступ к Jenkins CLI или UI, можно использовать:

```bash
# Через Jenkins CLI
java -jar jenkins-cli.jar -s http://jenkins-url/ set-build-number <job-name> <build-number>
```

Однако CLI может не поддерживать эту функциональность во всех версиях Jenkins.

## Связанные заметки

- [[Pipeline Scripts]] - Другие полезные скрипты для Jenkins
- [[Active Choice Parameter Logging]] - Настройка логгирования

