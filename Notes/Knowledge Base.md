# База знаний

Общие знания и справочные материалы.

## Инфраструктура

### Linux
- [[Linux/Kernel Management]] - Управление ядрами на Ubuntu, Debian, CentOS, RedOS
- [[Linux/Package Management]] - Операции с пакетами, сравнение версий, офлайн установка
- [[Linux/System Administration]] - LVM, SSH, GPG, NTP, операции с Git

### Контейнеры и виртуализация
- [[Docker/Container Operations]] - Управление контейнерами и интеграция с Jenkins
- [[Proxmox/Backup and Restore]] - Резервное копирование и восстановление VM

### Kubernetes
- [[K8s/Overview]] - Обзор Kubernetes и архитектура
- [[K8s/kubectl Commands]] - Полезные команды kubectl
- **Workload Resources:** [[K8s/Resources/Workload Resources/Pod]], [[K8s/Resources/Workload Resources/Pod Template Image Pull Policy]], [[K8s/Resources/Workload Resources/Deployment]], [[K8s/Resources/Workload Resources/StatefulSet]], [[K8s/Resources/Workload Resources/DaemonSet]], [[K8s/Resources/Workload Resources/Job]], [[K8s/Resources/Workload Resources/CronJob]]
- **Networking:** [[K8s/Resources/Networking/Service]], [[K8s/Resources/Networking/Ingress]]
- **Configuration:** [[K8s/Resources/Configuration Resources/ConfigMap]], [[K8s/Resources/Configuration Resources/Secret]], [[K8s/Resources/Configuration Resources/Namespace]]
- **Storage:** [[K8s/Resources/Storage Resources/PersistentVolume]], [[K8s/Resources/Storage Resources/PersistentVolumeClaim]]
- **RBAC:** [[K8s/Resources/RBAC/ServiceAccount]], [[K8s/Resources/RBAC/Role]], [[K8s/Resources/RBAC/RoleBinding]]

## Автоматизация

### CI/CD
- **Jenkins:**
  - [[Jenkins/Pipeline Scripts]] - Groovy скрипты для pipelines, версионирование, шаблонизация
  - [[Jenkins/Git Timeout Fix]] - Решение проблемы git timeout в MultiBranch Pipeline
  - [[Jenkins/Reset Build Number]] - Сброс индекса номера сборки на определенное значение
  - [[Jenkins/Active Choice Parameter Logging]] - Настройка логгирования для плагина Active Choice Parameter
  - [[Jenkins/Using Credentials in Active Choice Parameter]] - Использование секретов Jenkins для динамических списков в Active Choice Parameter
  - [[Jenkins/Nexus Component Download URL]] - Функции для получения ссылок на скачивание компонентов из Nexus
  - [[Jenkins/Exclude Infrastructure Changes from ChangeSets]] - Исключение изменений инфраструктурного репозитория из ChangeSets в Pipeline script from SCM
  - [[Jenkins/Role Strategy Roles and Users]] - Получение перечня ролей и назначенных пользователей/групп в Role-Based Authorization Strategy
  - [[Jenkins/Grype Configuration]] - Конфигурация Grype для сканирования уязвимостей на Jenkins агентах
  - [[Jenkins/Confluence Publisher Plugin Integration]] - Интеграция Confluence с Jenkins через пользовательские макросы и confluence-publisher-plugin
  - [[Jenkins/Jenkinsfile Code Style]] - Стандарты написания Jenkinsfile: структура, именование, форматирование, работа с параметрами
  - [[Jenkins/Shared Library Code Style]] - Стандарты разработки Jenkins Shared Library: Code Style, структура, Best Practices, известные ограничения

## Безопасность

- [[Security/Security Tools]] - Сканирование уязвимостей с помощью Trivy
- **Passbolt:**
  - [[Security/Passbolt/Passbolt to KeePassXC Export]] - Автоматизация импорта паролей из Passbolt в KeePassXC
  - [[Security/Passbolt/Passbolt Mass User Creation]] - Скрипт для массового создания пользователей в Passbolt

## Мониторинг

### Prometheus
- [[Monitoring/Prometheus/Windows Exporter]] - Установка и настройка Windows Exporter для сбора метрик Windows систем
- [[Monitoring/Prometheus/Proxmox VE Exporter]] - Установка и настройка Proxmox VE Exporter для сбора метрик Proxmox VE

## Инструменты

### Базы данных
- [[Databases/ClickHouse]] - Запросы ClickHouse и системные таблицы

### Управление секретами
- [[Vault/AppRole Management]] - Создание и управление AppRole в Vault
