---
created: 2025-01-27
tags:
  - jenkins
  - jenkinsfile
  - coding-standards
  - groovy
  - best-practices
category: documentation
---

# Стандарты написания Jenkinsfile

## Общие положения

### Цель документа

Обеспечить единообразие в написании и поддержке Jenkinsfile, повысить читаемость, упростить сопровождение и расширение пайплайнов.

### Типы пайплайнов

Основной акцент делается на **Declarative Pipeline**. При этом допускается использование **Scripted Pipeline**, но с соблюдением общих принципов стиля.

## Описание стиля кода

### Расположение и именование Jenkinsfile

#### Расположение

Рекомендуется, чтобы путь к Jenkinsfile в репозитории соответствовал следующей схеме:

```
${PROJECT_NAME}/${PIPELINE_TYPE}/${PIPELINE_NAME}
```

где:
- **PROJECT_NAME** - название проекта
- **PIPELINE_TYPE** - тип пайплайна. Т.е. к какой логической группе можно его отнести. Например, сборка бэкенда, сборка фронтенда, сборка docker-образов и т.п.
- **PIPELINE_NAME** - название пайплайна

**Примеры:**

```bash
DevOps/Backend/DevOpsBackendAutoBuild.groovy
Addvisor/Backend/abookDaemonBuild.groovy
Fabric/Infrastructure/updateProxmoxLoadBalancerConfigFile.groovy
```

#### Наименование

Используйте **lowerCamelCase** для именования пайплайна.

Файл должен иметь расширение `.groovy`.

### Структура и форматирование кода

#### Расположение глобальных переменных и функций

##### Глобальные переменные

Все глобальные переменные следует определять в начале файла, до блока `pipeline {}`. Это обеспечивает их доступность для всех этапов и упрощает понимание, какие переменные используются по всему файлу.

**Пример:**

```groovy
// Глобальные переменные
def globalVar1 = "значение1"
def globalVar2 = 42

pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                script {
                    println "Используем глобальную переменную: ${globalVar1}"
                }
            }
        }
    }
}
```

##### Функции и методы

Все функции и методы, которые используются в пайплайне, необходимо определять в конце файла, после блока `pipeline {}`.

Это помогает отделить логику пайплайна от вспомогательного кода и делает файл более структурированным.

**Пример:**

```groovy
pipeline {
    agent any
    stages {
        stage("Deploy") {
            steps {
                script {
                    println "Результат обработки: ${processData(globalVar2)}"
                }
            }
        }
    }
}

// Функции и методы
def processData(value) {
    return "Обработано значение: ${value}"
}
```

#### Отступы и максимальная длина строки

##### Отступы

Все блоки кода должны форматироваться с отступами по **4 пробела**.

**Пример:**

```groovy
pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                // код
            }
        }
    }
}
```

##### Максимальная длина строки

Рекомендуется, чтобы длина строки не превышала **120 символов**. Если строка получается длинной, используйте переносы строк с сохранением логики и читаемости.

#### Фигурные скобки

- Открывающая фигурная скобка `{` должна располагаться в конце строки, после определения блока (без переноса на новую строку)
- Закрывающая фигурная скобка `}` должна выровниваться с началом блока, то есть быть на одном уровне отступов с соответствующим ключевым словом

**Пример:**

```groovy
stage("Test") {
    // содержимое блока
}
```

### Именование и стилистика

#### Переменные

Используйте **lowerCamelCase** для именования переменных.

**Пример:**

```groovy
def buildStatus = "SUCCESS"
```

#### Функции и методы

- Название методов должно быть в **lowerCamelCase**
- Методы должны начинаться с глагола, описывающего их действие

**Пример:**

```groovy
def loadData() {}
def processRequest() {}
def fetchUserDetails() {}
```

#### Названия этапов (stage)

##### Обрамление в двойные кавычки

Названия этапов должны быть заключены в двойные кавычки.

##### Стиль названия

Названия этапов должны начинаться с заглавной буквы и отражать суть этапа.

**Пример:**

```groovy
stage("Build") {
    // шаги
}
stage("Test") {
    // шаги
}
```

### Работа с параметрами

#### Определение параметров

Все параметризованные параметры должны определяться внутри блока `pipeline{...}` в специальном блоке `parameters{...}`.

Избегайте вынесения параметров через вызов `properties` вне блока `pipeline{...}`.

> **Исключение для Active Choice плагина:**
> Если используются параметры от Active Choice плагина, допускается комбинированный подход.
> 
> Например, параметры можно определить как в блоке parameters, так и через properties, если того требует логика работы плагина.

**Примеры:**

**Стандартный случай:**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Целевое окружение для деплоя')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Запускать тесты после сборки')
    }
    stages {
        stage("Build") {
            steps {
                echo "Build"
            }
        }
        stage("Test") {
            steps {
                echo "Test"
            }
        }
    }
}
```

**Пример использования комбинированного подхода для Active Choice параметров:**

```groovy
properties([
    parameters ([
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_CHECKBOX',
            description: 'Укажите перечень сервисов, для которых необходимо развернуть контейнеры',
            filterLength: 1,
            filterable: false,
            name: 'Services',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    oldScript: '',
                    sandbox: false,
                    script: 'return ["error"]'
                ],
                script: [
                    classpath: [],
                    oldScript: '',
                    sandbox: false,
                    script: 'return ["frontend-client", "frontend-ui-showcase"]'
                ]
            ]
        ]
    ])
])

pipeline {
    agent any
    parameters {
        gitParameter branchFilter: 'origin/((develop)|(master))', 
                    defaultValue: 'develop', 
                    name: 'BRANCH', 
                    type: 'PT_BRANCH', 
                    quickFilterEnabled: true, 
                    description: 'Выберите ветку, для которой необходимо собрать контейнеры', 
                    useRepository: '.*product-release-tracker.git'
    }
    stages {
        stage("Build") {
            steps {
                echo "Build"
            }
        }
        stage("Test") {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                echo "Test"
            }
        }
    }
}
```

#### Обращения к параметрам

При обращении к параметрам в пайплайне всегда используйте конструкцию `params.PARAM_NAME`

где:
- **PARAM_NAME** - название параметра

> **Важно:**
> Не рекомендуется обращаться к параметрам напрямую через `PARAM_NAME` или `env.PARAM_NAME`, чтобы избежать неоднозначностей и обеспечить корректное получение значений.

**Пример:**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Целевое окружение для деплоя')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Запускать тесты после сборки')
    }
    stages {
        stage("Build") {
            steps {
                echo "Окружение: ${params.ENVIRONMENT}"
            }
        }
        stage("Test") {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                echo "Запуск тестов"
            }
        }
    }
}
```

### Работа с Post Actions

Подробнее с Post Actions можно ознакомиться в [официальной документации](https://www.jenkins.io/doc/book/pipeline/syntax/#post).

#### Использование cleanup

Если по окончанию работы пайплайна необходимо произвести очистку рабочего пространства с помощью функции cleanup, то рекомендуется использовать для этого условие `cleanup{...}` в блоке `post{...}`

**Пример:**

```groovy
pipeline {
    agent {
        kubernetes {
            yaml podTemplate
            defaultContainer 'utils'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
    }
    stages {
        stage("Test") {
            steps {
                script {
                    println "Start test"
                }
            }
        }
    }
    post {
        cleanup {
            cleanWs(deleteDirs: true, disableDeferredWipeout: true)
        }
    }
}
```

### Обработка ошибок и логирование

#### Обработка ошибок

Используйте конструкции `catchError` или `try/catch` для перехвата ошибок.

**Примеры:**

**Использование catchError:**

```groovy
stage("Deploy") {
    steps {
        catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
            sh 'deploy.sh'
        }
    }
}
```

**Использование try/catch:**

```groovy
try {
    sh 'executeTask.sh'
} catch (Exception e) {
    println "Ошибка выполнения: ${e.getMessage()}"
    // Дополнительные действия при ошибке
}
```

#### Логирование

Для вывода логов используйте вызов `println`.

Если в логе необходимо отразить содержимое переменной, следует использовать интерполяцию строки через конструкцию `${...}`. Это позволяет сделать код более читаемым и избежать ошибок при конкатенации.

**Пример (правильно):**

```groovy
println "Current result: ${result}"
```

**Избегайте конкатенации строк (неправильно):**

```groovy
println "Current result: " + result
```

### Модульность и использование shared libraries

#### Сложные шаги

Если функционал шага становится слишком сложным, избегайте длинных однострочных команд в блоке `sh`. Необходимо выделять логику в отдельные методы или использовать возможности Groovy для реализации функционала.

#### Shared libraries

Вынесите повторяющийся или сложный код в **Jenkins Shared Library**. Это позволит централизованно поддерживать общий код и упростит тестирование.

Подробнее о существующих библиотеках можно найти в соответствующей документации.

### Дополнительные рекомендации

#### Комментарии и документация

##### Комментарии

Комментируйте нестандартные решения, сложную логику или важные участки кода.

**Пример:**

```groovy
// Выполняем предварительную проверку переменных окружения
```

##### Документация

Для документирования пайплайна следует оформлять статьи в Confluence (или в соответствующей системе документации).

Общий шаблон документации должен включать:
- Описание назначения пайплайна
- Требования и зависимости
- Параметры пайплайна
- Этапы выполнения
- Примеры использования
- Troubleshooting

После создания документации необходимо связать документацию с исходным кодом пайплайна.

Для этого можно использовать [Sidebar Link Plugin](https://plugins.jenkins.io/sidebar-link/).

#### Стандарты оформления кода

Используйте пустые строки для разделения логических блоков кода.

## Резюме основных правил

### Структура файла

1. Глобальные переменные → в начале файла
2. Блок `pipeline {}` → основная логика
3. Функции и методы → в конце файла

### Именование

- Файлы и пайплайны: **lowerCamelCase** с расширением `.groovy`
- Переменные: **lowerCamelCase**
- Функции: **lowerCamelCase** с глаголом в начале
- Этапы: с заглавной буквы, в двойных кавычках

### Форматирование

- Отступы: **4 пробела**
- Максимальная длина строки: **120 символов**
- Фигурные скобки: открывающая `{` в конце строки, закрывающая `}` на уровне ключевого слова

### Параметры

- Определение: в блоке `parameters {}` внутри `pipeline {}`
- Использование: всегда через `params.PARAM_NAME`
- Исключение: Active Choice параметры могут быть определены через `properties`

### Post Actions

- Использовать `cleanup {}` в блоке `post {}` для очистки рабочего пространства

### Обработка ошибок

- Использовать `catchError` или `try/catch`
- Использовать `println` для логирования
- Использовать интерполяцию строк `${...}` вместо конкатенации

### Модульность

- Выносить сложную логику в методы
- Использовать Jenkins Shared Library для повторяющегося кода

## Связанные заметки

- [[Pipeline Scripts]] - Полезные Groovy скрипты для Jenkins pipelines
- [[Using Credentials in Active Choice Parameter]] - Использование секретов в Active Choice Parameter
- [[Active Choice Parameter Logging]] - Настройка логгирования Active Choice Parameter
- [[Confluence Publisher Plugin Integration]] - Интеграция с Confluence для документации
- [[Shared Library Code Style]] - Стандарты разработки Jenkins Shared Library

