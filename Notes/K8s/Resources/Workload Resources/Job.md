---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - job
  - workload
category: infrastructure
---

# Job

## Обзор

Job создает один или несколько подов и гарантирует, что указанное количество подов успешно завершится. Полезен для одноразовых задач, обработки данных, миграций.

## Использование

- Одноразовые задачи
- Обработка данных
- Миграции баз данных
- Импорт/экспорт данных

## Пример манифеста

### Простой Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

### Job с completions

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 5  # Выполнить 5 раз
  parallelism: 2  # Параллельно 2 пода
  template:
    metadata:
      labels:
        app: batch-processor
    spec:
      containers:
      - name: processor
        image: processor:latest
        command: ["process", "--input", "/data/input", "--output", "/data/output"]
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
      restartPolicy: OnFailure
```

### Job с активным дедлайном

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: time-limited-job
spec:
  activeDeadlineSeconds: 300  # Максимум 5 минут
  template:
    spec:
      containers:
      - name: worker
        image: worker:latest
        command: ["/bin/sh", "-c", "sleep 600"]  # Будет прерван через 5 минут
      restartPolicy: Never
```

### Job с ручным очищением

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: manual-cleanup-job
spec:
  ttlSecondsAfterFinished: 100  # Удалить через 100 секунд после завершения
  template:
    spec:
      containers:
      - name: cleanup
        image: cleanup:latest
      restartPolicy: Never
```

## Параметры манифеста

### spec.completions

Количество успешных выполнений, необходимых для завершения Job. По умолчанию: 1

### spec.parallelism

Количество подов, которые могут выполняться параллельно. По умолчанию: 1

### spec.backoffLimit

Количество повторных попыток перед помечением Job как неудачной. По умолчанию: 6

### spec.activeDeadlineSeconds

Максимальное время выполнения Job в секундах. После истечения Job прерывается.

### spec.ttlSecondsAfterFinished

Автоматическое удаление Job через указанное количество секунд после завершения.

### spec.template.spec.restartPolicy

Политика перезапуска:
- **Never** - никогда не перезапускать (рекомендуется)
- **OnFailure** - перезапускать при ошибке

## Управление Job

### Создание

```bash
# Из файла
kubectl apply -f job.yaml

# Импертивно
kubectl create job my-job --image=busybox -- echo "Hello World"
```

### Просмотр

```bash
# Список Job
kubectl get jobs

# Детальная информация
kubectl describe job <job-name>

# Поды Job
kubectl get pods -l job-name=<job-name>

# Логи пода
kubectl logs <pod-name>
```

### Удаление

```bash
# Удалить Job (поды также удалятся)
kubectl delete job <job-name>

# Удалить все завершенные Job
kubectl delete jobs --field-selector status.successful=1
```

## Статусы Job

- **Active** - Job активна, есть активные поды
- **Succeeded** - Job успешно завершена
- **Failed** - Job завершилась с ошибкой

## Параллельные Job

### Параллельные независимые задачи

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 10  # Выполнить 10 задач
  parallelism: 3  # Параллельно 3 пода
  template:
    spec:
      containers:
      - name: worker
        image: worker:latest
        # Каждый под обрабатывает свою часть работы
      restartPolicy: Never
```

### Рабочая очередь

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: queue-worker
spec:
  parallelism: 5  # 5 воркеров обрабатывают очередь
  completions: null  # Неограниченное количество
  template:
    spec:
      containers:
      - name: worker
        image: worker:latest
        command: ["/worker", "--queue=my-queue"]
      restartPolicy: OnFailure
```

## Best Practices

1. **Используйте Never для restartPolicy** - для большинства случаев
2. **Настройте backoffLimit** - ограничьте количество повторных попыток
3. **Используйте activeDeadlineSeconds** - для предотвращения зависаний
4. **Настройте ttlSecondsAfterFinished** - для автоматической очистки
5. **Мониторинг** - отслеживайте статус выполнения Job

## Отличие от Pod

| Характеристика | Job | Pod |
|---------------|-----|-----|
| Перезапуск | Автоматический при ошибке | Зависит от restartPolicy |
| Завершение | Ожидается успешное завершение | Может работать бесконечно |
| Использование | Одноразовые задачи | Долгоживущие приложения |

## Связанные заметки

- [[CronJob]] - Для периодических задач
- [[Pod]] - Базовый ресурс
- [[../../Overview]] - Общая информация о Kubernetes

