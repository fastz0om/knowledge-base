---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - daemonset
  - workload
category: infrastructure
---

# DaemonSet

## Обзор

DaemonSet обеспечивает запуск одного пода на каждом узле кластера (или на выбранных узлах). Полезен для системных агентов, логирования, мониторинга.

## Использование

- Логирование (log collector на каждом узле)
- Мониторинг (метрики узлов)
- Сетевые плагины
- Системные агенты

## Пример манифеста

### Базовый DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logging
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd-logging
  template:
    metadata:
      labels:
        name: fluentd-logging
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            memory: 200Mi
            cpu: 100m
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### DaemonSet с nodeSelector

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disktype: ssd  # Только на узлах с меткой disktype=ssd
      containers:
      - name: monitor
        image: monitor:latest
```

### DaemonSet с tolerations

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
      # Запускать даже на master узлах
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.3.1
        ports:
        - containerPort: 9100
          hostPort: 9100
```

## Параметры манифеста

### spec.selector

Селектор для выбора подов, управляемых этим DaemonSet.

### spec.template

Шаблон для создания подов. Каждый под будет создан на узле, который соответствует селектору.

### spec.updateStrategy

Стратегия обновления:

```yaml
updateStrategy:
  type: RollingUpdate  # или OnDelete
  rollingUpdate:
    maxUnavailable: 1  # Максимум недоступных подов при обновлении
```

- **RollingUpdate** - постепенное обновление (по умолчанию)
- **OnDelete** - обновление только при удалении подов

### spec.template.spec.nodeSelector

Выбор узлов для запуска подов:

```yaml
nodeSelector:
  disktype: ssd
  kubernetes.io/os: linux
```

### spec.template.spec.tolerations

Толерантности для запуска на tainted узлах:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

## Управление DaemonSet

### Создание

```bash
# Из файла
kubectl apply -f daemonset.yaml
```

### Просмотр

```bash
# Список DaemonSet
kubectl get daemonsets
kubectl get ds  # Краткая форма

# Детальная информация
kubectl describe daemonset <daemonset-name>

# Поды DaemonSet
kubectl get pods -l app=fluentd
```

### Обновление

```bash
# Обновить образ
kubectl set image daemonset/<daemonset-name> <container-name>=new-image:v2.0

# Проверить статус обновления
kubectl rollout status daemonset/<daemonset-name>
```

### Удаление

```bash
# Удалить DaemonSet (поды также удалятся)
kubectl delete daemonset <daemonset-name>

# Удалить с удалением подов
kubectl delete daemonset <daemonset-name> --cascade=foreground
```

## Использование с hostPath

DaemonSet часто использует hostPath для доступа к файлам узла:

```yaml
volumes:
- name: varlog
  hostPath:
    path: /var/log
    type: DirectoryOrCreate
```

## Использование с hostNetwork

```yaml
spec:
  template:
    spec:
      hostNetwork: true  # Использовать сеть узла
      containers:
      - name: agent
        image: agent:latest
```

## Best Practices

1. **Используйте для системных задач** - логирование, мониторинг, сетевые плагины
2. **Ограничивайте ресурсы** - каждый под на узле потребляет ресурсы
3. **Используйте nodeSelector** - для запуска на определенных узлах
4. **Настройте tolerations** - для запуска на master узлах при необходимости
5. **Мониторинг** - отслеживайте состояние подов на каждом узле

## Сравнение с Deployment

| Характеристика | DaemonSet | Deployment |
|---------------|-----------|------------|
| Количество подов | Один на узел | Указанное количество |
| Распределение | На всех узлах | По запросу |
| Использование | Системные задачи | Приложения |

## Связанные заметки

- [[Deployment]] - Для приложений с указанным количеством реплик
- [[Pod]] - Базовый ресурс
- [[../../Overview]] - Общая информация о Kubernetes

