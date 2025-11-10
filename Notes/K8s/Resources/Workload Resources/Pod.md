---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - pod
  - workload
category: infrastructure
---

# Pod

## Обзор

Pod - это наименьшая развертываемая единица в Kubernetes. Pod содержит один или несколько контейнеров, которые совместно используют сеть и хранилище.

## Основные характеристики

- **Поделимое сетевое пространство** - все контейнеры в поде имеют один IP адрес
- **Поделимое хранилище** - общие тома
- **Жизненный цикл** - под создается, запускается и завершается вместе

## Пример манифеста

### Простой Pod с одним контейнером

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      protocol: TCP
```

### Pod с несколькими контейнерами

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
  - name: redis
    image: redis:6.2
    ports:
    - containerPort: 6379
```

### Pod с переменными окружения

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      value: "postgresql://localhost/db"
    - name: LOG_LEVEL
      value: "info"
    envFrom:
    - configMapRef:
        name: app-config
```

### Pod с ресурсами и проверками

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-check-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
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
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

## Параметры манифеста

### metadata

- **name** - имя пода (обязательно, уникально в namespace)
- **namespace** - namespace для пода
- **labels** - метки для организации и фильтрации
- **annotations** - дополнительные метаданные

### spec.containers

- **name** - имя контейнера (обязательно)
- **image** - образ контейнера (обязательно)
- **imagePullPolicy** - политика загрузки образа:
  - `Always` - всегда загружать
  - `IfNotPresent` - загружать если отсутствует
  - `Never` - никогда не загружать

### spec.containers.ports

- **containerPort** - порт контейнера
- **protocol** - протокол (TCP, UDP, SCTP)

### spec.containers.resources

- **requests** - минимальные ресурсы (для планировщика)
- **limits** - максимальные ресурсы

### Probes (Проверки здоровья)

#### livenessProbe

Определяет, работает ли контейнер. При неудаче контейнер перезапускается.

**Типы:**
- `httpGet` - HTTP запрос
- `tcpSocket` - TCP подключение
- `exec` - выполнение команды

**Параметры:**
- `initialDelaySeconds` - задержка перед первой проверкой
- `periodSeconds` - интервал проверок
- `timeoutSeconds` - таймаут проверки
- `failureThreshold` - количество неудач перед действием

#### readinessProbe

Определяет, готов ли контейнер принимать трафик.

#### startupProbe

Определяет, запустился ли контейнер. Полезно для медленно запускающихся приложений.

### spec.restartPolicy

- `Always` - всегда перезапускать (по умолчанию)
- `OnFailure` - перезапускать при ошибке
- `Never` - никогда не перезапускать

## Управление Pod

### Создание

```bash
# Из файла
kubectl apply -f pod.yaml

# Импертивно
kubectl run nginx-pod --image=nginx:1.21
```

### Просмотр

```bash
# Список подов
kubectl get pods

# Детальная информация
kubectl describe pod <pod-name>

# Логи
kubectl logs <pod-name>

# Выполнение команды
kubectl exec -it <pod-name> -- /bin/bash
```

### Удаление

```bash
kubectl delete pod <pod-name>
```

## Volumes (Тома)

### EmptyDir

```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: cache
      mountPath: /cache
  volumes:
  - name: cache
    emptyDir: {}
```

### ConfigMap как том

```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

## Best Practices

1. **Не создавайте Pod напрямую** - используйте Deployment, StatefulSet или DaemonSet
2. **Настройте проверки здоровья** - liveness и readiness probes
3. **Ограничьте ресурсы** - установите requests и limits
4. **Используйте метки** - для организации и фильтрации
5. **Минимизируйте количество контейнеров** - один контейнер на под предпочтительнее

## Связанные заметки

- [[../../Overview]] - Общая информация о Kubernetes
- [[Pod Template Image Pull Policy]] - Решение проблемы alwaysPullImage в Jenkins podTemplate
- [[Deployment]] - Управление подами через Deployment
- [[../Configuration Resources/ConfigMap]] - Конфигурация через ConfigMap
- [[../Networking/Service]] - Предоставление доступа к подам

