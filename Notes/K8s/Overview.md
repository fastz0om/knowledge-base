---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - architecture
  - overview
category: infrastructure
---

# Kubernetes - Обзор

## Что такое Kubernetes

Kubernetes (K8s) - это платформа с открытым исходным кодом для автоматизации развертывания, масштабирования и управления контейнеризированными приложениями.

## Основные концепции

### Cluster (Кластер)

Кластер состоит из:
- **Master Node (Control Plane)** - управляющие узлы
- **Worker Nodes** - рабочие узлы для выполнения подов

### Control Plane (Плоскость управления)

Компоненты Control Plane:

1. **kube-apiserver** - API сервер, точка входа в кластер
2. **etcd** - распределенное хранилище конфигурации
3. **kube-scheduler** - планировщик, назначает поды на узлы
4. **kube-controller-manager** - контроллеры, следят за состоянием кластера
5. **cloud-controller-manager** - интеграция с облачными провайдерами

### Worker Nodes (Рабочие узлы)

Компоненты узлов:

1. **kubelet** - агент, который запускает поды
2. **kube-proxy** - сетевой прокси для правил доступа
3. **Container Runtime** - Docker, containerd, CRI-O

## Основные ресурсы Kubernetes

### Workload Resources (Ресурсы рабочей нагрузки)

- **Pod** - наименьшая единица развертывания
- **Deployment** - управление репликами подов
- **StatefulSet** - управление stateful приложениями
- **DaemonSet** - под на каждом узле
- **Job** - одноразовая задача
- **CronJob** - периодические задачи

### Service Resources (Сервисы)

- **Service** - стабильный сетевой endpoint для подов
- **Ingress** - управление входящим трафиком
- **Endpoint** - список IP адресов подов

### Configuration Resources (Конфигурация)

- **ConfigMap** - конфигурационные данные
- **Secret** - секретные данные (пароли, ключи)
- **Namespace** - изоляция ресурсов

### Storage Resources (Хранилище)

- **PersistentVolume (PV)** - выделенное хранилище
- **PersistentVolumeClaim (PVC)** - запрос на хранилище
- **StorageClass** - классы хранилища

### RBAC Resources (Управление доступом)

- **ServiceAccount** - учетная запись для подов
- **Role** - права доступа в namespace
- **ClusterRole** - права доступа в кластере
- **RoleBinding** - привязка роли к субъекту
- **ClusterRoleBinding** - привязка роли кластера

## Основные команды управления

```bash
# Получить информацию о кластере
kubectl cluster-info

# Просмотр узлов
kubectl get nodes

# Описание узла
kubectl describe node <node-name>

# Просмотр всех ресурсов
kubectl get all --all-namespaces
```

## Работа с ресурсами

### Применение манифестов

```bash
# Создать ресурс из файла
kubectl apply -f deployment.yaml

# Создать из каталога
kubectl apply -f ./manifests/

# Удалить ресурс
kubectl delete -f deployment.yaml
```

### Просмотр ресурсов

```bash
# Просмотр всех подов
kubectl get pods

# Просмотр в namespace
kubectl get pods -n <namespace>

# Детальная информация
kubectl describe pod <pod-name>
```

## Полезные ссылки

- [Официальная документация Kubernetes](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)

## Связанные заметки

- [[kubectl Commands]] - Полезные команды kubectl
- [[resources/Workload Resources/Pod]] - Базовый ресурс Kubernetes
- [[resources/Workload Resources/Deployment]] - Управление развертываниями

