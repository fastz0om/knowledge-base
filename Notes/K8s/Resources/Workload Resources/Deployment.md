---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - deployment
  - workload
category: infrastructure
---

# Deployment

## Обзор

Deployment управляет репликами подов и обеспечивает обновления приложений без простоя.

## Основные функции

- **Репликация** - поддержка нужного количества реплик
- **Обновления** - rolling updates без простоя
- **Откаты** - возможность откатить изменения
- **История** - отслеживание версий развертываний

## Пример манифеста

### Базовый Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Deployment с ресурсами и стратегией

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Deployment с переменными окружения

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log-level
```

## Параметры манифеста

### spec.replicas

Количество желаемых реплик подов. По умолчанию: 1

### spec.selector

Селектор для выбора подов, управляемых этим Deployment.

**Обязательные поля:**
- `matchLabels` - соответствие по меткам

### spec.template

Шаблон для создания подов. Должен соответствовать селектору.

### spec.strategy

Стратегия обновления:

- **type**: `RollingUpdate` (по умолчанию) или `Recreate`

#### RollingUpdate параметры:

- **maxSurge** - максимальное количество дополнительных подов во время обновления
- **maxUnavailable** - максимальное количество недоступных подов

### spec.revisionHistoryLimit

Количество старых ReplicaSet для хранения истории. По умолчанию: 10

### spec.progressDeadlineSeconds

Максимальное время ожидания прогресса обновления. По умолчанию: 600 секунд

## Управление Deployment

### Создание

```bash
# Из файла
kubectl apply -f deployment.yaml

# Импертивно
kubectl create deployment nginx --image=nginx:1.21 --replicas=3
```

### Просмотр

```bash
# Список deployments
kubectl get deployments

# Детальная информация
kubectl describe deployment <deployment-name>

# Статус rollout
kubectl rollout status deployment/<deployment-name>
```

### Масштабирование

```bash
# Изменить количество реплик
kubectl scale deployment <deployment-name> --replicas=5

# Автомасштабирование
kubectl autoscale deployment <deployment-name> --min=2 --max=10 --cpu-percent=80
```

### Обновление

```bash
# Обновить образ
kubectl set image deployment/<deployment-name> <container-name>=new-image:v2.0

# Обновить переменные окружения
kubectl set env deployment/<deployment-name> LOG_LEVEL=debug
```

### История и откаты

```bash
# История изменений
kubectl rollout history deployment/<deployment-name>

# Откат к предыдущей версии
kubectl rollout undo deployment/<deployment-name>

# Откат к конкретной версии
kubectl rollout undo deployment/<deployment-name> --to-revision=2

# Пауза rollout
kubectl rollout pause deployment/<deployment-name>

# Возобновление rollout
kubectl rollout resume deployment/<deployment-name>
```

### Удаление

```bash
kubectl delete deployment <deployment-name>
```

## Стратегии обновления

### RollingUpdate (по умолчанию)

- Постепенная замена подов
- Без простоя
- Контролируемая скорость обновления

### Recreate

- Все поды останавливаются
- Создаются новые поды
- Простой во время обновления

## Best Practices

1. **Используйте проверки здоровья** - readiness и liveness probes
2. **Ограничьте ресурсы** - установите requests и limits
3. **Настройте стратегию** - выберите подходящую стратегию обновления
4. **Версионируйте образы** - используйте конкретные теги, избегайте `latest`
5. **Храните историю** - настройте revisionHistoryLimit

## Связанные заметки

- [[Pod]] - Базовый ресурс для подов
- [[../Networking/Service]] - Предоставление доступа к deployment
- [[../Configuration Resources/ConfigMap]] - Конфигурация для deployment
- [[../../Overview]] - Общая информация о Kubernetes

