---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - rolebinding
  - rbac
  - security
category: infrastructure
---

# RoleBinding и ClusterRoleBinding

## Обзор

RoleBinding и ClusterRoleBinding связывают Role/ClusterRole с субъектами (пользователями, группами, ServiceAccount), предоставляя им соответствующие права доступа.

## RoleBinding vs ClusterRoleBinding

| Характеристика | RoleBinding | ClusterRoleBinding |
|---------------|-------------|-------------------|
| Область действия | Один namespace | Весь кластер |
| Может использовать | Role или ClusterRole | Только ClusterRole |
| Использование | Локальные права | Глобальные права |

## Пример манифеста

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

### RoleBinding с несколькими субъектами

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-managers
  namespace: production
subjects:
# ServiceAccount
- kind: ServiceAccount
  name: app-sa
  namespace: production
# Пользователь
- kind: User
  name: "john@example.com"
  apiGroup: rbac.authorization.k8s.io
# Группа
- kind: Group
  name: "developers"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: app-manager
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding для доступа ко всем namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-all-namespaces
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: view  # Системная роль для просмотра
  apiGroup: rbac.authorization.k8s.io
```

### RoleBinding с ClusterRole (доступ к namespace)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-admin-in-namespace
  namespace: production
subjects:
- kind: ServiceAccount
  name: admin-sa
  namespace: production
roleRef:
  kind: ClusterRole
  name: cluster-admin  # ClusterRole, но применяется только к namespace
  apiGroup: rbac.authorization.k8s.io
```

## Параметры манифеста

### subjects

Список субъектов, которым предоставляются права:

**ServiceAccount:**
```yaml
- kind: ServiceAccount
  name: app-serviceaccount
  namespace: default
```

**User:**
```yaml
- kind: User
  name: "john@example.com"
  apiGroup: rbac.authorization.k8s.io
```

**Group:**
```yaml
- kind: Group
  name: "developers"
  apiGroup: rbac.authorization.k8s.io
```

### roleRef

Ссылка на Role или ClusterRole:

```yaml
roleRef:
  kind: Role  # или ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Важно:** `roleRef` нельзя изменить после создания. Нужно пересоздать RoleBinding.

## Использование с разными типами субъектов

### ServiceAccount

```yaml
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default  # Обязательно указать namespace для ServiceAccount
```

### Пользователь

```yaml
subjects:
- kind: User
  name: "user@example.com"
  apiGroup: rbac.authorization.k8s.io
```

**Примечание:** Пользователи Kubernetes обычно управляются через внешние системы аутентификации.

### Группа

```yaml
subjects:
- kind: Group
  name: "developers"
  apiGroup: rbac.authorization.k8s.io
```

### Все ServiceAccount в namespace

```yaml
subjects:
- kind: Group
  name: system:serviceaccounts:default  # Все ServiceAccount в namespace default
  apiGroup: rbac.authorization.k8s.io
```

### Все аутентифицированные пользователи

```yaml
subjects:
- kind: Group
  name: system:authenticated
  apiGroup: rbac.authorization.k8s.io
```

## Управление RoleBinding

### Создание

```bash
# Из файла
kubectl apply -f rolebinding.yaml
```

### Просмотр

```bash
# Список RoleBinding
kubectl get rolebindings
kubectl get rolebinding  # Также работает

# Список ClusterRoleBinding
kubectl get clusterrolebindings
kubectl get clusterrolebinding

# Детальная информация
kubectl describe rolebinding <rolebinding-name>
kubectl describe clusterrolebinding <clusterrolebinding-name>
```

### Проверка прав субъекта

```bash
# Проверить права ServiceAccount
kubectl auth can-i create pods \
  --as=system:serviceaccount:default:app-serviceaccount

# Проверить права пользователя
kubectl auth can-i delete deployments \
  --as=user@example.com \
  --namespace=production
```

### Удаление

```bash
kubectl delete rolebinding <rolebinding-name>
kubectl delete clusterrolebinding <clusterrolebinding-name>
```

## Комбинации Role и RoleBinding

### RoleBinding с Role (локальные права)

```yaml
# Role в namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding в том же namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: read-pods
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### RoleBinding с ClusterRole (доступ к namespace)

```yaml
# ClusterRole (глобальный)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding в конкретном namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: production  # Применяется только к этому namespace
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: production
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding с ClusterRole (глобальные права)

```yaml
# ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
---
# ClusterRoleBinding (глобальный)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-global
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## Best Practices

1. **Минимальные права** - предоставляйте только необходимые права
2. **Используйте ServiceAccount** - для подов вместо пользователей
3. **RoleBinding для namespace** - если достаточно локальных прав
4. **ClusterRoleBinding с осторожностью** - только для глобальных прав
5. **Группируйте права** - один RoleBinding для группы связанных прав
6. **Регулярно пересматривайте** - удаляйте неиспользуемые RoleBinding

## Связанные заметки

- [[Role]] - Определение прав доступа
- [[ServiceAccount]] - Идентичность для подов
- [[../../Overview]] - Общая информация о Kubernetes

