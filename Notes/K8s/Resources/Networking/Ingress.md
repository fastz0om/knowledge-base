---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - ingress
  - networking
category: infrastructure
---

# Ingress

## Обзор

Ingress управляет внешним доступом к сервисам в кластере, обычно предоставляя HTTP/HTTPS маршрутизацию. Требует установки Ingress Controller.

## Ingress Controller

Популярные контроллеры:
- **nginx-ingress** - Nginx Ingress Controller
- **traefik** - Traefik
- **istio** - Istio Gateway
- **AWS ALB** - AWS Application Load Balancer

## Пример манифеста

### Базовый Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Ingress с несколькими хостами

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 8080
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### Ingress с TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    - www.example.com
    secretName: tls-secret  # Secret с TLS сертификатом
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Ingress с аннотациями (nginx)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### Ingress с путями

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /static
        pathType: Exact
        backend:
          service:
            name: static-service
            port:
              number: 80
```

## Параметры манифеста

### spec.ingressClassName

Имя Ingress Class для выбора контроллера. В Kubernetes 1.18+ заменил аннотацию `kubernetes.io/ingress.class`.

### spec.rules

Правила маршрутизации:

- **host** - имя хоста (опционально, если не указан - для всех хостов)
- **http.paths** - пути для маршрутизации

### spec.rules[].http.paths[].path

Путь URL:
- `/api` - точный путь
- `/api/` - префикс

### spec.rules[].http.paths[].pathType

Тип соответствия пути:

- **Exact** - точное соответствие
- **Prefix** - соответствие префиксу (по умолчанию самый длинный префикс)
- **ImplementationSpecific** - зависит от контроллера

### spec.rules[].http.paths[].backend

Сервис для маршрутизации:

```yaml
backend:
  service:
    name: web-service
    port:
      number: 80  # Или name: http
```

### spec.tls

TLS конфигурация:

```yaml
tls:
- hosts:
  - example.com
  secretName: tls-secret
```

Требуется Secret с типом `kubernetes.io/tls` содержащий `tls.crt` и `tls.key`.

### spec.defaultBackend

Сервис по умолчанию (если не совпадает ни одно правило):

```yaml
defaultBackend:
  service:
    name: default-service
    port:
      number: 80
```

## Создание TLS Secret

```bash
# Из файлов сертификата
kubectl create secret tls tls-secret \
  --cert=path/to/cert.pem \
  --key=path/to/key.pem

# Или из манифеста
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

## Аннотации (nginx-ingress)

### Базовые аннотации

```yaml
annotations:
  # Редирект на HTTPS
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  
  # Размер тела запроса
  nginx.ingress.kubernetes.io/proxy-body-size: "10m"
  
  # Rewrite пути
  nginx.ingress.kubernetes.io/rewrite-target: /
  
  # Включить CORS
  nginx.ingress.kubernetes.io/enable-cors: "true"
  
  # Rate limiting
  nginx.ingress.kubernetes.io/rate-limit: "100"
  
  # Basic Auth
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: basic-auth
```

### Полезные аннотации

```yaml
annotations:
  # Session Affinity
  nginx.ingress.kubernetes.io/affinity: "cookie"
  nginx.ingress.kubernetes.io/session-cookie-name: "route"
  
  # Проксирование WebSocket
  nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
  nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
  
  # Кастомная конфигурация
  nginx.ingress.kubernetes.io/server-snippet: |
    more_set_headers "X-Custom-Header: value";
```

## Управление Ingress

### Создание

```bash
# Из файла
kubectl apply -f ingress.yaml
```

### Просмотр

```bash
# Список Ingress
kubectl get ingress
kubectl get ing  # Краткая форма

# Детальная информация
kubectl describe ingress <ingress-name>

# Проверка правил маршрутизации
kubectl get ingress <ingress-name> -o yaml
```

### Тестирование

```bash
# Проверить доступность (требует настроенный Ingress Controller)
curl -H "Host: example.com" http://<ingress-ip>/

# С TLS
curl -k https://example.com/
```

### Удаление

```bash
kubectl delete ingress <ingress-name>
```

## Ingress Class

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

## Best Practices

1. **Используйте TLS** - для всех production сервисов
2. **Правильный pathType** - выбирайте подходящий тип пути
3. **Ограничивайте ресурсы** - используйте rate limiting
4. **Мониторинг** - отслеживайте доступность через Ingress
5. **Используйте аннотации** - для расширенной конфигурации контроллера

## Связанные заметки

- [[Service]] - Сервисы для маршрутизации
- [[../Configuration Resources/Secret]] - TLS сертификаты
- [[../../Overview]] - Общая информация о Kubernetes

