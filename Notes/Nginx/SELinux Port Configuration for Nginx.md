---
created: 2026-04-24
tags: [nginx, selinux, security, tcp, ports, rhel]
category: infrastructure
---

# Открытие портов для Nginx при включенном SELinux

## Обзор

При включенном `SELinux` Nginx (домен `httpd_t`) может слушать только порты, разрешенные типом `http_port_t`.
Даже если порт открыт в firewall и указан в `nginx.conf`, запуск/перезапуск может завершиться ошибкой `Permission denied`.

## Когда это нужно

Настройка требуется, если Nginx должен слушать нестандартные порты (например, `8080`, `8443`, `8888`, `8082`) на системах RHEL-совместимого семейства (RHEL, Rocky, AlmaLinux, CentOS).

## Предварительные требования

```bash
# Установить утилиты SELinux (если semanage отсутствует)
sudo dnf install -y policycoreutils-python-utils
```

Проверить режим:

```bash
getenforce
```

Ожидаемый режим для данной инструкции: `Enforcing` или `Permissive`.

## Разрешить исходящие подключения для Nginx (опционально)

Если Nginx должен подключаться к внешним backend-сервисам (upstream/proxy_pass), включите boolean:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

Проверка:

```bash
getsebool httpd_can_network_connect
```

## Добавление портов в SELinux policy

### Базовые команды

```bash
sudo semanage port -a -t http_port_t -p tcp 8443
sudo semanage port -a -t http_port_t -p tcp 8080
sudo semanage port -a -t http_port_t -p tcp 8888
sudo semanage port -a -t http_port_t -p tcp 8082
```

### Если порт уже существует

Если при `-a` получаете ошибку вида `already defined`, используйте изменение:

```bash
sudo semanage port -m -t http_port_t -p tcp 8443
```

## Проверка

Проверить, что порты назначены `http_port_t`:

```bash
sudo semanage port -l | grep http_port_t
```

Проверить конфигурацию Nginx и применить:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Проверить, что процесс слушает нужные порты:

```bash
sudo ss -lntp | grep nginx
```

## Частые ошибки и диагностика

1. `semanage: command not found`  
   Установить пакет `policycoreutils-python-utils`.
2. `ValueError: Port tcp/XXXX already defined`  
   Использовать `semanage port -m ...` вместо `-a`.
3. Nginx не стартует после изменения конфигурации  
   Проверить `nginx -t`, затем просмотреть `audit.log`:
   ```bash
   sudo ausearch -m AVC -ts recent
   ```

## Откат изменений

Удалить правило для конкретного порта:

```bash
sudo semanage port -d -t http_port_t -p tcp 8888
```

Отключить boolean (если включали и больше не нужен):

```bash
sudo setsebool -P httpd_can_network_connect 0
```

## Связанные заметки

- [[Default Server Configuration]] - Базовая конфигурация Nginx и SSL
- [[PFX Certificate and Key Extraction]] - Подготовка сертификата и ключа для HTTPS
- [[../Linux/System Administration]] - Общие операции администрирования Linux
