---
created: 2025-01-27
tags: [docker, containers, jenkins, how-to]
category: infrastructure
---

# Операции с Docker

## Обзор

Типовые операции Docker для управления контейнерами и интеграции с Jenkins.

## Управление контейнерами

### Получение ID контейнера по имени

Полезно для скриптов автоматизации, которым нужно останавливать/удалять контейнеры:

```bash
docker ps -a --filter name=dreamy_mendel --format json | jq -r .ID | xargs docker stop
```

**Объяснение:**
1. `--filter name=<pattern>` - Фильтровать контейнеры по имени
2. `--format json` - Вывод в формате JSON
3. `jq -r .ID` - Извлечь поле ID с помощью jq
4. `xargs docker stop` - Передать ID команде docker stop

**Альтернатива без jq:**
```bash
docker ps -a --filter name=dreamy_mendel --format "{{.ID}}" | xargs docker stop
```

## Среда разработки

### Запуск сборки Maven в контейнере

Полезно для сборок Jenkins, которым нужна конкретная версия Maven:

```bash
docker run -it --rm \
  -v /home/fastz0om/repos/external/confluence-publisher-plugin:/opt/ \
  maven:3.8.1-jdk-11 bash
```

**Опции:**
- `-it` - Интерактивный терминал
- `--rm` - Удалить контейнер после выхода
- `-v` - Примонтировать каталог как том
- `maven:3.8.1-jdk-11` - Образ Maven с Java 11

**Использование:**
```bash
cd /opt
mvn clean install
```

## Интеграция с Jenkins

### Остановка контейнера по имени (скрипт Jenkins)

Groovy скрипт для Jenkins pipeline:

```groovy
script {
    containerName = 'tsetba'
    oldContainerID = sh(
        returnStdout: true, 
        script: "docker ps -a --filter name=${containerName} --format json | jq -r .ID"
    ).trim()
    
    if (oldContainerID != '' && oldContainerID != ' ') {
        println 'Успешная остановка'
        println oldContainerID
        sh "docker stop ${oldContainerID}"
        sh "docker rm ${oldContainerID}"
    }
}
```

**Примечания:**
- Проверить существование контейнера перед остановкой
- Удалить контейнер после остановки
- Правильно обработать проверку пустой строки (использовать `&&`, а не `||`)

## Рекомендации

1. **Используйте флаг `--rm`** для временных контейнеров
2. **Остановить перед удалением** - Всегда останавливайте контейнер перед удалением
3. **Проверка существования** - Убедитесь, что контейнер существует перед операциями
4. **Используйте фильтры** - Фильтруйте по имени/статусу для точных операций
5. **Очистка** - Удаляйте неиспользуемые контейнеры для экономии места

## Связанные заметки

- [[../Jenkins/Pipeline Scripts]]
- [[../Security/Security Tools]]

