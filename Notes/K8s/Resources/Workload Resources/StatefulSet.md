---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - statefulset
  - workload
category: infrastructure
---

# StatefulSet

## Обзор

StatefulSet управляет развертыванием и масштабированием набора подов, обеспечивая упорядоченность и уникальность для stateful приложений (базы данных, кластеры).

## Особенности

- **Стабильные идентификаторы** - каждый под имеет уникальное имя
- **Упорядоченное развертывание** - поды создаются и удаляются по порядку
- **Стабильное хранилище** - каждый под имеет свой PersistentVolume
- **Стабильная сеть** - Headless Service для прямого доступа к подам

## Пример манифеста

### Базовый StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"  # Имя Headless Service (обязательно)
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### StatefulSet для базы данных

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
      - name: init-mysql
        image: mysql:5.7
        command:
        - bash
        - "-c"
        - |
          set -ex
          [[ $HOSTNAME =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          echo [mysqld] > /mnt/conf.d/server-id.cnf
          echo server-id=$((100 + $ordinal)) >> /mnt/conf-d/server-id.cnf
          cp /mnt/config-map/my.cnf /mnt/conf.d/
        volumeMounts:
        - name: conf
          mountPath: /mnt/conf.d
        - name: config-map
          mountPath: /mnt/config-map
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
      volumes:
      - name: conf
        emptyDir: {}
      - name: config-map
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
```

## Параметры манифеста

### spec.serviceName

**Обязательное поле** - имя Headless Service для стабильной сетевой идентичности подов.

### spec.replicas

Количество реплик. По умолчанию: 1

### spec.selector

Селектор для выбора подов, управляемых этим StatefulSet.

### spec.template

Шаблон для создания подов. Аналогично Deployment.

### spec.volumeClaimTemplates

Шаблоны для создания PVC для каждого пода:

```yaml
volumeClaimTemplates:
- metadata:
    name: data
  spec:
    accessModes: [ "ReadWriteOnce" ]
    storageClassName: fast-ssd
    resources:
      requests:
        storage: 10Gi
```

Каждый под получит свой PVC с именем: `<volumeClaimTemplate-name>-<statefulset-name>-<ordinal>`

### spec.podManagementPolicy

Политика управления подами:

- **OrderedReady** (по умолчанию) - упорядоченное создание/удаление
- **Parallel** - параллельное создание/удаление

### spec.updateStrategy

Стратегия обновления:

```yaml
updateStrategy:
  type: RollingUpdate  # или OnDelete
  rollingUpdate:
    partition: 2  # Обновить только поды с индексом >= partition
```

## Headless Service

StatefulSet требует Headless Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None  # Headless Service
  selector:
    app: web
  ports:
  - port: 80
    name: web
```

**DNS имена подов:**
- `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- Или просто: `<pod-name>.<service-name>`

## Управление StatefulSet

### Создание

```bash
# Из файла
kubectl apply -f statefulset.yaml
```

### Просмотр

```bash
# Список StatefulSet
kubectl get statefulsets
kubectl get sts  # Краткая форма

# Детальная информация
kubectl describe statefulset <statefulset-name>

# Поды StatefulSet
kubectl get pods -l app=web
```

### Масштабирование

```bash
# Увеличить количество реплик
kubectl scale statefulset <statefulset-name> --replicas=5

# Уменьшить количество реплик
kubectl scale statefulset <statefulset-name> --replicas=2
```

**Примечание:** Поды удаляются в обратном порядке (от большего индекса к меньшему).

### Обновление

```bash
# Обновить образ
kubectl set image statefulset/<statefulset-name> <container-name>=new-image:v2.0

# Обновить с partition (постепенное обновление)
kubectl patch statefulset <statefulset-name> -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":2}}}}'
```

### Удаление

```bash
# Удалить StatefulSet (поды удалятся, но PVC останутся)
kubectl delete statefulset <statefulset-name>

# Удалить StatefulSet и PVC
kubectl delete statefulset <statefulset-name>
kubectl delete pvc -l app=web  # Удалить связанные PVC
```

## Именование подов

Поды получают имена по шаблону: `<statefulset-name>-<ordinal>`

Например, для StatefulSet `web` с 3 репликами:
- `web-0`
- `web-1`
- `web-2`

## Стабильное хранилище

Каждый под получает свой PVC:
- `data-web-0`
- `data-web-1`
- `data-web-2`

PVC создаются автоматически при создании подов и удаляются при удалении подов (зависит от ReclaimPolicy PV).

## Best Practices

1. **Используйте для stateful приложений** - базы данных, очереди, кластеры
2. **Настройте Headless Service** - для стабильной сетевой идентичности
3. **Используйте volumeClaimTemplates** - для автоматического создания PVC
4. **Правильный accessMode** - обычно ReadWriteOnce для баз данных
5. **Мониторинг** - отслеживайте состояние каждого пода отдельно
6. **Backup** - регулярно делайте резервные копии данных каждого пода

## Ограничения

- Нельзя изменить volumeClaimTemplates после создания
- Удаление StatefulSet не удаляет PVC автоматически (зависит от ReclaimPolicy)
- Обновление медленнее, чем Deployment (упорядоченное)

## Связанные заметки

- [[Deployment]] - Для stateless приложений
- [[../Storage Resources/PersistentVolumeClaim]] - Стабильное хранилище
- [[../Networking/Service]] - Headless Service для StatefulSet

