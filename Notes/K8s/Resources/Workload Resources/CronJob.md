---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - cronjob
  - workload
category: infrastructure
---

# CronJob

## Обзор

CronJob создает Job по расписанию, используя формат cron. Полезен для периодических задач: резервное копирование, очистка данных, отправка отчетов.

## Формат расписания

```
┌───────────── минута (0 - 59)
│ ┌───────────── час (0 - 23)
│ │ ┌───────────── день месяца (1 - 31)
│ │ │ ┌───────────── месяц (1 - 12)
│ │ │ │ ┌───────────── день недели (0 - 6) (воскресенье = 0 или 7)
│ │ │ │ │
* * * * *
```

## Пример манифеста

### Базовый CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Каждый день в 2:00
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup:latest
            command: ["/backup.sh"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

### CronJob с активным дедлайном

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-job
spec:
  schedule: "0 */6 * * *"  # Каждые 6 часов
  startingDeadlineSeconds: 200  # Максимум 200 секунд на запуск
  concurrencyPolicy: Forbid  # Не запускать параллельно
  jobTemplate:
    spec:
      activeDeadlineSeconds: 600  # Максимум 10 минут на выполнение
      template:
        spec:
          containers:
          - name: cleanup
            image: cleanup:latest
            command: ["/cleanup.sh", "--older-than", "7d"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 3
```

### CronJob с timezone

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report-job
spec:
  schedule: "0 9 * * 1"  # Каждый понедельник в 9:00
  timeZone: "Europe/Moscow"  # Часовой пояс (Kubernetes 1.24+)
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: report
            image: report-generator:latest
          restartPolicy: Never
```

## Параметры манифеста

### spec.schedule

Расписание в формате cron (обязательно):

```yaml
schedule: "0 2 * * *"  # Каждый день в 2:00
schedule: "*/5 * * * *"  # Каждые 5 минут
schedule: "0 0 * * 0"  # Каждое воскресенье в полночь
schedule: "0 9-17 * * 1-5"  # С понедельника по пятницу с 9 до 17
```

### spec.timeZone

Часовой пояс для расписания (Kubernetes 1.24+). Используйте названия из базы данных tz.

### spec.startingDeadlineSeconds

Максимальное время в секундах, на которое запуск Job может быть отложен. Если пропущено - пропускается.

### spec.concurrencyPolicy

Политика параллельного выполнения:

- **Allow** (по умолчанию) - разрешить параллельные Job
- **Forbid** - запретить параллельные Job
- **Replace** - отменить текущую Job и создать новую

### spec.suspend

Приостановить CronJob:
```yaml
suspend: true  # Не создавать новые Job
```

### spec.successfulJobsHistoryLimit

Количество успешных Job для хранения. По умолчанию: 3

### spec.failedJobsHistoryLimit

Количество неудачных Job для хранения. По умолчанию: 1

### spec.jobTemplate

Шаблон для создания Job. Аналогично обычному Job манифесту.

## Примеры расписаний

```yaml
# Каждую минуту
schedule: "*/1 * * * *"

# Каждый час
schedule: "0 * * * *"

# Каждый день в полночь
schedule: "0 0 * * *"

# Каждый день в 2:30
schedule: "30 2 * * *"

# Каждое воскресенье в 3:00
schedule: "0 3 * * 0"

# С понедельника по пятницу в 9:00
schedule: "0 9 * * 1-5"

# Каждые 15 минут
schedule: "*/15 * * * *"

# Первое число каждого месяца в полночь
schedule: "0 0 1 * *"

# Каждый час в рабочие дни
schedule: "0 * * * 1-5"
```

## Управление CronJob

### Создание

```bash
# Из файла
kubectl apply -f cronjob.yaml
```

### Просмотр

```bash
# Список CronJob
kubectl get cronjobs
kubectl get cj  # Краткая форма

# Детальная информация
kubectl describe cronjob <cronjob-name>

# Созданные Job
kubectl get jobs -l app=myapp

# Последняя выполненная Job
kubectl get jobs --sort-by=.metadata.creationTimestamp
```

### Приостановка

```bash
# Приостановить CronJob
kubectl patch cronjob <cronjob-name> -p '{"spec":{"suspend":true}}'

# Возобновить CronJob
kubectl patch cronjob <cronjob-name> -p '{"spec":{"suspend":false}}'
```

### Запуск вручную

```bash
# Создать Job из CronJob вручную
kubectl create job --from=cronjob/<cronjob-name> manual-job-$(date +%s)
```

### Удаление

```bash
# Удалить CronJob (Job останутся)
kubectl delete cronjob <cronjob-name>

# Удалить CronJob и связанные Job
kubectl delete cronjob <cronjob-name>
kubectl delete jobs -l app=<app-label>
```

## Обработка пропущенных запусков

### startingDeadlineSeconds

Если пропущен запуск, он будет выполнен в следующий запланированный раз:

```yaml
spec:
  schedule: "0 2 * * *"
  startingDeadlineSeconds: 300  # Максимум 5 минут опоздания
```

Если запуск пропущен более чем на `startingDeadlineSeconds`, он не будет выполнен.

## Best Practices

1. **Используйте осмысленные расписания** - проверяйте формат cron
2. **Настройте concurrencyPolicy** - Forbid для критических задач
3. **Ограничивайте историю** - настройте successfulJobsHistoryLimit и failedJobsHistoryLimit
4. **Используйте activeDeadlineSeconds** - для предотвращения зависаний Job
5. **Мониторинг** - отслеживайте успешность выполнения CronJob
6. **Тестирование** - сначала запускайте Job вручную перед настройкой расписания

## Ограничения

- CronJob работает только в одном часовом поясе кластера
- Нет гарантии точного времени запуска (зависит от доступности ресурсов)
- Пропущенные запуски не восстанавливаются автоматически

## Связанные заметки

- [[Job]] - Одноразовые задачи
- [[Pod]] - Базовый ресурс
- [[../../Overview]] - Общая информация о Kubernetes

