---
created: 2025-01-27
tags: [linux, packages, apt, dpkg, dnf, how-to]
category: infrastructure
---

# Управление пакетами Linux

## Обзор

Основные операции управления пакетами для дистрибутивов Debian/Ubuntu и основанных на RHEL.

## Управление пакетами Debian/Ubuntu

### Сравнение версий пакетов

```bash
dpkg --compare-versions 0.1.38~pre-3 gt 0.1.38-4 && echo "yes" || echo "no"
```

**Операторы сравнения:**
- `lt` - меньше чем
- `le` - меньше или равно
- `eq` - равно
- `ne` - не равно
- `ge` - больше или равно
- `gt` - больше чем

### Загрузка пакетов с зависимостями (офлайн установка)

Полезно для установки пакетов на системах без доступа к интернету.

```bash
# Установить инструмент для зависимостей
apt-get install apt-rdepends

# Загрузить пакет со всеми зависимостями
apt-get download $(apt-rdepends vim | grep -v "^ " | sed 's/debconf-2.0/debconf/g')
```

**Шаги:**
1. Установить `apt-rdepends` для разрешения зависимостей
2. Получить список зависимостей с помощью `apt-rdepends`
3. Отфильтровать строки с отступом (подзависимости)
4. Загрузить все пакеты с помощью `apt-get download`

**Передача пакетов:**
```bash
# На исходной системе
apt-get download $(apt-rdepends <package> | grep -v "^ ")

# Передать на целевую систему
scp *.deb user@target:/tmp/

# На целевой системе
dpkg -i /tmp/*.deb
apt-get install -f  # Исправить зависимости при необходимости
```

## Обновление системы

### Обновление дистрибутива Ubuntu (Bionic → Focal)

```bash
# Обновить источники репозиториев
sed -i 's/bionic/focal/g' /etc/apt/*.list
sed -i 's/bionic/focal/g' /etc/apt/*/*.list

# Добавить новый GPG ключ (пример)
wget --quiet -O - https://nexus.example.local/repository/gpg-keys/keys/puppet_new_key.gpg | sudo apt-key add -

# Обновить и выполнить полное обновление
apt update && apt -y full-upgrade

# Запустить puppet после обновления
puppet agent -t
```

**Предупреждение:** Всегда делайте резервную копию перед крупными обновлениями!

## Связанные заметки

- [[Kernel Management]]
- [[System Administration]]

