---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - namespace
  - organization
category: infrastructure
---

# Namespace

## Обзор

Namespace обеспечивает логическое разделение ресурсов в кластере. Позволяет организовывать и изолировать ресурсы.

## Системные Namespace

- **default** - ресурсы без указанного namespace
- **kube-system** - системные компоненты Kubernetes
- **kube-public** - публичные данные (обычно пустой)
- **kube-node-lease** - информация об аренде узлов

## Пример манифеста

### Создание Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
    environment: prod
```

### Namespace с ограничениями ресурсов

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: limited-namespace
```

# ResourceQuota для ограничения ресурсов
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: limited-namespace
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

## Использование

### Указание namespace в ресурсах

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: production  # Указание namespace
spec:
  containers:
  - name: app
    image: myapp:latest
```

### Установка namespace по умолчанию

```bash
# Установить namespace для текущего контекста
kubectl config set-context --current --namespace=production

# Проверить текущий namespace
kubectl config view --minify | grep namespace
```

## Управление Namespace

### Создание

```bash
# Из файла
kubectl apply -f namespace.yaml

# Импертивно
kubectl create namespace production

# С метками
kubectl create namespace production --dry-run=client -o yaml | \
  kubectl label --local -f - name=production -o yaml | \
  kubectl apply -f -
```

### Просмотр

```bash
# Список namespace
kubectl get namespaces
kubectl get ns  # Краткая форма

# Детальная информация
kubectl describe namespace production

# Все ресурсы в namespace
kubectl get all -n production
```

### Удаление

```bash
# Удалить namespace (удалит все ресурсы внутри)
kubectl delete namespace production

# Принудительное удаление (если зависло)
kubectl delete namespace production --grace-period=0 --force
```

### Переключение между namespace

```bash
# Установить namespace по умолчанию
kubectl config set-context --current --namespace=production

# Использовать alias для удобства
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods --all-namespaces'
```

## Ограничения Namespace

### ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    persistentvolumeclaims: "10"
    services: "10"
```

### LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: production
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "250m"
    type: Container
```

## Изоляция ресурсов

### DNS внутри namespace

```bash
# Service доступен как: <service-name>.<namespace>.svc.cluster.local
# Или просто: <service-name> (если в том же namespace)

# Пример из пода в namespace production
curl http://app-service.production.svc.cluster.local
```

### Изоляция сети

Namespace не обеспечивает сетевую изоляцию по умолчанию. Используйте NetworkPolicy для этого.

### Изоляция RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: app-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

## Организация по Namespace

### Рекомендуемая структура

- **production** - production окружение
- **staging** - staging окружение
- **development** - development окружение
- **monitoring** - системы мониторинга
- **logging** - системы логирования
- **ingress-nginx** - ingress контроллеры

### Метки для организации

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
    team: backend
    cost-center: engineering
```

## Best Practices

1. **Используйте осмысленные имена** - production, staging, development
2. **Ограничивайте ресурсы** - используйте ResourceQuota
3. **Изолируйте окружения** - отдельные namespace для каждого окружения
4. **Управляйте доступом** - используйте RBAC для изоляции
5. **Мониторинг** - отслеживайте использование ресурсов по namespace
6. **Очистка** - регулярно удаляйте неиспользуемые namespace

## Полезные команды

```bash
# Все ресурсы во всех namespace
kubectl get all --all-namespaces

# Ресурсы конкретного типа во всех namespace
kubectl get pods --all-namespaces

# Ресурсы в конкретном namespace
kubectl get all -n production

# Удалить все ресурсы в namespace (кроме самого namespace)
kubectl delete all --all -n production
```

## Связанные заметки

- [[../../Overview]] - Общая информация о Kubernetes
- [[../../kubectl Commands]] - Команды для работы с namespace
- [[../Workload Resources/Pod]] - Ресурсы в namespace

