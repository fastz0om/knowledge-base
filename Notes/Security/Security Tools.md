---
created: 2025-01-27
tags: [security, trivy, vulnerability-scanning, containers]
category: security
---

# Инструменты безопасности

## Обзор

Инструменты сканирования безопасности для файловых систем и образов контейнеров.

## Сканер уязвимостей Trivy

Trivy - это комплексный сканер безопасности, который обнаруживает уязвимости в файловых системах, образах контейнеров и многом другом.

### Сканирование файловой системы

Сканирование локального каталога на уязвимости:

```bash
trivy fs \
  --cache-dir /opt/trivy-cache \
  --no-progress \
  --db-repository nexus.example.local/ghcr.io/aquasecurity/trivy-db \
  -f json \
  -o result.json \
  --timeout 25m0s \
  .
```

**Опции:**
- `--cache-dir` - Пользовательский каталог кэша
- `--no-progress` - Отключить индикатор прогресса (полезно для CI/CD)
- `--db-repository` - Пользовательский репозиторий базы данных уязвимостей
- `-f json` - Формат вывода (json, table, sarif)
- `-o` - Файл вывода
- `--timeout` - Максимальная длительность сканирования
- `.` - Целевой каталог (текущий каталог)

### Сканирование корневой файловой системы

Сканирование всей корневой файловой системы:

```bash
trivy rootfs \
  --cache-dir /opt/trivy-cache \
  --no-progress \
  --db-repository nexus.example.local/ghcr.io/aquasecurity/trivy-db \
  -f json \
  -o result.json \
  --timeout 25m0s \
  /
```

**Предупреждение:** Сканирование корневой файловой системы может занять много времени и может потребовать прав root.

### Дополнительные типы сканирования

```bash
# Сканирование образа контейнера
trivy image nginx:latest

# Сканирование репозитория
trivy repo https://github.com/user/repo.git

# Сканирование кластера Kubernetes
trivy k8s cluster

# Сканирование конкретного ресурса Kubernetes
trivy k8s deployment my-app -n production
```

## Форматы вывода

```bash
# JSON (для автоматизации)
trivy fs -f json -o result.json .

# Таблица (читаемый формат)
trivy fs -f table .

# SARIF (для GitHub Security)
trivy fs -f sarif -o result.sarif .
```

## Интеграция

### Пример Jenkins Pipeline

```groovy
stage('Security Scan') {
    steps {
        sh '''
            trivy fs \
                --cache-dir /opt/trivy-cache \
                --no-progress \
                --db-repository nexus.example.local/ghcr.io/aquasecurity/trivy-db \
                -f json \
                -o trivy-report.json \
                --exit-code 0 \
                --severity HIGH,CRITICAL \
                .
        '''
        archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
    }
}
```

## Рекомендации

1. **Кэш базы данных** - Используйте общий каталог кэша в CI/CD
2. **Пользовательский репозиторий** - Укажите на внутреннее зеркало для более быстрого сканирования
3. **Таймаут** - Установите соответствующий таймаут для больших сканирований
4. **Коды выхода** - Используйте `--exit-code` для контроля поведения CI/CD
5. **Фильтрация по серьезности** - Сосредоточьтесь на уязвимостях HIGH и CRITICAL
6. **Регулярное сканирование** - Запланируйте регулярное сканирование в CI/CD pipeline

## Связанные заметки

- [[../Docker/Container Operations]]
- [[../Jenkins/Pipeline Scripts]]
- [[../Jenkins/Grype Configuration]] - Конфигурация Grype для сканирования на Jenkins агентах
- [[Passbolt/Passbolt to KeePassXC Export]] - Импорт паролей из Passbolt в KeePassXC

