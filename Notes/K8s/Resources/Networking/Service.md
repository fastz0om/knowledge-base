---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - service
  - networking
category: infrastructure
---

# Service

## Обзор

Service обеспечивает стабильный сетевой endpoint для доступа к подам. Service абстрагирует изменяющиеся IP адреса подов.

## Типы Service

1. **ClusterIP** - внутренний IP (по умолчанию)
2. **NodePort** - порт на каждом узле
3. **LoadBalancer** - внешний балансировщик нагрузки
4. **ExternalName** - ссылка на внешний сервис

## Пример манифеста

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
    protocol: TCP
```

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  externalIPs:
  - 192.168.1.100
```

### Service с несколькими портами

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: 9090
    protocol: TCP
```

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None  # Headless service
  selector:
    app: stateful-app
  ports:
  - port: 80
    targetPort: 8080
```

## Параметры манифеста

### spec.type

Тип Service:
- **ClusterIP** - только внутри кластера (по умолчанию)
- **NodePort** - доступ через порт узла
- **LoadBalancer** - внешний балансировщик (облачные провайдеры)
- **ExternalName** - ссылка на внешний DNS

### spec.selector

Метки для выбора подов. Service направляет трафик на поды с этими метками.

### spec.ports

Массив портов:

- **port** - порт Service
- **targetPort** - порт подов (по умолчанию равен port)
- **nodePort** - порт на узле (только для NodePort)
- **protocol** - протокол (TCP, UDP, SCTP)
- **name** - имя порта

### spec.clusterIP

IP адрес Service. Для Headless Service установите `None`.

### spec.externalIPs

Внешние IP адреса для доступа к Service.

### spec.sessionAffinity

Привязка сессии:
- `None` - без привязки (по умолчанию)
- `ClientIP` - трафик от одного клиента идет на один под

### spec.externalTrafficPolicy

Для NodePort и LoadBalancer:
- `Cluster` - трафик может идти на любой узел
- `Local` - трафик идет только на узлы с подами

## Управление Service

### Создание

```bash
# Из файла
kubectl apply -f service.yaml

# Импертивно
kubectl expose deployment nginx-deployment --port=80 --target-port=8080
```

### Просмотр

```bash
# Список services
kubectl get services

# Детальная информация
kubectl describe service <service-name>

# Endpoints (поды за сервисом)
kubectl get endpoints <service-name>
```

### Удаление

```bash
kubectl delete service <service-name>
```

## Доступ к Service

### Внутри кластера

```bash
# DNS имя: <service-name>.<namespace>.svc.cluster.local
# Или просто: <service-name>.<namespace>
# В том же namespace: <service-name>

# Пример из пода
curl http://nginx-service.default.svc.cluster.local
```

### Внешний доступ

#### NodePort

```bash
# Доступ через <NodeIP>:<NodePort>
curl http://<node-ip>:30080
```

#### LoadBalancer

```bash
# Доступ через внешний IP балансировщика
kubectl get service <service-name>
# Использовать EXTERNAL-IP из вывода
```

## Селекторы и метки

Service использует селекторы для поиска подов:

```yaml
# Service
spec:
  selector:
    app: nginx
    tier: frontend

# Pod должен иметь соответствующие метки
metadata:
  labels:
    app: nginx
    tier: frontend
```

## Headless Service

Headless Service (clusterIP: None) не предоставляет балансировку. Используется для:
- StatefulSet
- Прямого доступа к подам по DNS
- Service discovery для отдельных подов

## Best Practices

1. **Используйте метки** - для правильного селектора
2. **Именуйте порты** - для Service с несколькими портами
3. **Headless для StatefulSet** - для прямого доступа к подам
4. **SessionAffinity при необходимости** - для stateful приложений
5. **ExternalTrafficPolicy: Local** - для сохранения source IP

## Связанные заметки

- [[../Workload Resources/Deployment]] - Развертывание приложений
- [[../Workload Resources/Pod]] - Базовый ресурс
- [[Ingress]] - Управление входящим трафиком
- [[../../Overview]] - Общая информация о Kubernetes

