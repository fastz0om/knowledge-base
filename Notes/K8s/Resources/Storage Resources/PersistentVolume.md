---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - persistentvolume
  - storage
category: infrastructure
---

# PersistentVolume (PV)

## Обзор

PersistentVolume (PV) - это ресурс хранилища в кластере, который был подготовлен администратором или динамически создан с помощью StorageClass.

## Типы PV

- **Статический** - создается администратором заранее
- **Динамический** - создается автоматически через StorageClass

## Пример манифеста

### Статический PV (HostPath)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv
    type: DirectoryOrCreate
```

### PV с NFS

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: nfs-server.example.com
    path: /exports/data
  persistentVolumeReclaimPolicy: Retain
```

### PV с AWS EBS

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ebs
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-12345678
    fsType: ext4
  persistentVolumeReclaimPolicy: Delete
```

### PV с StorageClass

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-gce
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  gcePersistentDisk:
    pdName: my-disk
    fsType: ext4
```

## Параметры манифеста

### spec.capacity

Объем хранилища:
```yaml
capacity:
  storage: 10Gi  # 10Gi, 100Mi, 1Ti и т.д.
```

### spec.accessModes

Режимы доступа:

- **ReadWriteOnce (RWO)** - монтирование одним узлом для чтения/записи
- **ReadOnlyMany (ROX)** - монтирование несколькими узлами только для чтения
- **ReadWriteMany (RWX)** - монтирование несколькими узлами для чтения/записи

### spec.persistentVolumeReclaimPolicy

Политика освобождения:

- **Retain** - сохранить данные после освобождения PVC (ручная очистка)
- **Recycle** - устарело, использовать Delete
- **Delete** - автоматически удалить данные и PV

### spec.storageClassName

Имя StorageClass. PV без storageClassName могут быть связаны только с PVC без storageClassName.

### spec.mountOptions

Опции монтирования:
```yaml
mountOptions:
  - nfsvers=4.1
  - noatime
```

## Типы томов

### hostPath

Локальное хранилище узла (только для разработки):
```yaml
hostPath:
  path: /data/pv
  type: DirectoryOrCreate
```

**Типы:**
- `DirectoryOrCreate` - создать если не существует
- `Directory` - должна существовать
- `FileOrCreate` - файл, создать если не существует
- `File` - файл должен существовать

### nfs

Network File System:
```yaml
nfs:
  server: nfs-server.example.com
  path: /exports/data
  readOnly: false
```

### awsElasticBlockStore

AWS EBS том:
```yaml
awsElasticBlockStore:
  volumeID: vol-12345678
  fsType: ext4
```

### gcePersistentDisk

Google Cloud Persistent Disk:
```yaml
gcePersistentDisk:
  pdName: my-disk
  fsType: ext4
```

### azureDisk

Azure Disk:
```yaml
azureDisk:
  diskName: my-disk
  diskURI: https://storageaccount.blob.core.windows.net/vhds/my-disk.vhd
```

## Управление PV

### Создание

```bash
# Из файла
kubectl apply -f pv.yaml
```

### Просмотр

```bash
# Список PV
kubectl get pv

# Детальная информация
kubectl describe pv <pv-name>

# Статусы:
# Available - доступен для использования
# Bound - связан с PVC
# Released - PVC удален, но данные еще не удалены
# Failed - ошибка при освобождении
```

### Удаление

```bash
# Удалить PV
kubectl delete pv <pv-name>

# При Retain политике нужно сначала удалить PVC
```

## Связь с PVC

PV связывается с PersistentVolumeClaim (PVC) по:
- Storage capacity
- Access modes
- StorageClass (если указан)

## Best Practices

1. **Используйте StorageClass** - для динамического создания PV
2. **Выбирайте правильный accessMode** - в зависимости от приложения
3. **Политика ReclaimPolicy** - Retain для важных данных
4. **Избегайте hostPath в production** - используйте сетевые хранилища
5. **Мониторинг** - отслеживайте использование хранилища

## Связанные заметки

- [[PersistentVolumeClaim]] - Запросы на хранилище
- [[../Workload Resources/Pod]] - Использование PV в подах

