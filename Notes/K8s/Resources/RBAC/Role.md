---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - role
  - rbac
  - security
category: infrastructure
---

# Role и ClusterRole

## Обзор

Role и ClusterRole определяют набор правил (permissions) для доступа к ресурсам Kubernetes. Role применяется к namespace, ClusterRole - ко всему кластеру.

## Role vs ClusterRole

| Характеристика | Role | ClusterRole |
|---------------|------|-------------|
| Область действия | Один namespace | Весь кластер |
| Ресурсы кластера | Нет | Да (nodes, persistentVolumes) |
| Использование | Локальные права | Глобальные права |

## Пример манифеста

### Role для чтения подов

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]  # "" = core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### Role для управления Deployment

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: deployment-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments/status"]
  verbs: ["get"]
```

### Role с несколькими ресурсами

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: app-manager
rules:
# Полный доступ к подам и сервисам
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["*"]  # Все действия
# Чтение ConfigMap и Secret
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
```

### Role с subresources

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-exec
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods/exec"]  # Subresource
  verbs: ["create"]
```

### ClusterRole для доступа к узлам

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### ClusterRole для всех ресурсов

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-view
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]  # Только чтение
```

### ClusterRole с агрегацией

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  matchLabels:
    rbac.example.com/aggregate-to-monitoring: "true"
rules: []  # Правила будут агрегированы из других ClusterRole
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-pods
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

## Параметры манифеста

### apiGroups

API группы ресурсов:
- `""` - core API group (pods, services, configmaps)
- `"apps"` - apps API group (deployments, statefulsets)
- `"rbac.authorization.k8s.io"` - RBAC API group

### resources

Ресурсы Kubernetes:
- `pods`, `services`, `deployments`, `configmaps`, `secrets`
- `nodes` (только для ClusterRole)
- `persistentvolumes` (только для ClusterRole)

### verbs

Действия:
- `get` - получить один ресурс
- `list` - список ресурсов
- `watch` - отслеживать изменения
- `create` - создать ресурс
- `update` - обновить ресурс
- `patch` - частично обновить
- `delete` - удалить ресурс
- `*` - все действия

### resourceNames

Ограничение конкретными ресурсами:

```yaml
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["my-secret"]  # Только конкретный Secret
  verbs: ["get"]
```

### nonResourceURLs

Для ClusterRole - доступ к не-ресурсным URL:

```yaml
rules:
- nonResourceURLs: ["/metrics", "/healthz"]
  verbs: ["get"]
```

## Использование с RoleBinding

### RoleBinding для ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-global
subjects:
- kind: ServiceAccount
  name: app-serviceaccount
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## Управление Role

### Создание

```bash
# Из файла
kubectl apply -f role.yaml
```

### Просмотр

```bash
# Список Role
kubectl get roles

# Список ClusterRole
kubectl get clusterroles

# Детальная информация
kubectl describe role <role-name>
kubectl describe clusterrole <clusterrole-name>
```

### Проверка прав

```bash
# Проверить права текущего пользователя
kubectl auth can-i create pods
kubectl auth can-i delete pods --namespace production

# Проверить права ServiceAccount
kubectl auth can-i create pods \
  --as=system:serviceaccount:default:app-serviceaccount
```

### Удаление

```bash
kubectl delete role <role-name>
kubectl delete clusterrole <clusterrole-name>
```

## Системные роли

Kubernetes создает системные ClusterRole:

- `cluster-admin` - полный доступ к кластеру
- `admin` - доступ к ресурсам namespace
- `edit` - редактирование ресурсов namespace
- `view` - только просмотр ресурсов namespace

## Best Practices

1. **Принцип наименьших привилегий** - предоставляйте только необходимые права
2. **Используйте Role для namespace** - если достаточно локальных прав
3. **Используйте ClusterRole с осторожностью** - только для глобальных прав
4. **Группируйте права логически** - создавайте роли для конкретных задач
5. **Регулярно пересматривайте права** - удаляйте неиспользуемые роли

## Связанные заметки

- [[RoleBinding]] - Привязка ролей к субъектам
- [[ServiceAccount]] - Идентичность для подов
- [[../../Overview]] - Общая информация о Kubernetes

