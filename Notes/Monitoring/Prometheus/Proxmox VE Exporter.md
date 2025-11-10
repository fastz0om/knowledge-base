---
created: 2025-01-27
tags:
  - prometheus
  - proxmox
  - pve
  - monitoring
  - exporter
category: monitoring
---

# Proxmox VE Exporter для Prometheus

## Описание

Prometheus PVE Exporter - это экспортер метрик для Proxmox VE, который позволяет собирать и экспортировать метрики виртуальных машин, контейнеров и хоста в формате Prometheus.

**GitHub репозиторий:** [https://github.com/prometheus-pve/prometheus-pve-exporter](https://github.com/prometheus-pve/prometheus-pve-exporter)

## Требования

- Proxmox VE сервер
- Python 3
- pip (Python package manager)
- Доступ к Nexus репозиторию (или публичный PyPI)
- API токен Proxmox VE для аутентификации

## Установка

### Шаг 1: Создание системного пользователя

Создайте системного пользователя для экспортера:

```bash
useradd -s /bin/false pve-exporter
```

Этот пользователь будет использоваться для запуска экспортера как systemd сервиса.

### Шаг 2: Установка Python и pip

Установите Python 3 и pip:

```bash
apt install -y python3-pip
```

### Шаг 3: Настройка pip для работы с Nexus

Если используется внутренний Nexus репозиторий для Python пакетов, необходимо настроить `pip.conf`:

```bash
nano /etc/pip.conf
```

Добавьте следующую конфигурацию:

```ini
[global]
index = https://nexus.example.local/repository/pypi-test-group/pypi
index-url = https://nexus.example.local/repository/pypi-test-group/simple
trusted_host = nexus.example.local
break-system-packages = true
```

**Примечание:** Если используется публичный PyPI, этот шаг можно пропустить.

### Шаг 4: Создание конфигурационного файла

Создайте директорию для конфигурации (если её еще нет):

```bash
mkdir -p /etc/prometheus
```

Создайте конфигурационный файл:

```bash
nano /etc/prometheus/pve.yaml
```

Добавьте следующую конфигурацию:

```yaml
default:
    user: prometheus@pam
    token_name: "prometheus"
    token_value: "<GO_TO_PASSBOLT.INRIGHTS.LOCAL>"
    verify_ssl: false
```

**Важные параметры:**

- **`user`** - Пользователь Proxmox VE для аутентификации (обычно в формате `username@pam` или `username@pve`)
- **`token_name`** - Имя токена (можно задать произвольно)
- **`token_value`** - Значение API токена из Passbolt (замените `<GO_TO_PASSBOLT.INRIGHTS.LOCAL>` на фактический токен)
- **`verify_ssl`** - Проверка SSL сертификатов (`false` для самоподписанных сертификатов)

**Получение API токена в Proxmox VE:**

1. Войдите в веб-интерфейс Proxmox VE
2. Перейдите в **Datacenter → Permissions → API Tokens**
3. Создайте новый токен для пользователя `prometheus@pam`
4. Скопируйте значение токена (формат: `username@realm!token-name=token-value`)
5. Вставьте значение токена в поле `token_value`

**Пример конфигурации с несколькими Proxmox узлами:**

```yaml
default:
    user: prometheus@pam
    token_name: "prometheus"
    token_value: "your-token-value"
    verify_ssl: false

proxmox1:
    user: prometheus@pam
    token_name: "prometheus"
    token_value: "your-token-value"
    verify_ssl: false
    host: 192.168.1.10

proxmox2:
    user: prometheus@pam
    token_name: "prometheus"
    token_value: "your-token-value"
    verify_ssl: false
    host: 192.168.1.11
```

### Шаг 5: Установка экспортера

Установите пакет `prometheus-pve-exporter`:

```bash
python3 -m pip install prometheus-pve-exporter
```

**Примечание:** Если используется внутренний Nexus, пакет будет установлен из него, в противном случае - из публичного PyPI.

### Шаг 6: Настройка прав доступа

Установите правильные права доступа на конфигурационный файл:

```bash
chmod 600 /etc/prometheus/pve.yaml
chown pve-exporter:pve-exporter /etc/prometheus/pve.yaml
```

### Шаг 7: Создание systemd сервиса

Создайте файл сервиса:

```bash
nano /etc/systemd/system/prometheus-pve-exporter.service
```

Добавьте следующее содержимое:

```ini
[Unit]
Description=Prometheus Proxmox VE Exporter
Documentation=https://github.com/prometheus-pve/prometheus-pve-exporter

[Service]
Restart=always
User=pve-exporter
ExecStart=pve_exporter --config.file="/etc/prometheus/pve.yaml"

[Install]
WantedBy=multi-user.target
```

**Важно:** Проверьте путь к исполняемому файлу `pve_exporter`. Если он находится не в PATH, укажите полный путь:

```ini
ExecStart=/usr/local/bin/pve_exporter --config.file="/etc/prometheus/pve.yaml"
```

Или найдите путь с помощью:

```bash
which pve_exporter
```

### Шаг 8: Запуск сервиса

Перезагрузите systemd и запустите сервис:

```bash
systemctl daemon-reload
systemctl enable prometheus-pve-exporter --now
```

Команда `--now` также запустит сервис немедленно.

## Проверка работы

### Проверка статуса сервиса

```bash
systemctl status prometheus-pve-exporter
```

### Проверка доступности метрик

Экспортер по умолчанию слушает на порту `9221`. Проверьте доступность метрик:

```bash
curl --silent http://127.0.0.1:9221/pve | grep pve_version_info
```

**Ожидаемый результат:**

```bash
# HELP pve_version_info Proxmox VE version info
# TYPE pve_version_info gauge
pve_version_info{release="7.2",repoid="963997e8",version="7.2-15"} 1.0
```

### Просмотр всех метрик

Для просмотра всех доступных метрик:

```bash
curl http://127.0.0.1:9221/pve
```

Или с помощью браузера откройте:

```
http://your-proxmox-host:9221/pve
```

## Основные метрики

Экспортер собирает следующие типы метрик:

- **Информация о версии Proxmox VE** - `pve_version_info`
- **Метрики узлов кластера** - CPU, память, диски
- **Метрики виртуальных машин** - CPU, память, диск, сеть
- **Метрики контейнеров (LXC)** - ресурсы, статус
- **Метрики хранилища** - использование, доступное пространство
- **Статус бэкапов** - статус и результаты бэкапов

## Настройка Prometheus

Добавьте конфигурацию в `prometheus.yml` для сбора метрик:

```yaml
scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets: 
          - 'proxmox-host-1:9221'
          - 'proxmox-host-2:9221'
        labels:
          cluster: 'production'
```

Если используется несколько Proxmox узлов в конфигурации, можно настроить отдельные job'ы:

```yaml
scrape_configs:
  - job_name: 'pve-default'
    static_configs:
      - targets: ['proxmox-host-1:9221']
    params:
      module: ['default']
  
  - job_name: 'pve-cluster'
    static_configs:
      - targets: ['proxmox-host-1:9221']
    params:
      module: ['proxmox1', 'proxmox2']
```

## Управление сервисом

### Остановка сервиса

```bash
systemctl stop prometheus-pve-exporter
```

### Перезапуск сервиса

```bash
systemctl restart prometheus-pve-exporter
```

### Просмотр логов

```bash
journalctl -u prometheus-pve-exporter -f
```

### Отключение автозапуска

```bash
systemctl disable prometheus-pve-exporter
```

## Troubleshooting

### Сервис не запускается

1. **Проверьте логи:**

```bash
journalctl -u prometheus-pve-exporter -n 50
```

2. **Проверьте конфигурационный файл:**

```bash
pve_exporter --config.file="/etc/prometheus/pve.yaml" --dry-run
```

3. **Проверьте права доступа:**

```bash
ls -la /etc/prometheus/pve.yaml
```

4. **Проверьте, что пользователь существует:**

```bash
id pve-exporter
```

### Ошибки аутентификации

**Симптом:** Ошибки вида "401 Unauthorized" или "authentication failed"

**Решение:**

1. Проверьте правильность API токена в конфигурационном файле
2. Убедитесь, что токен не истек
3. Проверьте, что пользователь имеет необходимые права в Proxmox VE
4. Убедитесь, что формат токена правильный

### Метрики не собираются

**Симптом:** Endpoint доступен, но метрики пустые или отсутствуют

**Решение:**

1. Проверьте доступность Proxmox VE API:

```bash
curl -k -H "Authorization: PVEToken prometheus@pam!prometheus=your-token" https://localhost:8006/api2/json/version
```

2. Проверьте логи экспортера на наличие ошибок
3. Убедитесь, что указан правильный модуль в конфигурации
4. Проверьте права пользователя в Proxmox VE

### Ошибки SSL

**Симптом:** Ошибки вида "SSL certificate verification failed"

**Решение:**

1. Установите `verify_ssl: false` в конфигурации (для самоподписанных сертификатов)
2. Или добавьте корневой сертификат в систему

### Проблемы с Nexus

**Симптом:** Не удается установить пакет из Nexus

**Решение:**

1. Проверьте доступность Nexus:

```bash
curl -I https://nexus.example.local/repository/pypi-test-group/simple
```

2. Проверьте конфигурацию `/etc/pip.conf`
3. Попробуйте установить с явным указанием репозитория:

```bash
pip3 install --index-url https://nexus.example.local/repository/pypi-test-group/simple prometheus-pve-exporter
```

## Best Practices

1. **Безопасность:**
   - Ограничьте доступ к порту 9221 через файрвол
   - Используйте минимально необходимые права для API токена
   - Регулярно обновляйте API токены
   - Храните конфигурационный файл с правами 600

2. **Мониторинг:**
   - Настройте алерты на недоступность экспортера
   - Мониторьте использование ресурсов экспортером
   - Отслеживайте ошибки в логах

3. **Обслуживание:**
   - Регулярно обновляйте экспортер до последней версии
   - Проверяйте совместимость версий после обновления Proxmox VE
   - Делайте резервные копии конфигурационных файлов

4. **Производительность:**
   - Используйте фильтры для ограничения собираемых метрик при необходимости
   - Настройте интервалы сбора данных в Prometheus

## Дополнительная конфигурация

### Изменение порта

Для изменения порта по умолчанию (9221), добавьте параметр в systemd сервис:

```ini
ExecStart=pve_exporter --config.file="/etc/prometheus/pve.yaml" --web.listen-address=":9222"
```

### Фильтрация метрик

Экспортер собирает метрики со всех узлов и VM по умолчанию. При необходимости можно настроить фильтрацию через параметры запуска или конфигурацию.

### Множественные Proxmox узлы

Для мониторинга нескольких Proxmox узлов, добавьте их в конфигурационный файл (см. пример выше) и укажите нужные модули при обращении к экспортеру или в конфигурации Prometheus.

## Связанные заметки

- [[Windows Exporter]] - Установка и настройка Windows Exporter для Prometheus
- [[../../Proxmox/Backup and Restore]] - Операции с резервным копированием Proxmox

