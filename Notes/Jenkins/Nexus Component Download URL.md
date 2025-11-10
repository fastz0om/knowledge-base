---
created: 2025-01-27
tags:
  - jenkins
  - nexus
  - groovy
  - api
  - artifact
category: automation
---

# Получение ссылок на скачивание компонентов из Nexus

## Обзор

Функция для получения полной ссылки на скачивание компонентов из Nexus репозиториев (apt, yum, raw, maven2) через Search API.

## Базовая функция

### Получение URL для скачивания

```groovy
def getDownloadUrlByComponent(baseUrl, repo='', componentName='', componentVersion='') {
    def encodedVersion = URLEncoder.encode(componentVersion, "UTF-8")
    def url = "${baseUrl}/service/rest/v1/search?repository=${repo}&name=${componentName}&version=${encodedVersion}"
    def downloadUrls = []
    def continuationToken = null
    def tempUrl = url
     
    while (true) {
        def conn = new URL(tempUrl).openConnection()
        conn.setRequestMethod("GET")
        Map parsedJson = new groovy.json.JsonSlurper().parseText(conn.content.text)
 
        for (item in parsedJson.items){
            downloadUrls.add(item.assets[0].downloadUrl)
        }
         
        continuationToken = parsedJson.continuationToken
        if (continuationToken != null) {
            tempUrl = url + "&continuationToken=" + continuationToken
        }
        else {
            break;
        }
    }
 
    return downloadUrls
}
```

### Параметры функции

- **baseUrl** - Базовый URL Nexus сервера (например: `https://nexus.example.local`)
- **repo** - Имя репозитория (опционально)
- **componentName** - Имя компонента (опционально)
- **componentVersion** - Версия компонента (опционально)

### Примеры использования

#### Поиск по имени и версии

```groovy
def base = "https://nexus.example.local"
def repository = 'repo-standalone-develop'
def packageName = 'package-name'
def packageVersion = '0.3+dev-5.15.0-102.6'

// Найдет ссылку в указанном репозитории, по заданному имени и версии
downloadUrls = getDownloadUrlByComponent(base, repository, packageName, packageVersion)

// Вывод результатов
downloadUrls.each { url ->
    println "Download URL: ${url}"
}
```

#### Поиск по имени (все версии)

```groovy
def base = "https://nexus.example.local"
def repository = 'repo-standalone-develop'
def packageName = 'package-name'

// Найдет ссылки в указанном репозитории по заданному имени (все версии)
downloadUrls = getDownloadUrlByComponent(base, repository, packageName, '')

// Вывод всех найденных ссылок
downloadUrls.each { url ->
    println "Download URL: ${url}"
}
```

#### Поиск только в репозитории

```groovy
def base = "https://nexus.example.local"
def repository = 'maven-releases'

// Найдет все компоненты в указанном репозитории
downloadUrls = getDownloadUrlByComponent(base, repository, '', '')
```

## Расширенная функция

### Получение детальной информации о компоненте

```groovy
def getNexusArtifactInfo(baseUrl, repo='', componentName='', componentVersion='') {
    def encodedVersion = URLEncoder.encode(componentVersion, "UTF-8")
    def url = "${baseUrl}/service/rest/v1/search?repository=${repo}&name=${componentName}&version=${encodedVersion}"
    def componentInfo = [:]
    def continuationToken = null
    def tempUrl = url
     
    while (true) {
        def conn = new URL(tempUrl).openConnection()
        conn.setRequestMethod("GET")
        Map parsedJson = new groovy.json.JsonSlurper().parseText(conn.content.text)
 
        for (item in parsedJson.items){
            componentInfo += [('url') : (item.assets[0].downloadUrl)]
            componentInfo += [('md5') : (item.assets[0].checksum.md5)]
            componentInfo += [('sha1') : (item.assets[0].checksum.sha1)]
            componentInfo += [("name") : (item.assets[0].maven2.artifactId)]
            componentInfo += [("extension") : (item.assets[0].maven2.extension)]
            componentInfo += [("version") : (item.assets[0].maven2.version)]
            componentInfo += [("size") : (item.assets[0].fileSize)]
        }
         
        continuationToken = parsedJson.continuationToken
        if (continuationToken != null) {
            tempUrl = url + "&continuationToken=" + continuationToken
        }
        else {
            break;
        }
    }
    
    println 'Information about ' + componentName + ': ' + componentInfo
    return componentInfo
}
```

### Использование расширенной функции

```groovy
def base = "https://nexus.example.local"
def repository = 'maven-releases'
def packageName = 'my-artifact'
def packageVersion = '1.0.0'

// Получить детальную информацию о компоненте
artifactInfo = getNexusArtifactInfo(base, repository, packageName, packageVersion)

// Доступ к информации
println "Download URL: ${artifactInfo.url}"
println "MD5: ${artifactInfo.md5}"
println "SHA1: ${artifactInfo.sha1}"
println "Size: ${artifactInfo.size} bytes"
```

## Информация в расширенной функции

Расширенная функция возвращает:

- **url** - URL для скачивания компонента
- **md5** - MD5 хеш файла
- **sha1** - SHA1 хеш файла
- **name** - Имя артефакта (для Maven)
- **extension** - Расширение файла (для Maven)
- **version** - Версия компонента (для Maven)
- **size** - Размер файла в байтах

## Особенности

### Обработка пагинации

Функция автоматически обрабатывает пагинацию результатов через `continuationToken`, получая все доступные компоненты.

### Кодирование версии

Версия автоматически кодируется в UTF-8 для корректной работы с URL.

### Обработка ошибок

Для production использования рекомендуется добавить обработку ошибок:

```groovy
def getDownloadUrlByComponent(baseUrl, repo='', componentName='', componentVersion='') {
    try {
        def encodedVersion = URLEncoder.encode(componentVersion, "UTF-8")
        // ... остальной код
        
        return downloadUrls
    } catch (Exception e) {
        println "Error getting download URL: ${e.message}"
        return []
    }
}
```

## Использование в Pipeline

### Пример в Declarative Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Get Nexus URL') {
            steps {
                script {
                    def base = "https://nexus.example.local"
                    def repository = 'repo-standalone-develop'
                    def packageName = 'package-name'
                    def packageVersion = '0.3+dev-5.15.0-102.6'
                    
                    downloadUrls = getDownloadUrlByComponent(base, repository, packageName, packageVersion)
                    
                    if (downloadUrls.size() > 0) {
                        env.DOWNLOAD_URL = downloadUrls[0]
                        echo "Download URL: ${env.DOWNLOAD_URL}"
                    } else {
                        error("Component not found")
                    }
                }
            }
        }
    }
}
```

## Расширение функциональности

Функцию можно расширять:

1. **Фильтрация результатов** - добавить условия на добавление в список
2. **Дополнительные поля** - извлекать другую информацию из Search API
3. **Сортировка** - сортировать результаты по версии, дате и т.д.
4. **Кэширование** - кэшировать результаты для повторных запросов

### Пример с фильтрацией

```groovy
def getLatestVersionUrl(baseUrl, repo='', componentName='') {
    def urls = getDownloadUrlByComponent(baseUrl, repo, componentName, '')
    // Вернуть только последнюю версию (требует дополнительной логики парсинга версий)
    return urls.size() > 0 ? urls[0] : null
}
```

## Best Practices

1. **Кэшируйте результаты** - для частых запросов к одному и тому же компоненту
2. **Добавьте таймауты** - для предотвращения зависания при проблемах с сетью
3. **Логируйте запросы** - для отладки и мониторинга
4. **Обрабатывайте ошибки** - добавляйте try-catch блоки
5. **Валидируйте параметры** - проверяйте корректность входных данных

## Связанные заметки

- [[Pipeline Scripts]] - Работа с Jenkins pipelines
- [[../Linux/System Administration]] - Системное администрирование

