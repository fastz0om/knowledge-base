---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - kubectl
  - commands
category: infrastructure
---

# kubectl - Полезные команды

## Обзор

Справочник полезных команд `kubectl` для работы с Kubernetes.

## Получение информации

### Информация о кластере

```bash
# Информация о кластере
kubectl cluster-info

# Версия кластера
kubectl version

# Контексты и кластеры
kubectl config get-contexts

# Текущий контекст
kubectl config current-context

# Переключение контекста
kubectl config use-context <context-name>
```

### Просмотр ресурсов

```bash
# Все ресурсы в namespace
kubectl get all -n <namespace>

# Все ресурсы во всех namespace
kubectl get all --all-namespaces

# Конкретный ресурс
kubectl get pods
kubectl get deployments
kubectl get services

# С широким выводом
kubectl get pods -o wide

# В формате YAML
kubectl get pod <pod-name> -o yaml

# В формате JSON
kubectl get pod <pod-name> -o json

# Описание ресурса
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
```

## Применение манифестов

```bash
# Применить манифест
kubectl apply -f deployment.yaml

# Применить из каталога
kubectl apply -f ./manifests/

# Создать (без обновления)
kubectl create -f deployment.yaml

# Удалить
kubectl delete -f deployment.yaml

# Применить с сухим прогоном
kubectl apply -f deployment.yaml --dry-run=client -o yaml

# Diff перед применением
kubectl diff -f deployment.yaml
```

## Управление подами

```bash
# Логи пода
kubectl logs <pod-name>

# Логи с несколькими контейнерами
kubectl logs <pod-name> -c <container-name>

# Логи с потоком
kubectl logs -f <pod-name>

# Логи предыдущего контейнера
kubectl logs <pod-name> --previous

# Выполнение команды в поде
kubectl exec -it <pod-name> -- /bin/bash

# Выполнение в конкретном контейнере
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# Копирование файлов
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

## Масштабирование

```bash
# Масштабирование deployment
kubectl scale deployment <deployment-name> --replicas=3

# Автомасштабирование
kubectl autoscale deployment <deployment-name> --min=2 --max=10 --cpu-percent=80
```

## Обновления и откаты

```bash
# Обновление образа
kubectl set image deployment/<deployment-name> <container-name>=<new-image>:<tag>

# История изменений
kubectl rollout history deployment/<deployment-name>

# Откат к предыдущей версии
kubectl rollout undo deployment/<deployment-name>

# Откат к конкретной версии
kubectl rollout undo deployment/<deployment-name> --to-revision=2

# Статус rollout
kubectl rollout status deployment/<deployment-name>

# Пауза rollout
kubectl rollout pause deployment/<deployment-name>

# Возобновление rollout
kubectl rollout resume deployment/<deployment-name>
```

## Удаление ресурсов

```bash
# Удалить pod
kubectl delete pod <pod-name>

# Удалить deployment
kubectl delete deployment <deployment-name>

# Удалить по меткам
kubectl delete pods -l app=myapp

# Удалить все в namespace
kubectl delete all --all -n <namespace>

# Принудительное удаление
kubectl delete pod <pod-name> --grace-period=0 --force

# Удалить все evicted поды в namespace
kubectl get pod -n <namespace> | grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n <namespace>
```

## Порт-форвардинг

```bash
# Проброс порта
kubectl port-forward <pod-name> 8080:80

# Проброс для service
kubectl port-forward service/<service-name> 8080:80

# Проброс для deployment
kubectl port-forward deployment/<deployment-name> 8080:80
```

## Работа с namespace

```bash
# Создать namespace
kubectl create namespace <namespace-name>

# Просмотр namespace
kubectl get namespaces

# Удалить namespace
kubectl delete namespace <namespace-name>

# Установить namespace по умолчанию
kubectl config set-context --current --namespace=<namespace-name>
```

## Отладка и диагностика

```bash
# События
kubectl get events --sort-by='.lastTimestamp'

# События конкретного ресурса
kubectl get events --field-selector involvedObject.name=<pod-name>

# Топ по ресурсам (требует metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pod <pod-name> --containers

# Проверка доступа к API
kubectl auth can-i create pods
kubectl auth can-i delete pods --namespace <namespace>
```

## Полезные фильтры и форматы

```bash
# Фильтрация по меткам
kubectl get pods -l app=myapp

# Фильтрация по нескольким меткам
kubectl get pods -l app=myapp,env=prod

# JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Вывод в таблицу
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Сортировка
kubectl get pods --sort-by='.metadata.creationTimestamp'
```

## Batch операции

```bash
# Применить к нескольким ресурсам
kubectl get pods -o name | xargs -I {} kubectl delete {}

# Обновить образ во всех deployment
kubectl get deployments -o name | xargs -I {} kubectl set image {} <container>=<new-image>
```

## Связанные заметки

- [[Overview]] - Общая информация о Kubernetes
- [[resources/Workload Resources/Pod]] - Работа с подами
- [[resources/Workload Resources/Deployment]] - Управление развертываниями

