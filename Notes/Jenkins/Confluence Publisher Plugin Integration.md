---
created: 2025-01-27
tags:
  - jenkins
  - confluence
  - integration
  - macros
  - automation
category: automation
---

# Интеграция Confluence с Jenkins через Confluence Publisher Plugin

## Введение

В данной статье описаны шаги, которые необходимо выполнить, чтобы создать пользовательские макросы в Confluence для интеграции с Jenkins посредством плагина [confluence-publisher-plugin](https://github.com/jenkinsci/confluence-publisher-plugin).

Данный плагин позволяет:
- Публиковать артефакты сборки из Jenkins пайплайнов в виде вложений на вики-страницах Confluence
- Редактировать содержимое вики-страниц Confluence

### Способы редактирования содержимого страниц

Доступны следующие способы редактирования:

1. **Вся страница** - Заменяется все содержимое страницы
   - *Никаких маркеров не требуется*

2. **Prepend** - Контент добавляется в самое начало страницы
   - *Никаких маркеров не требуется*

3. **Добавить** - Контент добавляется в самый конец страницы
   - *Никаких маркеров не требуется*

4. **Перед токеном** - Содержимое вставляется перед настраиваемым маркером
   - *Требуется один маркер*

5. **После токена** - Содержимое вставляется после настраиваемого маркерного токена
   - *Требуется один маркер*

6. **Между токенами** - Содержимое вставляется между двумя настраиваемыми маркерами начала/конца. Существующий контент между токенами заменяется
   - *Требуется маркер начала и конца*

### Преимущества интеграции

Подобная интеграция позволяет:
- Своевременно обновлять содержимое вики-страниц Confluence с информацией об актуальных версиях (release, dev, test, актуальные ссылки к артефактам и т.п.) разрабатываемого ПО
- Публиковать сформированные change-логи после выхода очередного релиза
- Поддерживать содержимое вики-страниц в актуальном состоянии
- Снизить количество времени, затрачиваемое разработкой на актуализацию той части информации, которая может быть обновлена автоматически

## Создание пользовательских макросов

Общая документация по созданию пользовательских макросов представлена по [данной ссылке](https://confluence.atlassian.com/doc/writing-user-macros-4485.html).

Ниже представлены примеры реализации пользовательских макросов `jenkins-marker` и `jenkins-between`, которые необходимы для интеграции.

> **Важно:**
> Представленные примеры актуальны для версии Confluence 7.13.2 (актуальная на момент 18.03.2024)

### Макрос jenkins-marker

Для создания пользовательского макроса **`jenkins-marker`** необходимо:

1. Перейти в раздел **Администрирование → Основные настройки → Пользовательские макросы**
2. Выбрать **"Создать пользовательский макрос"**
3. Заполнить необходимые поля:
   - **Имя макроса:** `jenkins-marker`
   - **Видимость:** "Сделать видимой для всех пользователей в браузере макросов"
   - **Название макроса:** `jenkins-marker`
   - **Определение макроса пользователя → Обработка тела макроса → Нет тела макроса**
   - **Шаблон:**

```velocity
## Macro title: Jenkins Token Marker
## Macro has a body: N
## Body processing: No macro body
## Output: None
##
## Developed by: System Administrator
## Date created: 03/18/2024
## Installed by: System Administrator

## Allows an "id" parameter in order to make multiple unique markers
## @param id
```

4. Нажать **"Сохранить"**

**Пример реализованного макроса jenkins-marker:**

![Image: jenkins-marker макрос](Attachments/Jenkins/confluence-macro-1.png)

### Макрос jenkins-between

Для создания пользовательского макроса **`jenkins-between`** необходимо:

1. Перейти в раздел **Администрирование → Основные настройки → Пользовательские макросы**
2. Выбрать **"Создать пользовательский макрос"**
3. Заполнить необходимые поля:
   - **Имя макроса:** `jenkins-between`
   - **Видимость:** "Сделать видимой для всех пользователей в браузере макросов"
   - **Название макроса:** `jenkins-between`
   - **Определение макроса пользователя → Обработка тела макроса → Представленный**
   - **Шаблон:**

```velocity
## Macro title: Jenkins Between-Tokens section
## Macro has a body: Y
## Body processing: Rendered
## Output: Piped input
##
## Developed by: System Administrator
## Date created: 03/18/2024
## Installed by: System Administrator

## Allows an "id" parameter in order to make multiple unique markers
## @param id
$body
```

4. Нажать **"Сохранить"**

**Пример реализованного макроса jenkins-between:**

![Image: jenkins-between макрос](Attachments/Jenkins/confluence-macro-2.png)

## Использование плагина

### Установка плагина

Необходимо установить плагин `confluence-publisher-plugin`. В настоящее время он отсутствует в списке плагинов, поскольку имеет неудовлетворенные Jenkins'ом сторонние зависимости.

**Варианты получения плагина:**

1. Скачать с зеркала: [https://mirrors.huaweicloud.com/jenkins/plugins/confluence-publisher/](https://mirrors.huaweicloud.com/jenkins/plugins/confluence-publisher/)
2. Если доступна внутренняя ссылка (например, из Confluence), использовать её

**Установка:**

1. Перейти в **Manage Jenkins → Plugins → Advanced settings → Deploy Plugin**
2. Загрузить `.hpi` файл с плагином
3. Нажать **Deploy**

### Подключение Confluence

Необходимо подключить Confluence к Jenkins.

**Шаги:**

1. Перейти в **Manage Jenkins → System → Confluence**
2. Указать URL адрес до Confluence
3. Указать учетные данные

> **Примечание:**
> Учетные данные хранятся в vault (`jenkins/creds/confluence-token`) - токен для доступа в Confluence под учетной записью `fabric_jenkins`.
> 
> У данной учетной записи ограниченный доступ к проектам, поэтому необходимо убедиться, что у неё есть доступ к проекту, с которым вы хотите работать через плагин.

**Пример конфигурации:**

![Image: Настройка Confluence в Jenkins](Attachments/Jenkins/confluence-macro-3.png)

### Работа с плагином

Общую информацию о работе плагина можно почитать в [README.md](https://github.com/jenkinsci/confluence-publisher-plugin) на Github.

Список возможных настроек можно посмотреть через `${JENKINS_URL}/pipeline-syntax → publishConfluence`.

**Важно:** Страницы Confluence, которые редактируются данным пайплайном, должны быть созданы и подготовлены заранее.

### Пример Jenkins Pipeline

Ниже представлен пример пайплайна, демонстрирующий основные возможности плагина:

```groovy
pipeline {
    agent any
    stages {
        stage('Test Confluence Full Replace') {
            steps {
                script {
                    publishConfluence editorList: [
                        confluenceWritePage(
                            confluenceText('''
                                <p>Автоматическое обновление через Jenkins. Полная замена <a href="${JOB_URL}">текст ${BUILD_NUMBER}</a></p>
                            ''')
                        )
                    ], 
                    pageName: 'FullReplace', 
                    parentId: 181315827, 
                    siteName: 'confluence.example.ru', 
                    spaceName: 'DevOps'
                    println('Full Replace Success')
                }
            }
        }
        
        stage('Test Confluence Before Token Replace') {
            steps {
                script {
                    publishConfluence editorList: [
                        confluenceAfterToken(
                            generator: confluenceText('''
                                <p>Автоматическое обновление через Jenkins. Вставка после маркера <a href="${JOB_URL}">текст ${BUILD_NUMBER}</a></p>
                            '''), 
                            markerToken: '<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="25e5f082-97cd-4514-969a-33f06d6cfb66"><ac:parameter ac:name="id">before</ac:parameter></ac:structured-macro></p>'
                        )
                    ], 
                    pageName: 'BeforeReplace', 
                    parentId: 181315827, 
                    siteName: 'confluence.example.ru', 
                    spaceName: 'DevOps'
                }
            }
        }
        
        stage('Test Confluence After Token Replace') {
            steps {
                script {
                    publishConfluence editorList: [
                        confluenceAfterToken(
                            generator: confluenceText('''
                                <p>Автоматическое обновление через Jenkins. Вставка после маркера <a href="${JOB_URL}">текст ${BUILD_NUMBER}</a></p>
                            '''), 
                            markerToken: '<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="8ae50f4d-2e6b-4531-b09b-9c3b7fdfac4c"><ac:parameter ac:name="id">after</ac:parameter></ac:structured-macro></p>'
                        )
                    ], 
                    pageName: 'AfterReplace', 
                    parentId: 181315827, 
                    siteName: 'confluence.example.ru', 
                    spaceName: 'DevOps'
                }
            }
        }
        
        stage('Test Confluence Between Token Replace') {
            steps {
                script {
                    publishConfluence editorList: [
                        confluenceBetweenTokens(
                            endMarkerToken: '<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="d416099d-9451-4c12-95e7-f6d0d59072bf"><ac:parameter ac:name="id">end</ac:parameter></ac:structured-macro></p>', 
                            generator: confluenceText('''
                                <p>Автоматическое обновление через Jenkins. Вставка между маркерами <a href="${JOB_URL}">текст ${BUILD_NUMBER}</a></p>
                            '''), 
                            startMarkerToken: '<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="1feeea24-93ba-42df-b8f4-ff025ef5d890"><ac:parameter ac:name="id">start</ac:parameter></ac:structured-macro></p>'
                        )
                    ], 
                    pageName: 'BetweenReplace', 
                    parentId: 181315827, 
                    siteName: 'confluence.example.ru', 
                    spaceName: 'DevOps'
                }
            }
        }
        
        stage('Test Confluence AttachArtifacts') {
            steps {
                script {
                    fileOperations([
                        fileCreateOperation(
                            fileContent: "Это артефакт сборки под номером ${BUILD_NUMBER}", 
                            fileName: 'test-artifact.txt'
                        )
                    ])
                    archiveArtifacts allowEmptyArchive: true, artifacts: "test-artifact.txt", fingerprint: true

                    publishConfluence attachArchivedArtifacts: true, 
                                    pageName: 'AttachArtifacts', 
                                    parentId: 181315827, 
                                    replaceAttachments: true, 
                                    siteName: 'confluence.example.ru', 
                                    spaceName: 'DevOps'
                }
            }
        }
        
        stage('Test Confluence Attach other files') {
            steps {
                script {
                    fileOperations([
                        fileCreateOperation(
                            fileContent: "Это файл, который не является артефактом сборки под номером ${BUILD_NUMBER}", 
                            fileName: 'test-other.txt'
                        )
                    ])

                    publishConfluence fileSet: 'test-other.txt', 
                                    pageName: 'AttachOther', 
                                    parentId: 181315827, 
                                    replaceAttachments: true, 
                                    siteName: 'confluence.example.ru', 
                                    spaceName: 'DevOps'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```

### Подробнее про markerToken | startMarkerToken | endMarkerToken

В отличие от примеров реализации, представленных в README.md, текущая версия Confluence не позволяет использовать пользовательские макросы напрямую (это связано с переходом от Wiki Markup к XHTML-Based Storage Format).

Поэтому в параметрах `markerToken | startMarkerToken | endMarkerToken` необходимо указывать "базовый формат хранения".

**Как получить формат хранения:**

1. Перейти на страницу Confluence, которую вы собираетесь редактировать
2. Перейти в параметры страницы (три точки в верхнем правом углу)
3. Перейти в **"Посмотреть формат хранения"** (может быть **"View Storage Format"**)
4. В открывшемся окне найти строку с добавленным вами макросом и скопировать её

**Пример:**

Для страницы со следующим форматом хранения:

```xml
<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="25e5f082-97cd-4514-969a-33f06d6cfb66"><ac:parameter ac:name="id">before</ac:parameter></ac:structured-macro></p>

<p>Автоматическое обновление через Jenkins. Вставка после маркера <a href="https://jenkins.example.local/job/test-projects/job/other/job/test-confluence/">текст 3</a></p>

<p>Автоматическое обновление через Jenkins. Вставка после маркера <a href="https://jenkins.example.local/job/test-projects/job/other/job/test-confluence/">текст 2</a></p>

<p>Автоматическое обновление через Jenkins. Вставка после маркера <a href="https://jenkins.example.local/job/test-projects/job/other/job/test-confluence/">текст 1</a></p>

<p>Это оригинальная версия, перед ней будут появляться новенькие</p>
```

В качестве `markerToken | startMarkerToken | endMarkerToken` будет выступать:

```xml
<p><ac:structured-macro ac:name="jenkins-marker" ac:schema-version="1" ac:macro-id="25e5f082-97cd-4514-969a-33f06d6cfb66"><ac:parameter ac:name="id">before</ac:parameter></ac:structured-macro></p>
```

## Основные параметры publishConfluence

### Обязательные параметры

- **pageName** - Название страницы в Confluence
- **siteName** - Название сайта Confluence (из конфигурации в Jenkins)
- **spaceName** - Название пространства (space) в Confluence
- **parentId** - ID родительской страницы (можно найти в URL страницы)

### Опциональные параметры

- **attachArchivedArtifacts** - Прикрепить архивированные артефакты сборки
- **fileSet** - Набор файлов для прикрепления (glob pattern)
- **replaceAttachments** - Заменить существующие вложения
- **editorList** - Список редакторов для изменения содержимого страницы:
  - `confluenceWritePage()` - Полная замена страницы
  - `confluencePrepend()` - Добавить в начало
  - `confluenceAppend()` - Добавить в конец
  - `confluenceBeforeToken()` - Вставить перед маркером
  - `confluenceAfterToken()` - Вставить после маркера
  - `confluenceBetweenTokens()` - Вставить между маркерами

## Best Practices

1. **Подготовка страниц заранее** - Создайте и подготовьте страницы Confluence перед использованием в пайплайне
2. **Уникальные маркеры** - Используйте параметр `id` для создания уникальных маркеров на одной странице
3. **Хранение формата** - Сохраните формат хранения маркеров в переменных пайплайна для удобства использования
4. **Тестирование** - Используйте тестовое пространство для проверки работы перед применением в продакшене
5. **Права доступа** - Убедитесь, что учетная запись Jenkins имеет необходимые права доступа к пространству
6. **Обработка ошибок** - Добавьте обработку ошибок при работе с Confluence API

## Troubleshooting

### Ошибка: "Page not found"

**Причина:** Неверное название страницы или отсутствие прав доступа

**Решение:**
- Проверьте точное название страницы (регистр важен)
- Убедитесь, что учетная запись Jenkins имеет доступ к пространству

### Ошибка: "Marker token not found"

**Причина:** Неверный формат маркера или маркер не существует на странице

**Решение:**
- Проверьте формат хранения маркера (должен быть скопирован из "View Storage Format")
- Убедитесь, что маркер действительно добавлен на страницу
- Проверьте, что `macro-id` совпадает с реальным ID макроса на странице

### Ошибка: "Authentication failed"

**Причина:** Неверные учетные данные или истекший токен

**Решение:**
- Проверьте настройки подключения в Manage Jenkins → System → Confluence
- Обновите токен доступа в vault

### Контент не обновляется

**Причина:** Неверный формат содержимого или конфликт версий

**Решение:**
- Проверьте формат HTML в `confluenceText()`
- Убедитесь, что версия плагина совместима с версией Confluence

## Связанные заметки

- [[Pipeline Scripts]] - Другие полезные скрипты для Jenkins pipelines
- [[Using Credentials in Active Choice Parameter]] - Использование секретов в Jenkins
- [[Nexus Component Download URL]] - Получение ссылок на компоненты из Nexus

