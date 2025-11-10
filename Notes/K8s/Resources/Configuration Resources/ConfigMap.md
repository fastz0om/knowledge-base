---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - configmap
  - configuration
category: infrastructure
---

# ConfigMap

## Обзор

ConfigMap хранит конфигурационные данные в виде пар ключ-значение. Позволяет отделить конфигурацию от образов контейнеров.

## Использование

- Конфигурационные файлы
- Переменные окружения
- Параметры командной строки
- Любые данные, не являющиеся секретами

## Пример манифеста

### ConfigMap с данными

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Простые ключ-значение
  log-level: "info"
  database-url: "postgresql://localhost/db"
  
  # Многострочные данные
  config.yaml: |
    server:
      port: 8080
      host: localhost
    database:
      type: postgresql
      pool-size: 10
  
  # JSON конфигурация
  app.json: |
    {
      "app": {
        "name": "myapp",
        "version": "1.0.0"
      }
    }
```

### ConfigMap из файлов

```bash
# Создание из файла
kubectl create configmap app-config --from-file=config.properties

# Создание из каталога
kubectl create configmap app-config --from-file=./configs/

# Создание из литерала
kubectl create configmap app-config \
  --from-literal=log-level=info \
  --from-literal=database-url=postgresql://localhost/db
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
    - configMapRef:
        name: app-config
    # Или отдельные ключи
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log-level
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
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      # Опционально: выбрать конкретные ключи
      items:
      - key: config.yaml
        path: config.yaml
      - key: app.json
        path: app.json
```

### Частичное монтирование

```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config/config.yaml
      subPath: config.yaml
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

## Параметры манифеста

### metadata

- **name** - имя ConfigMap (обязательно)
- **namespace** - namespace для ConfigMap

### data

Простые строковые ключ-значение пары.

### binaryData

Бинарные данные в base64 формате:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: binary-config
binaryData:
  binary-file: <base64-encoded-data>
```

## Управление ConfigMap

### Создание

```bash
# Из файла
kubectl apply -f configmap.yaml

# Импертивно из файла
kubectl create configmap app-config --from-file=config.properties

# Импертивно из литералов
kubectl create configmap app-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
```

### Просмотр

```bash
# Список ConfigMap
kubectl get configmaps

# Детальная информация
kubectl describe configmap <configmap-name>

# Данные ConfigMap
kubectl get configmap <configmap-name> -o yaml
```

### Редактирование

```bash
# Интерактивное редактирование
kubectl edit configmap <configmap-name>
```

### Удаление

```bash
kubectl delete configmap <configmap-name>
```

## Ограничения

- Размер до 1 MiB для data
- Размер до 1 MiB для binaryData
- Всего до 256 KiB для metadata.annotations

## Обновление ConfigMap

### Автоматическое обновление

При монтировании как том, изменения применяются автоматически, но может потребоваться перезапуск подов для применения изменений.

### Принудительный перезапуск

```bash
# Перезапуск deployment после изменения ConfigMap
kubectl rollout restart deployment/<deployment-name>
```

## Immutable ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
immutable: true  # Запрещает изменения после создания
data:
  config: "value"
```

## Best Practices

1. **Не храните секреты** - используйте Secret для чувствительных данных
2. **Именуйте осмысленно** - понятные имена для ConfigMap
3. **Группируйте логически** - объединяйте связанные конфигурации
4. **Используйте immutable** - для стабильных конфигураций
5. **Версионируйте изменения** - отслеживайте изменения конфигурации

## Связанные заметки

- [[Secret]] - Хранение секретных данных
- [[../Workload Resources/Pod]] - Использование ConfigMap в подах
- [[../Workload Resources/Deployment]] - Использование ConfigMap в deployment

