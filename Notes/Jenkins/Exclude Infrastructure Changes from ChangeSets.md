---
created: 2025-01-27
tags:
  - jenkins
  - pipeline
  - scm
  - changeset
  - groovy
  - troubleshooting
category: automation
---

# Исключение изменений инфраструктурного репозитория из ChangeSets

## Проблема

При использовании **Pipeline script from SCM** для обычных Jenkins pipeline возникает необходимость исключить информацию об изменениях в инфраструктурном репозитории (где хранится Jenkinsfile) из общего списка изменений (ChangeSets).

Базовая функциональность плагина не позволяет сделать это напрямую, в отличие от Remote Jenkinsfile Provider Plugin для Multibranch pipelines.

## Решение 1: Pre-stage (удаление всех ChangeSets)

Самый простой способ - создать pre-stage, который полностью очищает changeSets перед запуском основного pipeline.

### Преимущества

- Простая реализация
- Не требует дополнительных разрешений (script approval)
- Быстрое выполнение

### Недостатки

- Удаляет **все** изменения, включая изменения в репозитории приложения
- Нельзя фильтровать по конкретному репозиторию

### Пример реализации

```groovy
pipeline {
    agent any
    
    stages {
        stage("Delete changeset") {
            steps {
                script {
                    currentBuild.changeSets.clear()
                }
            }
        }
        
        stage("Main build") {
            steps {
                // Ваши основные шаги сборки
                echo "Building application..."
            }
        }
    }
}
```

## Решение 2: Post-actions (фильтрация по URL репозитория)

Более гибкий способ - добавление в post-actions блока, который удаляет только изменения из инфраструктурного репозитория.

### Преимущества

- Сохраняет изменения из репозитория приложения
- Фильтрация по URL репозитория
- Точный контроль над тем, что исключается

### Недостатки

- Требует script approval в Manage Jenkins → In-process Script Approval
- Более сложная реализация
- Использует функционал, заблокированный в песочнице Jenkins

### Пример реализации

```groovy
pipeline {
    agent any
    
    stages {
        stage("Main build") {
            steps {
                // Ваши основные шаги сборки
                echo "Building application..."
            }
        }
    }
    
    post {
        always {
            script {
                def oldChangeSetsList = currentBuild.changeSets
                
                def excludeChangeSetsList = []
                oldChangeSetsList.each { elem ->
                    if (elem.getBrowser()) {
                        if (elem.getBrowser().getRepoUrl().contains("jenkins-pipeline")) {
                            excludeChangeSetsList.add(elem)
                        }
                    }
                }
                
                def updatedChangeSetsList = oldChangeSetsList.findAll { 
                    !excludeChangeSetsList.contains(it) 
                }
                
                Jenkins.getInstance()
                    .getItemByFullName(env.JOB_NAME)
                    .getBuild(env.BUILD_NUMBER)
                    .changeSets = updatedChangeSetsList
            }
        }
        cleanup {
            cleanWs()
        }
    }
}
```

### Пошаговое объяснение

1. **Считывание changeSets:**
   ```groovy
   def oldChangeSetsList = currentBuild.changeSets
   ```

2. **Итерация по элементам:**
   ```groovy
   oldChangeSetsList.each { elem ->
       if (elem.getBrowser()) {
           if (elem.getBrowser().getRepoUrl().contains("jenkins-pipeline")) {
               excludeChangeSetsList.add(elem)
           }
       }
   }
   ```
   - Проверяется наличие browser (информация о репозитории)
   - Извлекается URL репозитория
   - Если URL содержит "jenkins-pipeline", элемент добавляется в список для исключения

3. **Формирование обновленного списка:**
   ```groovy
   def updatedChangeSetsList = oldChangeSetsList.findAll { 
       !excludeChangeSetsList.contains(it) 
   }
   ```
   - Создается новый список без исключенных элементов

4. **Замена оригинала:**
   ```groovy
   Jenkins.getInstance()
       .getItemByFullName(env.JOB_NAME)
       .getBuild(env.BUILD_NUMBER)
       .changeSets = updatedChangeSetsList
   ```
   - Обновляется changeSets для текущей сборки

## Настройка Script Approval

При использовании Решения 2 необходимо одобрить следующие методы:

1. Открыть: **Manage Jenkins → In-process Script Approval**
2. Одобрить методы:
   - `method hudson.model.Run getChangeSets`
   - `method hudson.model.Run setChangeSets`
   - `method hudson.plugins.git.GitChangeSetList getBrowser`
   - `method org.jenkinsci.plugins.github_branch_source.ChangeRequestAction getBrowser`
   - И другие связанные методы (Jenkins предложит их автоматически)

### Автоматическое одобрение

Можно одобрить все скрипты автоматически (не рекомендуется для production):

```groovy
// В Jenkins Script Console
import jenkins.model.Jenkins
Jenkins.instance.getExtensionList('org.jenkinsci.plugins.scriptsecurity.scripts.ScriptApproval')[0].approveAllSignatures()
```

## Кастомизация фильтрации

### Фильтрация по нескольким репозиториям

```groovy
def excludeRepos = ["jenkins-pipeline", "infrastructure", "jenkins-shared-library"]
oldChangeSetsList.each { elem ->
    if (elem.getBrowser()) {
        def repoUrl = elem.getBrowser().getRepoUrl()
        excludeRepos.each { excludeRepo ->
            if (repoUrl.contains(excludeRepo)) {
                excludeChangeSetsList.add(elem)
            }
        }
    }
}
```

### Фильтрация по точному совпадению URL

```groovy
def infrastructureRepo = "https://git.example.com/infrastructure/jenkins-pipeline.git"
oldChangeSetsList.each { elem ->
    if (elem.getBrowser()) {
        if (elem.getBrowser().getRepoUrl() == infrastructureRepo) {
            excludeChangeSetsList.add(elem)
        }
    }
}
```

### Фильтрация по паттерну

```groovy
def pattern = ~/.*jenkins.*/
oldChangeSetsList.each { elem ->
    if (elem.getBrowser()) {
        if (pattern.matcher(elem.getBrowser().getRepoUrl()).matches()) {
            excludeChangeSetsList.add(elem)
        }
    }
}
```

## Сравнение решений

| Характеристика | Pre-stage | Post-actions |
|---------------|-----------|--------------|
| Простота реализации | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Требует script approval | ❌ | ✅ |
| Сохраняет изменения приложения | ❌ | ✅ |
| Фильтрация по репозиторию | ❌ | ✅ |
| Скорость выполнения | ⚡⚡⚡⚡⚡ | ⚡⚡⚡ |

## Best Practices

1. **Используйте Pre-stage** если:
   - Не нужно сохранять изменения в ChangeSets
   - Важен простой и быстрый способ
   - Нет возможности настроить script approval

2. **Используйте Post-actions** если:
   - Нужно сохранить изменения из репозитория приложения
   - Важна точная фильтрация
   - Есть доступ к script approval

3. **Для Multibranch pipelines:**
   - Рассмотрите использование Remote Jenkinsfile Provider Plugin
   - Плагин имеет встроенную поддержку исключения инфраструктурных репозиториев

## Troubleshooting

### Изменения не удаляются

- Проверьте, что скрипт выполняется в правильном контексте
- Убедитесь, что методы одобрены в Script Approval
- Проверьте логи сборки на наличие ошибок

### Неправильная фильтрация

- Проверьте URL репозитория через:
  ```groovy
  currentBuild.changeSets.each { 
      println it.getBrowser()?.getRepoUrl() 
  }
  ```
- Убедитесь, что паттерн фильтрации корректен

### Ошибки доступа

- Проверьте права доступа Jenkins к API
- Убедитесь, что все необходимые методы одобрены

## Связанные заметки

- [[Pipeline Scripts]] - Работа с Jenkins pipelines
- [[Git Timeout Fix]] - Решение проблем с Jenkins

