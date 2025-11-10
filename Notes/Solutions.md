# Решения

Коллекция решений типовых проблем.

## Решения для инфраструктуры

- [[Linux/Kernel Management]] - Как устанавливать, удалять и управлять ядрами
- [[Linux/Package Management]] - Сравнение версий пакетов, офлайн установка
- [[Linux/System Administration]] - Изменение размера LVM, SSH ключи, GPG, настройка NTP
- [[Docker/Container Operations]] - Управление жизненным циклом контейнеров
- [[Proxmox/Backup and Restore]] - Операции с резервным копированием Proxmox
- [[Databases/ClickHouse]] - Работа с ClickHouse
- [[Vault/AppRole Management]] - Операции с Vault
- **Kubernetes:**
  - [[K8s/Overview]] - Обзор Kubernetes
  - [[K8s/kubectl Commands]] - Команды для работы с кластером
  - Работа с ресурсами: [[K8s/Resources/Workload Resources/Pod]], [[K8s/Resources/Workload Resources/Pod Template Image Pull Policy]], [[K8s/Resources/Workload Resources/Deployment]], [[K8s/Resources/Networking/Service]], [[K8s/Resources/Networking/Ingress]]
  - Конфигурация: [[K8s/Resources/Configuration Resources/ConfigMap]], [[K8s/Resources/Configuration Resources/Secret]]
  - Хранилище: [[K8s/Resources/Storage Resources/PersistentVolumeClaim]]
  - Безопасность: [[K8s/Resources/RBAC/ServiceAccount]], [[K8s/Resources/RBAC/Role]]
  - Проблемы: [[K8s/Solutions/PVC Mount Error]] - Решение проблемы монтирования Persistent Volume Claims

## Решения для автоматизации

- **Jenkins:**
  - [[Jenkins/Pipeline Scripts]] - Управление контейнерами, версионирование, сброс задач
  - [[Jenkins/Git Timeout Fix]] - Исправление git timeout в MultiBranch Pipeline
  - [[Jenkins/Reset Build Number]] - Сброс индекса номера сборки на определенное значение
  - [[Jenkins/Active Choice Parameter Logging]] - Настройка логгирования Active Choice Parameter для отладки
  - [[Jenkins/Using Credentials in Active Choice Parameter]] - Динамические списки в Active Choice Parameter с использованием секретов Jenkins
  - [[Jenkins/Nexus Component Download URL]] - Получение ссылок на скачивание компонентов из Nexus через Search API
  - [[Jenkins/Exclude Infrastructure Changes from ChangeSets]] - Исключение изменений инфраструктурного репозитория из ChangeSets (Pre-stage и Post-actions методы)
  - [[Jenkins/Role Strategy Roles and Users]] - Получение списка ролей и назначенных пользователей в Role Strategy через Script Console
  - [[Jenkins/Grype Configuration]] - Установка и настройка Grype для сканирования уязвимостей на Jenkins агентах
  - [[Jenkins/Confluence Publisher Plugin Integration]] - Настройка пользовательских макросов в Confluence и интеграция с Jenkins для автоматической публикации артефактов и обновления страниц

## Решения для безопасности

- [[Security/Security Tools]] - Сканирование уязвимостей с помощью Trivy
