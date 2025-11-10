---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - jenkins
  - pod-template
  - image-pull-policy
  - troubleshooting
category: infrastructure
---

# PodTemplate: alwaysPullImage vs imagePullPolicy

## Предыстория

У вас есть `podTemplate` в Jenkins, в котором находится образ `nexus.example.local/docker-example/jenkins-gradle-slave:java17`.

Вы обновили образ и не понимаете, почему пайплайн до сих пор не использует обновленный образ. Ведь у вас стоит `alwaysPullImage: true`, который согласно инструкции должен делать следующее:

> **Примечание:** Если установлено, последняя версия образа будет загружаться каждый раз при его использовании. См. [Images - Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#updating-images) для поведения Kubernetes по умолчанию.

## Проблема

Вы не понимаете, в чем дело и думаете - что проблема в вас? Начинаете читать документацию и видите там то, что по идее должно работать. Ведь вы попадаете под условие из документации Kubernetes:

> **Политика Always:**
> - Установите `imagePullPolicy` для контейнера значение `Always`
> - Опустите `imagePullPolicy` и используйте `:latest` в качестве тега для изображения; Kubernetes установит политику `Always` при отправке пода
> - Опустите `imagePullPolicy` тег и для используемого изображения; Kubernetes установит политику `Always` при отправке пода

**Однако при этом:**

> **Если вы опустите `imagePullPolicy`** и укажете тег для образа контейнера, который **не равен `:latest`**, то `imagePullPolicy` автоматически будет установлено значение `IfNotPresent`.

## Корень проблемы

Если вы в UI найдете информацию о созданном с помощью пайплайна pod, вы увидите:

![Image: Pod с IfNotPresent policy](Attachments/Jenkins/allwaysPullImage-error.png)

Что означает: **`alwaysPullImage: true` не работает должным образом** → применяется дефолтная политика `IfNotPresent` (см. пункт выше).

## Решение

Избавиться от параметра `alwaysPullImage: true` и использовать вместо него: **`imagePullPolicy: Always`**, который обеспечит желаемое поведение:

![Image: Pod с Always policy](Attachments/Jenkins/allwaysPullImage-fix.png)

## Пример корректного podTemplate

### С imagePullPolicy: Always

```groovy
def podTemplate = '''
    apiVersion: v1
    kind: Pod
    metadata:
      labels: 
        some-label: gitleaks-scan
    spec:
      nodeSelector:
        node: "ci"
        kubernetes.io/os: "linux"
      containers:
      - name: jnlp
        image: nexus.example.local/docker-example/jenkins-inbound:latest
        imagePullPolicy: Always
      - name: gitleaks
        image: nexus.example.local/docker-example/gitleaks:latest
        imagePullPolicy: Always
        securityContext:
          fsGroup: 1000
        command:
        - sleep
        args:
        - 99d
      dnsConfig:
        options:
        - name: ndots
          value: "2"
      imagePullSecrets:
      - name: nexus-registry
    '''
```

### Неправильный вариант (alwaysPullImage)

```groovy
def podTemplate = '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: jnlp
        image: nexus.example.local/docker-example/jenkins-gradle-slave:java17
        # ❌ Не работает для тегов != latest
        alwaysPullImage: true
    '''
```

## Объяснение проблемы

### Почему alwaysPullImage не работает

1. **Плагин Kubernetes Plugin** использует `alwaysPullImage: true` только как подсказку
2. **Если тег != latest**, Kubernetes автоматически устанавливает `imagePullPolicy: IfNotPresent`
3. **Kubernetes игнорирует** `alwaysPullImage` параметр Jenkins плагина при формировании Pod манифеста
4. **Результат:** используется старый образ из локального кэша

### Как работает imagePullPolicy: Always

1. **Kubernetes всегда загружает образ** при создании пода
2. **Работает для любого тега** (не только `:latest`)
3. **Явное указание политики** в манифесте Pod
4. **Гарантированное обновление** образа при каждом запуске

## Рекомендации

### Для разных типов образов

1. **Образы с тегом `:latest`:**
   ```yaml
   image: myimage:latest
   imagePullPolicy: Always  # Рекомендуется явно указать
   ```

2. **Образы с конкретным тегом:**
   ```yaml
   image: myimage:v1.2.3
   imagePullPolicy: Always  # Обязательно для принудительного обновления
   ```

3. **Образы без тега:**
   ```yaml
   image: myimage
   imagePullPolicy: Always  # Рекомендуется
   ```

### Использование в Jenkins Pipeline

```groovy
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: nexus.example.local/docker-example/jenkins-slave:latest
    imagePullPolicy: Always
  - name: gradle
    image: nexus.example.local/docker-example/jenkins-gradle-slave:java17
    imagePullPolicy: Always
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('gradle') {
                    sh './gradlew build'
                }
            }
        }
    }
}
```

## Проверка работы

### Проверить текущую политику

```bash
# Получить информацию о поде
kubectl get pod <pod-name> -o yaml | grep imagePullPolicy

# Или через describe
kubectl describe pod <pod-name> | grep "Image:"
```

### Логи для отладки

```groovy
// В Jenkins pipeline для проверки
sh '''
    kubectl get pod $HOSTNAME -o yaml | grep imagePullPolicy
    kubectl describe pod $HOSTNAME | grep "Image:"
'''
```

## Best Practices

1. **Всегда используйте `imagePullPolicy: Always`** для образов, которые должны обновляться
2. **Не полагайтесь на `alwaysPullImage: true`** - он не работает для тегов != latest
3. **Для production:** используйте конкретные теги с `imagePullPolicy: Always` для контроля версий
4. **Проверяйте в UI Kubernetes** - смотрите фактическую политику в описании пода
5. **Для development:** можно использовать `:latest` с явным `Always` для автоматических обновлений

## Дополнительная информация

### Разница между политиками

| Политика | Поведение |
|---------|-----------|
| `Always` | Всегда загружать образ из registry |
| `IfNotPresent` | Загружать только если отсутствует локально |
| `Never` | Никогда не загружать из registry |

### Поведение по умолчанию

- **`:latest` тег** → `Always` (если не указано)
- **Конкретный тег** → `IfNotPresent` (если не указано)
- **Без тега** → `Always` (если не указано)

## Связанные заметки

- [[Pod]] - Основная информация о Pod в Kubernetes
- [[../../Jenkins/Pipeline Scripts]] - Скрипты для Jenkins pipelines
- [[../../Jenkins/Using Credentials in Active Choice Parameter]] - Использование секретов в Jenkins

