# Индекс базы знаний

Добро пожаловать в вашу базу знаний! Используйте это как отправную точку.

## Быстрые ссылки

- [[Problems]] - Список встреченных проблем
- [[Solutions]] - Решения типовых проблем
- [[Knowledge Base]] - Общие заметки знаний, организованные по категориям

## Содержимое базы знаний

### Инфраструктура

#### Linux
- [[Linux/Kernel Management]] - Операции с ядрами для Ubuntu/Debian/CentOS/RedOS
- [[Linux/Package Management]] - Операции с пакетами и офлайн установка
- [[Linux/System Administration]] - LVM, SSH, GPG, NTP, операции с Git

#### Контейнеры
- [[Docker/Container Operations]] - Управление жизненным циклом контейнеров

#### Виртуализация
- [[Proxmox/Backup and Restore]] - Резервное копирование и восстановление VM

#### Kubernetes
- [[K8s/Overview]] - Обзор Kubernetes и архитектура
- [[K8s/kubectl Commands]] - Полезные команды kubectl
- **Workload Resources:**
  - [[K8s/Resources/Workload Resources/Pod]] - Базовый ресурс Kubernetes
  - [[K8s/Resources/Workload Resources/Pod Template Image Pull Policy]] - Решение проблемы alwaysPullImage в Jenkins podTemplate
  - [[K8s/Resources/Workload Resources/Deployment]] - Управление развертываниями
  - [[K8s/Resources/Workload Resources/StatefulSet]] - Stateful приложения
  - [[K8s/Resources/Workload Resources/DaemonSet]] - Поды на каждом узле
  - [[K8s/Resources/Workload Resources/Job]] - Одноразовые задачи
  - [[K8s/Resources/Workload Resources/CronJob]] - Периодические задачи
- **Networking:**
  - [[K8s/Resources/Networking/Service]] - Стабильный сетевой endpoint
  - [[K8s/Resources/Networking/Ingress]] - Управление входящим трафиком
- **Configuration:**
  - [[K8s/Resources/Configuration Resources/ConfigMap]] - Конфигурационные данные
  - [[K8s/Resources/Configuration Resources/Secret]] - Секретные данные
  - [[K8s/Resources/Configuration Resources/Namespace]] - Логическое разделение ресурсов
- **Storage:**
  - [[K8s/Resources/Storage Resources/PersistentVolume]] - Выделенное хранилище
  - [[K8s/Resources/Storage Resources/PersistentVolumeClaim]] - Запросы на хранилище
- **RBAC:**
  - [[K8s/Resources/RBAC/ServiceAccount]] - Идентичность для подов
  - [[K8s/Resources/RBAC/Role]] - Права доступа
  - [[K8s/Resources/RBAC/RoleBinding]] - Привязка ролей
- **Problems:**
  - [[K8s/Solutions/PVC Mount Error]] - Решение проблемы монтирования Persistent Volume Claims

### Автоматизация и CI/CD
- **Jenkins:**
  - [[Jenkins/Pipeline Scripts]] - Groovy скрипты, версионирование, управление задачами
  - [[Jenkins/Git Timeout Fix]] - Исправление git timeout в MultiBranch Pipeline
  - [[Jenkins/Reset Build Number]] - Сброс индекса номера сборки на определенное значение
  - [[Jenkins/Active Choice Parameter Logging]] - Настройка логгирования Active Choice Parameter
  - [[Jenkins/Using Credentials in Active Choice Parameter]] - Использование секретов Jenkins в Active Choice Parameter для динамических списков
  - [[Jenkins/Nexus Component Download URL]] - Получение ссылок на скачивание компонентов из Nexus
  - [[Jenkins/Exclude Infrastructure Changes from ChangeSets]] - Исключение изменений инфраструктурного репозитория из ChangeSets
  - [[Jenkins/Role Strategy Roles and Users]] - Получение списка ролей и назначенных пользователей в Role Strategy
  - [[Jenkins/Grype Configuration]] - Конфигурация Grype для сканирования уязвимостей на Jenkins агентах
  - [[Jenkins/Confluence Publisher Plugin Integration]] - Интеграция Confluence с Jenkins через пользовательские макросы
  - [[Jenkins/Jenkinsfile Code Style]] - Стандарты написания и оформления Jenkinsfile
  - [[Jenkins/Shared Library Code Style]] - Стандарты разработки Jenkins Shared Library: Code Style и Best Practices

### Безопасность
- [[Security/Security Tools]] - Сканирование уязвимостей Trivy
- **Passbolt:**
  - [[Security/Passbolt/Passbolt to KeePassXC Export]] - Скрипт для импорта паролей из Passbolt в KeePassXC
  - [[Security/Passbolt/Passbolt Mass User Creation]] - Массовое создание пользователей в Passbolt

### Мониторинг

#### Prometheus
- [[Monitoring/Prometheus/Windows Exporter]] - Установка и настройка Windows Exporter для Prometheus
- [[Monitoring/Prometheus/Proxmox VE Exporter]] - Установка и настройка Proxmox VE Exporter для Prometheus

### Инструменты

#### Базы данных
- [[Databases/ClickHouse]] - Запросы и системные таблицы ClickHouse

#### Управление секретами
- [[Vault/AppRole Management]] - Операции с Vault AppRole

## Как использовать

1. **Ежедневные заметки**: Используйте `Ctrl+P` → "Daily Note" для создания ежедневных заметок
2. **Проблемы**: Используйте шаблон `Problem Template` при документировании проблем
3. **Решения**: Используйте шаблон `Solution Template` для документирования решений
4. **Знания**: Используйте шаблон `Knowledge Note Template` для общих знаний

## Горячие клавиши

- `Ctrl+P` - Командная палитра
- `Ctrl+I` - Вставить шаблон
- `Ctrl+O` - Открыть ссылку
- `Ctrl+G` - Граф связей (просмотр связей)
