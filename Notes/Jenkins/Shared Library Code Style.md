---
created: 2025-01-27
tags:
  - jenkins
  - shared-library
  - groovy
  - coding-standards
  - best-practices
category: documentation
---

# Jenkins Shared Library: Code Style и Best Practices

## Описание

### Общая информация

Jenkins Shared Library предоставляет унифицированный набор операций и функций, позволяющих стандартизировать и упростить реализацию CI/CD пайплайнов.

Библиотека включает в себя набор инструментов для автоматизации сборки, тестирования, деплоя и мониторинга.

Использование библиотек повышает стабильность и прозрачность процессов, способствуя эффективной реализации DevOps-практик.

### Существующие библиотеки

Ссылки на документацию по существующим внутренним библиотекам можно найти в соответствующей документации Confluence.

## Полезные ссылки

- **Официальная документация по Jenkins Shared Library:** [Extending with Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- **Статья на Habr:** [Разработка и тестирование Jenkins Shared Library](https://habr.com/ru/companies/slurm/articles/674506/)
- **Статья на Habr:** [Jenkins Pipeline Shared Libraries](https://habr.com/ru/articles/338032/)
- **Статья на Dev.To:** [Introduction To Jenkins Shared Libraries](https://dev.to/cwprogram/introduction-to-jenkins-shared-libraries-3p67)

## Code Style

Этот раздел описывает правила кодирования для разработки Jenkins Shared Library, охватывая стиль кода как для внутренних классов (`src/`), так и для глобальных функций (`vars/`).

### Общие рекомендации

#### Отступы и форматирование

- Использовать отступ в **4 пробела**
- Длина строки не регламентирована, но рекомендуется разбивать длинные выражения для улучшения читаемости
- Открывающая фигурная скобка `{` располагается на той же строке, что и оператор, к которому она относится
- Закрывающая `}` должна находиться на отдельной строке, выровненной с началом конструкции

**Примеры:**

❌ **Плохой стиль:**

```groovy
def doSomething()
{
    println "Неправильное расположение скобок"
}
```

✅ **Хороший стиль:**

```groovy
def doSomething() {
    println "Правильное расположение скобок"
}
```

### Именование

#### Классы

- Название класса должно быть в **UpperCamelCase**
- Название класса должно заканчиваться именем пакета, в котором он расположен

> **Исключение:** Если в пакете уже содержится название организации или проекта, можно опустить его дублирование.

**Примеры:**

```groovy
package src.mycompany.clients

class NexusApiClient {
    // Логика класса
}
```

```groovy
package src.mycompany.build

class DockerImageBuilder {
    // Логика класса
}
```

#### Методы и переменные

- Название методов и переменных должно быть в **camelCase**

**Пример:**

```groovy
def firstName
String loadData
Map processRequest
```

- Методы должны начинаться с глагола, описывающего их действие
- Константы должны быть написаны в **SCREAMING_SNAKE_CASE** (Пример: `DEFAULT_TIMEOUT`, `MAX_RETRIES`)

**Примеры:**

❌ **Плохой стиль:**

```groovy
def Load_Data() {} // Использование snake_case в Groovy не принято
def Processrequest() {} // Странный стиль именования
def x() {} // Непонятное имя метода
```

✅ **Хороший стиль:**

```groovy
def loadData() {}
def processRequest() {}
def fetchUserDetails() {}
```

### Документирование

#### Общие правила

- Обязательны GroovyDoc-комментарии к глобальным функциям (`vars/`) и публичным методам классов (`src/`)
- Важно указывать назначение метода, параметры и возвращаемое значение
- Комментарии должны быть на кириллице

#### Оформление GroovyDoc

Используйте стандартные теги `@param`, `@return` и `@throws` (если применимо).

**Пример для метода в src/:**

```groovy
/**
 * Выполняет HTTP GET запрос по указанному URL с дополнительными HTTP-заголовками.
 *
 * @param url URL запроса.
 * @param headers Map заголовков (по умолчанию пустая).
 * @return Строка с телом ответа.
 * @throws IOException если HTTP-код ответа не равен 200.
 */
String get(String url, Map<String, String> headers = [:]) {
    def connection = new URL(url).openConnection() as HttpURLConnection
    connection.requestMethod = "GET"
    headers.each { key, value ->
        connection.setRequestProperty(key, value)
    }
    if (connection.responseCode != 200) {
        throw new IOException("HTTP GET request for '${url}' ended with the code ${connection.responseCode}")
    }
    return connection.inputStream.text
}
```

**Пример для глобальной функции в vars/:**

```groovy
/**
 * Добавляет элементы в список, если:
 *  - элемент не равен null,
 *  - элемент не пустая строка,
 *  - список не содержит этот элемент.
 *
 * @param elems Список элементов для добавления
 * @param list Список, куда производится добавление
 * @return Обновлённый список
 */
def call(List elems, List list) {
    return ListUtils.addElemsToListIfNotExist(elems, list)
}
```

### Структура библиотеки

#### Расположение файлов

##### Глобальные функции (vars/)

Файл должен называться так же, как и вызываемая в нём функция.

Например, `vars/deployApp.groovy` должен содержать `def call()`.

##### Классы и утилиты (src/)

Используйте пакеты, соответствующие структуре библиотеки.

Например, `src/ru/company/fabric/jenkins/common/components/utils/JsonUtils.groovy`.

#### Пример структуры

Далее представлен пример корректной структуры каталогов:

```bash
── src
│   └── ru
│       └── company
│           └── fabric
│               └── jenkins
│                   └── common
│                       └── components
│                           ├── clients
│                           │   ├── DependencyTrackApiClient.groovy
│                           │   ├── GitlabApiClient.groovy
│                           ├── formatters
│                           │   └── TemplateFormatter.groovy
│                           ├── notifications
│                           │   └── EmailNotification.groovy
│                           ├── transformers
│                           │   ├── DependencyTrackTransformer.groovy
│                           ├── utils
│                           │   ├── ChangeSetUtils.groovy
│                           └── wrappers
│                               └── HttpResponse.groovy
├── test
│   ├── ru
│   │   └── company
│   │       └── fabric
│   │           └── jenkins
│   │               └── common
│   │                   └── components
│   │                       ├── BasePipelineTest.groovy
│   │                       ├── clients
│   │                       │   ├── JiraApiClientSpec.groovy
│   │                       │   └── NexusApiClientSpec.groovy
│   │                       ├── formatters
│   │                       │   ├── BannerFormatterSpec.groovy
│   │                       │   └── TemplateFormatterTest.groovy
│   │                       ├── helpers
│   │                       │   └── FakeHttpClient.groovy
│   │                       ├── notifications
│   │                       │   └── EmailNotificationSpec.groovy
│   │                       ├── transformers
│   │                       │   └── NexusTransformerSpec.groovy
│   │                       └── utils
│   │                           ├── PipelineUtilsTest.groovy
│   │                           └── SvaceUtilsSpec.groovy
│   └── vars
│       ├── DoTemplateFunctionTest.groovy
│       └── UpdateJiraIssueCustomFiledFunctionSpec.groovy
└── vars
    ├── addElemsToListIfNotExist.groovy
    ├── addElemsToListIfNotExist.txt
    └── updateJiraIssueCustomField.txt
```

### Best Practices

#### Используйте именованные параметры через Map

**Пример:**

```groovy
def call(Map params) {
    def appName = params.get('appName', 'default-app')
    def env = params.get('environment', 'staging')
    echo "Deploying ${appName} to ${env}"
}
```

#### Разделяйте бизнес-логику и пайплайн-код

- Логика обработки данных должна находиться в `src/`
- В `vars/` остаются только вызовы Jenkins-специфичных шагов (`echo`, `sh`, `bat`)

#### Соблюдайте единообразие в логах

Выводите сообщения с префиксом `[INFO]`, `[ERROR]`, `[DEBUG]` и т.д.

#### Избегайте глобального состояния

Не используйте статические переменные без необходимости.

#### Документируйте сложные функции и классы

Чем сложнее код, тем более детально его стоит документировать.

## Известные ограничения

В данном разделе представлен набор обнаруженных ограничений в работе Jenkins Shared Library.

### Выполнение кода на Jenkins Master по умолчанию

#### Описание

Если функция из Shared Library не обёрнута в блок `node` (или не является вызовом шагов, таких как `sh`, `bat`, `echo`), то её код выполняется на контроллере (Jenkins Master).

#### Рекомендации

Всегда оборачивайте код, который зависит от рабочего пространства или должен выполняться на агенте, в блок `node`.

### Передача параметров с дефолтными значениями

#### Описание

При использовании нескольких последовательных параметров с значениями по умолчанию невозможно пропустить один из них и задать значение следующему.

**Пример:**

Для функции `compareFileData(Comparator comparator, file1="one.txt", file2="two.txt"){...}`

Вызов вида `compareFileData(file2="my-file.txt", myComparator)` недопустим.

> **Примечание:**
> Это ограничение является особенностью языка Groovy, а не специфичным для Shared Library.

#### Рекомендации

Для повышения гибкости используйте именованные параметры, передаваемые через Map.

**Пример:**

```groovy
// Реализация функции с именованными параметрами
def compareFileData(Map params = [:]) {
    // Получение параметров из Map
    Comparator comparator = params.comparator
    String file1 = params.get('file1', 'one.txt')
    String file2 = params.get('file2', 'two.txt')
    // вызов основной функции
    compareFileData(comparator, file1, file2)
}

// Основная реализация
def compareFileData(Comparator comparator, file1="one.txt", file2="two.txt") {
    // реализация функции
    // ...
}

// Примеры вызова
// Вызов с именованными параметрами
compareFileData(comparator: myComparator, file2: "my-file.txt")

// Вызов основной функции
compareFileData(myComparator)

// Вызов основной функции с определением всех параметров
compareFileData(myComparator, fileOnePath, fileTwoPath)
```

### Работа с файловой системой

#### Описание

Использование `new File` из [java.io](https://docs.oracle.com/javase/8/docs/api/java/io/File.html) приводит к поиску файла на Jenkins Master, даже если код запускается на агенте.

#### Рекомендации

Применяйте объект `new FilePath` из [hudson](https://javadoc.jenkins.io/hudson/FilePath.html) для работы с файлами, так как он корректно обрабатывает локальное расположение файлов на агенте.

### Глобальное состояние и многопоточность

#### Описание

Shared Library загружается один раз для каждого запуска pipeline, что может привести к накоплению глобального состояния (например, статических переменных). При параллельном выполнении веток это может вызвать гонки или непредсказуемое поведение.

#### Рекомендации

Избегайте использования изменяемых глобальных переменных внутри библиотеки. Если требуется хранить состояние, ограничивайте область его видимости или используйте локальные переменные.

### Ограничения Sandbox и безопасность

#### Описание

При работе в режиме Groovy Sandbox некоторые методы и операции могут быть запрещены. Это связано с механизмом безопасности, направленным на предотвращение выполнения потенциально опасного кода.

#### Рекомендации

При возникновении ошибок, связанных с ограничениями Sandbox, одобрите необходимые методы через Script Approval, либо добавляйте библиотеку как Global Trusted Library.

### Ограничения документации

#### Описание

Jenkins по умолчанию не предоставляет возможностей для расширения глобального раздела pipeline-syntax информацией об использовании глобальных функций Jenkins Shared Library.

Это связано с особенностью работы генераторов документации.

#### Рекомендации

Вместо этого для просмотра перечня доступных глобальных функций есть возможность использовать раздел **Global Variable Reference** в разделе пайплайна, который хотя бы единожды импортировал библиотеку.

> **Примечание 1:**
> Информация и описание глобальных функций будет доступна лишь в том пайплайне, который хотя бы 1 раз успешно импортировал библиотеку.

> **Примечание 2:**
> Для того, чтобы появилось описание функции необходимо самостоятельно подготовить `.txt` файл, который должен иметь то же название, что и глобальная функция и лежать в директории `vars/`.
> 
> Файл должен содержать описание в виде HTML-разметки.

### Ограничения логирования во внутренних классах

#### Описание

Код внутри классов из каталога `src/` не исполняется в контексте пайплайна, где доступны шаги (например, `echo`). Таким образом, вызовы `println` в таких классах не будут перенаправлены в консоль Jenkins.

#### Рекомендации

Чтобы обойти это ограничение, можно реализовать механизм инъекции логгера. Например, внутри глобальной функции (определённой в `vars/`) можно создать closure для логирования, который использует `echo`. Затем эту функцию можно передать в конструктор или метод класса из `src/`, чтобы внутри него использовать именно её для вывода сообщений.

**Пример:**

```groovy
// Класс в каталоге src/
class MyInternalClass {
    def logger

    MyInternalClass(logger) {
        this.logger = logger
    }

    def doSomething() {
        logger("Отладочное сообщение: начало выполнения doSomething")
        // Логика работы метода...
        logger("Отладочное сообщение: завершение выполнения doSomething")
    }
}

// В глобальной функции (vars/yourGlobalFunction.groovy):
def call() {
    // Определяем логгер, который использует echo
    def myLogger = { msg -> echo msg }
    def instance = new MyInternalClass(myLogger)
    instance.doSomething()
}
```

Таким образом, даже при вызове только глобальной функции, отладочная информация будет выводиться через `echo` и попадать в лог пайплайна.

### Ограничения использования параметров из Active Choice

#### Описание

Функции из Jenkins Shared Library напрямую внутри блоков `script` плагина Active Choice использовать нельзя.

Это связано с тем, что блоки Active Choice исполняются в другом контексте (обычно в sandboxed Groovy) и не поддерживают директивы, предназначенные для загрузки shared library, такие как `@Library('my-library') _`.

#### Рекомендации

Если вам необходимо использовать функции из shared library, можно вычислить требуемые значения в основном пайплайне (где библиотека уже доступна) и затем передать их в параметры.

> **Ограничение:**
> Не подойдет для случаев, когда необходимо формировать динамические списки значений.

## Резюме основных правил

### Структура и форматирование

- Отступы: **4 пробела**
- Фигурные скобки: `{` в конце строки, `}` на новой строке, выровнена с началом блока

### Именование

- Классы: **UpperCamelCase**, название должно заканчиваться именем пакета
- Методы и переменные: **camelCase**
- Константы: **SCREAMING_SNAKE_CASE**
- Методы должны начинаться с глагола

### Документирование

- Обязательны GroovyDoc-комментарии для глобальных функций и публичных методов
- Использовать теги `@param`, `@return`, `@throws`
- Комментарии на кириллице

### Best Practices

- Использовать именованные параметры через Map
- Разделять бизнес-логику (`src/`) и пайплайн-код (`vars/`)
- Единообразие в логах с префиксами `[INFO]`, `[ERROR]`, `[DEBUG]`
- Избегать глобального состояния

### Важные ограничения

- Обёртывать код, зависящий от workspace, в блок `node`
- Использовать `FilePath` вместо `File` для работы с файлами
- Избегать статических переменных в многопоточных сценариях
- Использовать инъекцию логгера для классов в `src/`
- Не использовать функции shared library напрямую в Active Choice

## Связанные заметки

- [[Jenkinsfile Code Style]] - Стандарты написания Jenkinsfile
- [[Pipeline Scripts]] - Полезные Groovy скрипты для Jenkins
- [[Using Credentials in Active Choice Parameter]] - Работа с секретами в Active Choice Parameter

