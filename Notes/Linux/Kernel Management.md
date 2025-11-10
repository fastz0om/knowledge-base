---
created: 2025-01-27
tags: [linux, kernel, administration, how-to]
category: infrastructure
---

# Управление ядром Linux

## Обзор

Управление версиями ядра в различных дистрибутивах Linux: Ubuntu, Debian, CentOS и RedOS.

## Управление ядром Ubuntu

### Просмотр доступных версий ядра

```bash
apt list linux-headers-5.15.0-*-generic
```

### Установка конкретной версии ядра

```bash
apt install linux-headers-5.15.0-130-generic \
            linux-modules-5.15.0-130-generic \
            linux-image-unsigned-5.15.0-130-generic \
            linux-cloud-tools-5.15.0-130-generic \
            linux-tools-5.15.0-130-generic
```

### Проверка установленных ядер

```bash
# Список всех установленных заголовков ядра
dpkg --list | grep linux-headers

# Список всех установленных модулей ядра
dpkg --list | grep linux-modules

# Список всех установленных образов ядра
dpkg --list | grep linux-image

# Список заголовков HWE (Hardware Enablement)
dpkg --list | grep linux-hwe-headers
dpkg --list | grep linux-hwe
```

### Удаление конкретной версии ядра

```bash
apt-get remove --purge ${package_name}
```

**Пример:**
```bash
apt-get remove --purge linux-headers-5.15.0-100-generic
```

### Проверка доступных ядер

```bash
# Проверить установленные образы ядра
dpkg --list | grep linux-image

# Проверить каталог модулей ядра
ls -ls /lib/modules
```

## Управление ядром Debian

### Установка конкретной версии ядра

```bash
apt-get install linux-headers-6.1.0-29-amd64 \
                linux-image-6.1.0-29-amd64
```

## Управление ядром CentOS/RHEL

### Удаление старых ядер (оставить только последнее)

```bash
dnf remove $(dnf repoquery --installonly --latest-limit=-1)
```

**Примечание:** Это удаляет все ядра кроме последнего. Будьте осторожны!

## Управление ядром RedOS

### Проверка доступных модулей ядра

```bash
dnf list --showduplicates kernel-lt-devel
```

### Установка конкретного модуля ядра

```bash
# Установить конкретную версию
dnf install kernel-lt-devel-6.1.128-2.el7.3

# Установить заголовки и модули ядра
dnf install kernel-headers kernel-modules
```

## Рекомендации

1. **Всегда оставляйте резервное ядро** - Не удаляйте все старые ядра, пока не убедитесь, что новое работает
2. **Перезагрузка после установки** - Новое ядро активируется только после перезагрузки
3. **Проверка конфигурации GRUB** - Убедитесь, что записи загрузки корректны
4. **Тестирование нового ядра** - Загрузитесь в новое ядро и проверьте стабильность системы

## Связанные заметки

- [[Package Management]]
- [[System Administration]]

