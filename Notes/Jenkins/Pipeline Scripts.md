---
created: 2025-01-27
tags: [jenkins, groovy, pipeline, automation, ci/cd]
category: automation
---

# Скрипты Jenkins Pipeline

## Обзор

Полезные Groovy скрипты и функции для Jenkins pipelines: управление контейнерами, версионирование, управление задачами и шаблонизация.

## Управление контейнерами

### Остановка и удаление контейнера по имени

```groovy
script {
    containerName = 'tsetba'
    oldContainerID = sh(
        returnStdout: true, 
        script: "docker ps -a --filter name=${containerName} --format json | jq -r .ID"
    ).trim()
    
    if (oldContainerID != '' && oldContainerID != ' ') {
        println 'Найден контейнер: ' + oldContainerID
        sh "docker stop ${oldContainerID}"
        sh "docker rm ${oldContainerID}"
    }
}
```

**Улучшения:**
- Проверка пустой строки с `&&` (не `||`)
- Добавление обработки ошибок
- Логирование операций для отладки

## Управление задачами

### Получение переменных окружения из последней успешной сборки

```groovy
def getLastSuccessful(jobName) {
    assert jobName != null
    return Jenkins.getInstance()
        .getItemByFullName(jobName)
        .lastSuccessfulBuild
        .getEnvVars()
}

// Использование
def envVars = getLastSuccessful('test-projects/DevOps/DevOpsBackendBuildReleaseFork')
envVars.each { key, value ->
    echo "${key} = ${value}"
}
```

**Случаи использования:**
- Отладка успешных сборок
- Сравнение конфигураций окружения
- Устранение различий в сборках

## Управление версиями

### Автоматическое версионирование из Git тегов

Генерация следующей версии на основе семантического версионирования (major.minor.patch):

```groovy
def nextVersionFromGit(scope) {
    def latestVersion = sh(
        returnStdout: true, 
        script: 'git describe --tags --abbrev=0 --match *.*.* 2> /dev/null || echo 0.0.0'
    ).trim()
    
    def (major, minor, patch) = latestVersion.tokenize('.').collect { it.toInteger() }
    def nextVersion
    
    switch (scope) {
        case 'major':
            nextVersion = "${major + 1}.0.0"
            break
        case 'minor':
            nextVersion = "${major}.${minor + 1}.0"
            break
        case 'patch':
            nextVersion = "${major}.${minor}.${patch + 1}"
            break
        default:
            error("Неверный scope: ${scope}. Используйте: major, minor или patch")
    }
    
    return nextVersion
}

// Использование в pipeline
pipeline {
    agent any
    parameters {
        choice(
            name: 'VERSION_SCOPE',
            choices: ['major', 'minor', 'patch'],
            description: 'Область инкремента версии'
        )
    }
    stages {
        stage('Version') {
            steps {
                script {
                    def nextVersion = nextVersionFromGit(params.VERSION_SCOPE)
                    echo "Следующая версия: ${nextVersion}"
                    env.VERSION = nextVersion
                }
            }
        }
    }
}
```

**Особенности:**
- Поддержка семантического версионирования
- Обработка отсутствующих тегов (по умолчанию 0.0.0)
- Соответствие только тегам формата x.y.z
- Инкремент версии на основе области действия

## Движок шаблонов

### Движок шаблонов Groovy (Non-CSP)

Генерация текста из шаблонов без ограничений Content Security Policy:

```groovy
String doTemplate(text, binding) {
    def engine = new groovy.text.StreamingTemplateEngine().createTemplate(text)
    def template = engine.make(binding)
    def result = template.toString()
    
    // Очистка
    engine = null
    template = null
    
    return result
}

// Использование
def templateText = '''
Привет ${name}!
Версия: ${version}
Дата: ${new Date()}
'''

def binding = [
    name: 'Jenkins',
    version: '2.400'
]

def result = doTemplate(templateText, binding)
echo result
```

**Случаи использования:**
- Генерация конфигурационных файлов
- Создание манифестов развертывания
- Форматирование уведомлений
- Динамическая генерация контента

**Синтаксис шаблона:**
- `${variable}` - Интерполяция переменных
- `<% code %>` - Выполнение кода Groovy
- `<%= expression %>` - Оценка выражения

## Рекомендации

1. **Обработка ошибок** - Всегда проверяйте входные данные и обрабатывайте ошибки
2. **Безопасность null** - Проверяйте значения null перед операциями
3. **Логирование** - Используйте `echo` или `println` для отладки
4. **Очистка ресурсов** - Устанавливайте переменные в null после использования в циклах
5. **Документация** - Добавляйте комментарии, объясняющие сложную логику
6. **Тестирование** - Сначала тестируйте скрипты в консоли скриптов Jenkins

## Связанные заметки

- [[Reset Build Number]] - Подробная инструкция по сбросу номеров сборок
- [[../Docker/Container Operations]]
- [[../Databases/ClickHouse]]
- [[Jenkinsfile Code Style]] - Стандарты написания и оформления Jenkinsfile

