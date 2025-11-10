---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - persistentvolumeclaim
  - storage
category: infrastructure
---

# PersistentVolumeClaim (PVC)

## Обзор

PersistentVolumeClaim (PVC) - это запрос на хранилище от пользователя. PVC запрашивает определенное количество хранилища с определенными режимами доступа.

## Пример манифеста

### Базовый PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-app-data
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### PVC с StorageClass

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-fast-storage
spec:
  storageClassName: fast-ssd  # Указать StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

### PVC без StorageClass

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-static
spec:
  storageClassName: ""  # Пустая строка = использовать статический PV
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### PVC для ReadWriteMany

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-shared
spec:
  accessModes:
    - ReadWriteMany  # Для нескольких подов одновременно
  resources:
    requests:
      storage: 100Gi
  storageClassName: nfs-storage
```

## Параметры манифеста

### spec.accessModes

Режимы доступа (должны соответствовать PV):

- **ReadWriteOnce** - монтирование одним подом
- **ReadOnlyMany** - монтирование несколькими подами только для чтения
- **ReadWriteMany** - монтирование несколькими подами для чтения/записи

### spec.resources.requests.storage

Запрашиваемый объем хранилища:
- `10Gi`, `100Mi`, `1Ti` и т.д.

### spec.storageClassName

- Указать имя StorageClass для динамического создания
- Пустая строка `""` - использовать статический PV без StorageClass
- Не указано - использовать default StorageClass кластера

### spec.volumeName

Опционально: указать конкретный PV для связывания.

### spec.selector

Селектор для выбора конкретного PV:
```yaml
selector:
  matchLabels:
    tier: database
  matchExpressions:
    - {key: environment, operator: In, values: [production]}
```

## Использование в Pod

### Монтирование PVC в Pod

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
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: pvc-app-data
```

### Использование в Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        volumeMounts:
        - name: data-volume
          mountPath: /data
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: pvc-app-data  # Один PVC для всех реплик (RWO)
```

## Управление PVC

### Создание

```bash
# Из файла
kubectl apply -f pvc.yaml

# Импертивно
kubectl create pvc pvc-app-data \
  --storage-class=fast-ssd \
  --access-mode=ReadWriteOnce \
  --storage=10Gi
```

### Просмотр

```bash
# Список PVC
kubectl get pvc

# Детальная информация
kubectl describe pvc <pvc-name>

# Статусы:
# Pending - ожидает связывания с PV
# Bound - связан с PV
# Lost - связанный PV не найден
```

### Удаление

```bash
# Удалить PVC
kubectl delete pvc <pvc-name>

# При ReclaimPolicy: Delete, PV также будет удален
# При ReclaimPolicy: Retain, PV останется в статусе Released
```

## Расширение PVC

### Изменение размера (если поддерживается)

```bash
# Отредактировать манифест
kubectl edit pvc <pvc-name>
# Изменить spec.resources.requests.storage на больший размер

# Или напрямую
kubectl patch pvc <pvc-name> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
```

**Требования:**
- StorageClass должен поддерживать расширение (`allowVolumeExpansion: true`)
- PVC должен быть в статусе Bound
- Можно только увеличивать размер

## Обновление PVC

### Изменение StorageClass

```yaml
# НЕ МОЖНО изменить storageClassName существующего PVC
# Нужно создать новый PVC и мигрировать данные
```

### Миграция данных

1. Создать новый PVC
2. Скопировать данные из старого PVC
3. Обновить поды для использования нового PVC
4. Удалить старый PVC

## Best Practices

1. **Используйте StorageClass** - для динамического создания
2. **Правильный accessMode** - RWO для одного пода, RWX для нескольких
3. **Размер хранилища** - запрашивайте достаточный размер, расширение не всегда возможно
4. **Один PVC на приложение** - для изоляции данных
5. **Backup** - регулярно делайте резервные копии важных данных

## Ограничения

- Нельзя уменьшить размер PVC
- Нельзя изменить accessMode после создания
- Нельзя изменить storageClassName после создания
- Один PVC (RWO) может быть смонтирован только одним подом одновременно

## Связанные заметки

- [[PersistentVolume]] - Статические тома
- [[../Workload Resources/Pod]] - Использование PVC в подах
- [[../Workload Resources/StatefulSet]] - Использование PVC в StatefulSet
- [[../../Problems/PVC Mount Error]] - Решение проблемы монтирования Persistent Volume Claims

