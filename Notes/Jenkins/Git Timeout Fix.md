---
created: 2025-01-27
tags:
  - jenkins
  - git
  - timeout
  - multibranch-pipeline
  - troubleshooting
category: automation
---

# Исправление Git Timeout в MultiBranch Pipeline

## Проблема

При подключении больших репозиториев через MultiBranch Pipeline возникает ошибка git timeout из-за недостаточного времени на выполнение git операций.

## Решение

Увеличить таймаут для Git операций в Jenkins через системную переменную окружения.

### Добавление параметра в Jenkins

Добавить в переменные окружения Jenkins (голова/мастера):

```
-Dorg.jenkinsci.plugins.gitclient.Git.timeOut=MM
```

Где `MM` - время в минутах (например, `60` для 60 минут).

### Где изменить

1. **Jenkins UI:**
   - Manage Jenkins → Configure System
   - Раздел "Global properties" → "Environment variables"
   - Добавить переменную: `JAVA_OPTS` или `JENKINS_JAVA_OPTIONS`
   - Значение: `-Dorg.jenkinsci.plugins.gitclient.Git.timeOut=60`

2. **Через systemd (если Jenkins запущен как сервис):**
   ```bash
   # Отредактировать /etc/systemd/system/jenkins.service
   # Добавить в [Service]:
   Environment="JENKINS_JAVA_OPTIONS=-Dorg.jenkinsci.plugins.gitclient.Git.timeOut=60"
   
   # Перезагрузить сервис
   systemctl daemon-reload
   systemctl restart jenkins
   ```

3. **Через файл запуска Jenkins:**
   ```bash
   # В файле запуска Jenkins (например, /etc/default/jenkins или /etc/sysconfig/jenkins)
   JENKINS_JAVA_OPTIONS="-Dorg.jenkinsci.plugins.gitclient.Git.timeOut=60"
   ```

### Проверка применения

После перезапуска Jenkins проверить:

1. Убедиться, что Jenkins перезапустился
2. Выполнить повторную проверку подключения MultiBranch Pipeline
3. Проверить логи Jenkins на наличие ошибок

## Рекомендуемые значения

- **Средние репозитории:** 30-60 минут
- **Большие репозитории:** 60-120 минут
- **Очень большие репозитории:** 120+ минут

## Дополнительная информация

- [JENKINS-11286](https://issues.jenkins.io/browse/JENKINS-11286) - Issue о git timeout
- [JENKINS-22547](https://issues.jenkins.io/browse/JENKINS-22547) - Дополнительная информация по таймаутам

## Альтернативные решения

Если увеличение таймаута не помогает:

1. **Клонировать только необходимые ветки:**
   - Настроить в MultiBranch Pipeline фильтрацию веток
   - Использовать sparse checkout

2. **Увеличить ресурсы Jenkins:**
   - Больше памяти для JVM
   - Оптимизация Git конфигурации

3. **Использовать shallow clone:**
   - Настроить ограничение глубины истории

## Связанные заметки

- [[Pipeline Scripts]] - Настройка Jenkins pipelines
- [[../Linux/System Administration]] - Системное администрирование

