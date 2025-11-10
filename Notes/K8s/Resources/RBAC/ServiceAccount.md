---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - serviceaccount
  - rbac
  - security
category: infrastructure
---

# ServiceAccount

## Обзор

ServiceAccount предоставляет идентичность для подов, работающих в кластере. Используется для RBAC (Role-Based Access Control) и доступа к API Kubernetes.

## Системные ServiceAccount

- **default** - ServiceAccount по умолчанию для каждого namespace
- **system:serviceaccount:kube-system:default** - системный ServiceAccount

## Пример манифеста

### Базовый ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-serviceaccount
  namespace: default
```

### ServiceAccount с секретом для imagePullSecrets

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: private-registry-sa
  namespace: default
imagePullSecrets:
- name: docker-registry-secret
```

### ServiceAccount с аннотациями

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/app-role
```

## Использование в Pod

### Указание ServiceAccount в Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  serviceAccountName: app-serviceaccount  # Использовать конкретный ServiceAccount
  containers:
  - name: app
    image: myapp:latest
```

### Использование в Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  template:
    spec:
      serviceAccountName: app-serviceaccount
      containers:
      - name: app
        image: myapp:latest
```

## Автоматическое создание Secret

Kubernetes автоматически создает Secret для каждого ServiceAccount:

```bash
# Просмотр созданных Secret
kubectl get secrets
# Имя: <serviceaccount-name>-token-<random>

# Содержимое Secret содержит:
# - namespace
# - ca.crt (CA сертификат кластера)
# - token (JWT токен для доступа к API)
```

## Доступ к API из Pod

### Использование ServiceAccount токена

```bash
# Токен автоматически монтируется в:
# /var/run/secrets/kubernetes.io/serviceaccount/token

# CA сертификат:
# /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Namespace:
# /var/run/secrets/kubernetes.io/serviceaccount/namespace

# Пример использования в поде
curl -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

## RBAC с ServiceAccount

### Role для ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### RoleBinding

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

### ClusterRole и ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
---
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
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

## Управление ServiceAccount

### Создание

```bash
# Из файла
kubectl apply -f serviceaccount.yaml

# Импертивно
kubectl create serviceaccount app-sa
```

### Просмотр

```bash
# Список ServiceAccount
kubectl get serviceaccounts
kubectl get sa  # Краткая форма

# Детальная информация
kubectl describe serviceaccount <serviceaccount-name>

# Секреты ServiceAccount
kubectl get secrets -l kubernetes.io/service-account.name=<serviceaccount-name>
```

### Удаление

```bash
kubectl delete serviceaccount <serviceaccount-name>
```

## Автоматическое монтирование Secret

По умолчанию каждый под автоматически монтирует Secret своего ServiceAccount. Отключить:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  automountServiceAccountToken: false  # Отключить автоматическое монтирование
  containers:
  - name: app
    image: myapp:latest
```

## Best Practices

1. **Создавайте отдельные ServiceAccount** - для каждого приложения
2. **Минимальные права** - предоставляйте только необходимые права (принцип наименьших привилегий)
3. **Используйте RBAC** - связывайте ServiceAccount с Role/ClusterRole
4. **Не используйте default** - создавайте именованные ServiceAccount
5. **Мониторинг доступа** - отслеживайте использование ServiceAccount токенов

## Security Considerations

1. **Токены не истекают** - используйте внешние системы управления токенами для production
2. **Ограничивайте доступ** - используйте RBAC для ограничения прав
3. **Ротация токенов** - регулярно пересоздавайте ServiceAccount для новых токенов
4. **Аудит** - отслеживайте использование ServiceAccount в логах аудита

## Связанные заметки

- [[Role]] - Роли для RBAC
- [[RoleBinding]] - Привязка ролей к ServiceAccount
- [[../Configuration Resources/Secret]] - Автоматически создаваемые Secret
- [[../Workload Resources/Pod]] - Использование ServiceAccount в подах

