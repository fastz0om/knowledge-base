---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - secret
  - security
category: infrastructure
---

# Secret

## Обзор

Secret хранит чувствительные данные, такие как пароли, токены и ключи. Данные хранятся в base64, но не шифруются по умолчанию.

## Типы Secret

- **Opaque** - произвольные данные (по умолчанию)
- **kubernetes.io/dockerconfigjson** - учетные данные Docker registry
- **kubernetes.io/tls** - TLS сертификаты
- **kubernetes.io/service-account-token** - токены ServiceAccount

## Пример манифеста

### Opaque Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  # stringData автоматически кодируется в base64
  username: admin
  password: secret-password
  database-url: postgresql://user:pass@localhost/db

# Или data с уже закодированными данными
# data:
#   username: YWRtaW4=  # base64 encoded
#   password: c2VjcmV0LXBhc3N3b3Jk
```

### TLS Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>

# Или создание из файлов
# kubectl create secret tls tls-secret \
#   --cert=path/to/cert.pem \
#   --key=path/to/key.pem
```

### Docker Registry Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>

# Или создание импертивно
# kubectl create secret docker-registry docker-registry-secret \
#   --docker-server=registry.example.com \
#   --docker-username=user \
#   --docker-password=pass \
#   --docker-email=user@example.com
```

## Использование в Pod

### Как переменные окружения

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - secretRef:
        name: app-secret
    # Или отдельные ключи
    env:
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

### Как файлы (том)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
      # Опционально: выбрать конкретные ключи
      items:
      - key: password
        path: db-password
        mode: 0400
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
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: password
        volumeMounts:
        - name: secrets
          mountPath: /etc/secrets
      volumes:
      - name: secrets
        secret:
          secretName: app-secret
```

## Параметры манифеста

### metadata

- **name** - имя Secret (обязательно)
- **namespace** - namespace для Secret

### type

Тип Secret:
- `Opaque` - произвольные данные
- `kubernetes.io/dockerconfigjson` - Docker registry
- `kubernetes.io/tls` - TLS сертификаты
- `kubernetes.io/service-account-token` - ServiceAccount токен

### data

Данные в base64 формате. Используется для уже закодированных данных.

### stringData

Текстовые данные, которые автоматически кодируются в base64 при создании. Удобно для редактирования манифестов.

## Управление Secret

### Создание

```bash
# Из файла
kubectl apply -f secret.yaml

# Импертивно из литералов
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=secret

# Из файлов
kubectl create secret generic app-secret \
  --from-file=username.txt \
  --from-file=password.txt

# TLS Secret
kubectl create secret tls tls-secret \
  --cert=cert.pem \
  --key=key.pem

# Docker Registry Secret
kubectl create secret docker-registry docker-secret \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass
```

### Просмотр

```bash
# Список Secret
kubectl get secrets

# Детальная информация (без значений)
kubectl describe secret <secret-name>

# Данные Secret (base64)
kubectl get secret <secret-name> -o yaml

# Декодирование значения
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
```

### Редактирование

```bash
# Интерактивное редактирование
kubectl edit secret <secret-name>
```

### Удаление

```bash
kubectl delete secret <secret-name>
```

## Безопасность

### Ограничения

- Secret хранятся в etcd в base64 (не шифруются по умолчанию)
- Требуется включение encryption at rest для etcd
- Используйте RBAC для ограничения доступа
- Рассмотрите использование внешних систем управления секретами (Vault, Sealed Secrets)

### Защита Secret

```yaml
# Использование immutable для предотвращения изменений
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
immutable: true
type: Opaque
data:
  password: <base64-encoded>
```

### RBAC для Secret

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["app-secret"]
  verbs: ["get", "list"]
```

## Использование с ImagePullSecrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-registry-pod
spec:
  imagePullSecrets:
  - name: docker-registry-secret
  containers:
  - name: app
    image: registry.example.com/myapp:latest
```

## Best Practices

1. **Не коммитьте Secret** - используйте отдельные файлы, не в git
2. **Используйте внешние системы** - Vault, Sealed Secrets для production
3. **Включите encryption at rest** - для защиты в etcd
4. **Минимизируйте доступ** - используйте RBAC
5. **Ротация секретов** - регулярно обновляйте секреты
6. **Используйте immutable** - для предотвращения случайных изменений

## Связанные заметки

- [[ConfigMap]] - Хранение несекретных конфигураций
- [[../RBAC/ServiceAccount]] - Автоматическое создание Secret для ServiceAccount
- [[../Workload Resources/Pod]] - Использование Secret в подах

