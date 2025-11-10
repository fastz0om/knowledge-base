---
created: 2025-01-27
tags:
  - passbolt
  - keepassxc
  - password-management
  - automation
  - bash
  - scripting
category: security
---

# Импорт паролей Passbolt в KeePassXC

## Описание

Скрипт для автоматического импорта текущих паролей из Passbolt в KeePassXC. Пароли импортируются в указанную группу базы данных KeePassXC (по умолчанию `./ROOT`).

## Требования

Для работы скрипта требуются два CLI инструмента:

1. **Passbolt CLI** - [https://github.com/passbolt/passbolt_cli](https://github.com/passbolt/passbolt_cli)
2. **KeePassXC CLI** - входит в состав KeePassXC [https://www.mankier.com/1/keepassxc-cli](https://www.mankier.com/1/keepassxc-cli)

### Проверка установки

Перед использованием скрипта убедитесь, что оба инструмента установлены и доступны в PATH:

```bash
which passbolt
which keepassxc-cli
```

## Настройка

Перед использованием скрипта необходимо настроить следующие переменные:

```bash
pb_server="https://passbolt.example.local"  # URL сервера Passbolt
pb_asc=""                                      # Путь к файлу <restore-key>.asc
kpxc_db_path=''                                # Путь к базе данных KeePassXC (например: /home/user/test.kdbx)
kpxc_db_key_file=''                            # Путь к файлу ключа KeePassXC (например: /home/user/key.keyx). Может быть пустым
kpxc_db_passbolt_write=""                     # Путь к записи с паролем Passbolt в KeePassXC (например: MY/passbolt)
```

### Описание переменных

- **`pb_server`** - URL адрес сервера Passbolt
- **`pb_asc`** - Путь к приватному ключу Passbolt (файл `.asc` для восстановления)
- **`kpxc_db_path`** - Полный путь к файлу базы данных KeePassXC (`.kdbx`)
- **`kpxc_db_key_file`** - Путь к файлу ключа KeePassXC (`.keyx`), если используется. Может быть пустым, если база защищена только паролем
- **`kpxc_db_passbolt_write`** - Путь к записи в KeePassXC, где хранится пароль для доступа к Passbolt. Путь указывается относительно корневой директории базы (например: `MY/passbolt`). Корневую директорию (`ROOT`) указывать не нужно

### Получение пути к записи в KeePassXC

Путь к записи (`kpxc_db_passbolt_write`) формируется из названий групп и записи в интерфейсе KeePassXC:

1. Откройте базу данных KeePassXC
2. Найдите запись с паролем Passbolt
3. Посмотрите путь к записи (например, `MY` → `passbolt`)
4. Используйте этот путь в формате `MY/passbolt` (без `ROOT`)

## Скрипт

```bash
#!/bin/bash

###
# Скрипт для импорта текущих паролей Passbolt в KeePassXC (в группу ./ROOT)
# Для работы потребуется 2 cli: от passbolt и от keepassxc (идет в комплекте)
# https://github.com/passbolt/passbolt_cli
# https://www.mankier.com/1/keepassxc-cli
# Экспортируемый passbolt.kdbx загрузится в ту же директорию, откуда и запускался скрипт
###

###
# Variables
###
pb_server="https://passbolt.example.local" # Default URL
pb_asc=""                                   # Path to your <restore-key>.asc
kpxc_db_path=''                             # Example: /home/sy-admin/test.kdbx
kpxc_db_key_file=''                         # Example: /home/sy-admin/key.keyx
kpxc_db_passbolt_write=""                   # In which group the password is stored passbolt Example: MY/passbolt. View in UI. Please exlude Root dir.

###
# Check if installed
###
passbolt_cli_exe=$(which passbolt)
keepassxc_cli_exe=$(which keepassxc-cli)

if [[ -x "$passbolt_cli_exe" ]]; then
    echo "Passbolt installed, continue"
else
    echo "Passbolt cli not installed or broken"
    exit 1
fi

if [[ -x "$keepassxc_cli_exe" ]]; then
    echo "KeePassXC cli installed, continue"
else
    echo "KeePassXC cli not installed or broken"
    exit 1
fi

###
# Get passbolt password
##
if [[ "${kpxc_db_key_file}" != null ]]; then
    pb_pass=$(keepassxc-cli show "${kpxc_db_path}" "${kpxc_db_passbolt_write}" -k "${kpxc_db_key_file}" --show-protected | sed -n 3p)
    pb_pass=${pb_pass#*: }
else
    pb_pass=$(keepassxc-cli show "${kpxc_db_path}" "${kpxc_db_passbolt_write}" --show-protected | sed -n 3p)
    pb_pass=${pb_pass#*: }
fi

###
# Configure passbolt
###
passbolt configure --serverAddress "${pb_server}" --userPassword "${pb_pass}" --userPrivateKeyFile "${pb_asc}"

###
# Export passbolt database to folder
###
if [[ "${kpxc_db_key_file}" != null ]]; then
    echo "Create export kdbx password"
    passbolt export keepass
    while true; do
        read -r -p "Do you wish to merge new database with current? (Y/N): " answer
        case $answer in
        [Yy]*)
            keepassxc-cli merge "${kpxc_db_path}" ./passbolt-export.kdbx -k "${kpxc_db_key_file}"
            break
            ;;
        [Nn]*)
            exit
            ;;
        *) echo "Please answer У or N." ;;
        esac
    done
else
    echo "Create export kdbx password"
    passbolt export keepass
    while true; do
        read -r -p "Do you wish to merge new database with current? (Y/N): " answer
        case $answer in
        [Yy]*)
            keepassxc-cli merge "${kpxc_db_path}" ./passbolt-export.kdbx
            break
            ;;
        [Nn]*)
            exit
            ;;
        *) echo "Please answer У or N." ;;
        esac
    done
fi
```

## Использование

### Шаг 1: Настройка переменных

Отредактируйте скрипт и укажите значения всех необходимых переменных:

```bash
pb_server="https://passbolt.example.local"
pb_asc="/path/to/your/restore-key.asc"
kpxc_db_path="/home/user/passwords.kdbx"
kpxc_db_key_file="/home/user/passwords.keyx"  # Опционально, если используется файл ключа
kpxc_db_passbolt_write="Credentials/passbolt"
```

### Шаг 2: Создание файла скрипта

Сохраните скрипт в файл, например `autoexport_database.sh`:

```bash
nano autoexport_database.sh
# Вставьте содержимое скрипта
```

### Шаг 3: Установка прав на выполнение

```bash
chmod +x autoexport_database.sh
```

### Шаг 4: Запуск скрипта

```bash
./autoexport_database.sh
```

## Как работает скрипт

1. **Проверка установки инструментов** - Проверяет наличие `passbolt` и `keepassxc-cli` в PATH
2. **Получение пароля Passbolt** - Извлекает пароль Passbolt из базы данных KeePassXC:
   - Если указан файл ключа, использует его для открытия базы
   - Извлекает пароль из указанной записи (`kpxc_db_passbolt_write`)
3. **Настройка Passbolt CLI** - Настраивает подключение к серверу Passbolt с использованием пароля и приватного ключа
4. **Экспорт базы данных** - Экспортирует базу данных Passbolt в формате KeePass (создает файл `passbolt-export.kdbx` в текущей директории)
5. **Слияние баз** - Предлагает объединить экспортированную базу с существующей базой KeePassXC:
   - При ответе `Y` - объединяет базы
   - При ответе `N` - завершает работу без изменений

## Примеры использования

### Пример 1: База защищена только паролем

```bash
pb_server="https://passbolt.example.local"
pb_asc="/home/user/.passbolt/restore-key.asc"
kpxc_db_path="/home/user/passwords.kdbx"
kpxc_db_key_file=""  # Пусто, база защищена только паролем
kpxc_db_passbolt_write="Passwords/passbolt-main"
```

### Пример 2: База защищена паролем и файлом ключа

```bash
pb_server="https://passbolt.example.local"
pb_asc="/home/user/.passbolt/restore-key.asc"
kpxc_db_path="/home/user/passwords.kdbx"
kpxc_db_key_file="/home/user/passwords.keyx"  # Файл ключа указан
kpxc_db_passbolt_write="Work/passbolt"
```

## Важные замечания

1. **Безопасность:**
   - Файл `.asc` с приватным ключом должен быть защищен от несанкционированного доступа
   - Рекомендуется использовать права доступа `600` для файла скрипта и ключевых файлов
   - Не храните пароли в открытом виде в скрипте

2. **Резервное копирование:**
   - Перед слиянием баз рекомендуется создать резервную копию базы данных KeePassXC

3. **Формат экспорта:**
   - Экспортированный файл `passbolt-export.kdbx` создается в текущей директории
   - После успешного слияния этот файл можно удалить

4. **Интерактивный режим:**
   - Скрипт запрашивает подтверждение перед слиянием баз
   - Можно ответить `N`, чтобы отменить операцию

## Troubleshooting

### Ошибка: "Passbolt cli not installed or broken"

**Решение:** Установите Passbolt CLI согласно инструкциям на [GitHub](https://github.com/passbolt/passbolt_cli)

### Ошибка: "KeePassXC cli not installed or broken"

**Решение:** Убедитесь, что KeePassXC установлен. CLI инструмент обычно находится в `/usr/bin/keepassxc-cli` или доступен через `keepassxc-cli`

### Ошибка при получении пароля из KeePassXC

**Причины:**
- Неверный путь к записи (`kpxc_db_passbolt_write`)
- Неверный путь к файлу ключа (если используется)
- Неверный пароль базы данных

**Решение:**
- Проверьте путь к записи в интерфейсе KeePassXC
- Убедитесь, что путь указан относительно корневой директории (без `ROOT`)
- Проверьте правильность пути к файлу ключа

### Ошибка при настройке Passbolt

**Причины:**
- Неверный URL сервера
- Неверный пароль
- Неверный путь к файлу `.asc`

**Решение:**
- Проверьте доступность сервера Passbolt
- Убедитесь, что пароль извлекается корректно
- Проверьте существование и читаемость файла `.asc`

### Ошибка при слиянии баз

**Решение:**
- Убедитесь, что у вас есть права на запись в базу данных KeePassXC
- Проверьте, что база данных не открыта в KeePassXC GUI
- Убедитесь, что файл `passbolt-export.kdbx` существует в текущей директории

## Связанные заметки

- [[../Security Tools]] - Другие инструменты безопасности
- [[Passbolt Mass User Creation]] - Массовое создание пользователей в Passbolt

