---
created: 2025-01-27
tags:
  - kubernetes
  - k8s
  - problems
  - pvc
  - persistent-volume
  - mount-error
category: problems
---

# Ошибка монтирования Persistent Volume Claims

## Описание проблемы

При выполнении команды:

```bash
kubectl -n <namespace> describe pod <podName>
```

Возникают следующие ошибки:

```
Unable to attach or mount volumes: unmounted volumes=[postgresql-example-dep-2 stub-example-dep-2], unattached volumes=[application-properties postgresql-example-dep-2 stub-example-dep-2 kube-api-access-9khgb configuration data env jolokia-properties update-config]: timed out waiting for the condition
MountVolume.MountDevice failed for volume "pvc-e55b2447-5eb4-4784-86f1-10cc357568a7" : rpc error: code = Aborted desc = an operation with the given Volume ID 0001-0024-7af86dc7-561d-11ed-83bc-d7630632ac58-0000000000000003-05b82a81-d27e-11ef-8a25-061716ae0cf9 already exists
```

Эти ошибки указывают на то, что под не может стартовать из-за невозможности монтирования Persistent Volume Claims.

## Причина проблемы

Ошибка возникает из-за того, что kubelet не может корректно обработать уже существующие операции монтирования для указанного Volume ID. Это может быть связано с тем, что предыдущие попытки монтирования завершились с ошибкой, и kubelet не смог корректно очистить ресурсы.

## Решение проблемы

### Шаг 1: Определение ноды, на которой находится под

Найдите ноду, на которой запущен проблемный под, с помощью команды:

```bash
kubectl -n <namespace> get pod <podName> -o wide
```

### Шаг 2: Проверка логов kubelet

1. Подключитесь к ноде и проверьте логи kubelet:

```bash
docker logs -n 1000 -f kubelet
```

2. Найдите в логах строки вида:

```
"There were many similar errors. Turn up verbosity to see them." err="orphaned pod \"bd788c54-5b0b-46e9-865e-fd37c3843d5d\" found, but error not a directory occurred when trying to remove the volumes dir" numErrs=1
```

### Шаг 3: Удаление орфанных папок

1. Перейдите в директорию `/var/lib/kubelet/pods` на ноде и удалите папку, соответствующую UUID орфанного пода (например, `bd788c54-5b0b-46e9-865e-fd37c3843d5d`):

```bash
cd /var/lib/kubelet/pods
rm -rf bd788c54-5b0b-46e9-865e-fd37c3843d5d
```

2. Если таких папок несколько, удалите их все.

**⚠️ Внимание:** Убедитесь, что удаляете папки, соответствующие действительно орфанным подам, а не активным. Проверьте статус пода перед удалением:

```bash
kubectl -n <namespace> get pod <podName>
```

### Шаг 4: Перезапуск пода

После удаления орфанных папок перезапустите под:

```bash
kubectl -n <namespace> delete pod <podName>
```

Под будет автоматически пересоздан контроллером (Deployment, StatefulSet и т.п.), и монтирование должно пройти успешно.

## Альтернативные решения

### Проверка статуса PVC

Перед выполнением вышеуказанных шагов, проверьте статус PersistentVolumeClaim:

```bash
kubectl -n <namespace> get pvc
kubectl -n <namespace> describe pvc <pvc-name>
```

### Проверка статуса PV

Если проблема сохраняется, проверьте статус PersistentVolume:

```bash
kubectl get pv <pv-name>
kubectl describe pv <pv-name>
```

### Полное удаление пода с очисткой

Если проблема не решается, можно попробовать удалить под с принудительным завершением:

```bash
kubectl -n <namespace> delete pod <podName> --grace-period=0 --force
```

Затем подождите несколько секунд и проверьте, был ли под пересоздан.

## Примечания

- Данная проблема может возникать на версии Kubernetes 1.22 и связана с известными багами, описанными в следующих issues:
  - [Kubernetes Issue #105536](https://github.com/kubernetes/kubernetes/issues/105536)
  - [Kubernetes Issue #60987](https://github.com/kubernetes/kubernetes/issues/60987)

- Проблема также может возникать при использовании определенных CSI драйверов (Container Storage Interface)

- В некоторых случаях может потребоваться перезапуск kubelet на ноде:

```bash
# На ноде
systemctl restart kubelet
# или, если kubelet запущен в Docker
docker restart kubelet
```

## Предотвращение проблемы

Для предотвращения подобных проблем в будущем:

1. **Регулярный мониторинг:** Следите за состоянием подов и их логами
2. **Обновление Kubernetes:** Обновите кластер до более новой версии, если используется 1.22
3. **Правильная настройка CSI драйверов:** Убедитесь, что CSI драйверы настроены корректно и обновлены до последней версии
4. **Мониторинг нод:** Регулярно проверяйте логи kubelet на наличие подобных ошибок

## Связанные заметки

- [[../resources/Storage Resources/PersistentVolume]] - Информация о PersistentVolume
- [[../resources/Storage Resources/PersistentVolumeClaim]] - Информация о PersistentVolumeClaim
- [[../resources/Workload Resources/Pod]] - Работа с подами
- [[../kubectl Commands]] - Полезные команды kubectl

