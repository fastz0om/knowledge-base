---
created: 2025-01-27
tags:
  - passbolt
  - user-management
  - automation
  - bash
  - scripting
category: security
---

# Массовое создание пользователей в Passbolt

## Описание

Скрипт для массового создания пользователей в Passbolt через CLI. Позволяет автоматизировать процесс добавления нескольких пользователей одновременно на основе массива данных.

## Требования

- **Passbolt CLI** - [https://github.com/passbolt/passbolt_cli](https://github.com/passbolt/passbolt_cli)
- **Административные права** - Для создания пользователей требуется роль администратора в Passbolt

### Проверка установки

Перед использованием скрипта убедитесь, что Passbolt CLI установлен:

```bash
which passbolt
```

### Проверка прав доступа

Убедитесь, что у используемой учетной записи есть права администратора:

```bash
passbolt whoami
```

## Настройка

Перед использованием скрипта необходимо настроить следующие переменные:

```bash
pb_server="'https://passbolt.example.local'"  # URL сервера Passbolt (в одинарных кавычках)
pb_pass="''"                                    # Пароль администратора Passbolt (в одинарных кавычках)
pb_asc="''"                                     # Путь к файлу приватного ключа (.asc)
```

### Описание переменных

- **`pb_server`** - URL адрес сервера Passbolt, обернутый в одинарные кавычки
- **`pb_pass`** - Пароль администратора Passbolt, обернутый в одинарные кавычки
- **`pb_asc`** - Путь к приватному ключу администратора (файл `.asc` для восстановления), обернутый в одинарные кавычки

### Формат таблицы пользователей

Пользователи задаются в массиве `table` в следующем формате:

```bash
"Имя Фамилия Почта@example.ru"
```

Где:
- **Первое слово** - Фамилия (last name)
- **Второе слово** - Имя (first name)
- **Третье слово** - Email адрес (username)

**Пример:**

```bash
table=(
    "Иванов Иван ivan.ivanov@example.ru"
    "Петров Петр petr.petrov@example.ru"
    "Сидоров Сидор sidor.sidorov@example.ru"
)
```

## Скрипт

```bash
#!/bin/bash

###
# Need passbolt-cli
# Need admin status
###

# Need passbolt-cli installed!
pb_server="'https://passbolt.example.local'" # by default
pb_pass="''"# your passwd
pb_asc="''" # path to your.asc

table=("Имя Фамилия Почта@example.ru"
    "Имя Фамилия Почта@example.ru"
)
user_names=()
first_names=()
last_names=()

for i in "${table[@]}"; do
    user_name=$(echo "$i" | awk '{print $3}')
    first_name=$(echo "$i" | awk '{print $2}')
    last_name=$(echo "$i" | awk '{print $1}')
    user_names+=("$user_name")
    first_names+=("$first_name")
    last_names+=("$last_name")
done

i=0
len=${#user_names[@]}
while [ $i -lt $len ]; do
    echo "/usr/bin/passbolt create user -u ${user_names[$i]} -f ${first_names[$i]} -l ${last_names[$i]} --userPassword $pb_pass --serverAddress $pb_server --userPrivateKeyFile $pb_asc"
    let i++
done

echo "Summary created users: $len"
```

## Использование

### Шаг 1: Настройка переменных

Отредактируйте скрипт и укажите значения переменных:

```bash
pb_server="'https://passbolt.example.local'"
pb_pass="'ваш_пароль_администратора'"
pb_asc="'/path/to/your/restore-key.asc'"
```

### Шаг 2: Подготовка списка пользователей

Отредактируйте массив `table` с данными пользователей:

```bash
table=(
    "Иванов Иван ivan.ivanov@example.ru"
    "Петров Петр petr.petrov@example.ru"
    "Сидоров Сидор sidor.sidorov@example.ru"
)
```

**Важно:** Каждая строка должна содержать ровно 3 слова, разделенные пробелами.

### Шаг 3: Сохранение скрипта

Сохраните скрипт в файл, например `passbolt_add_user.sh`:

```bash
nano passbolt_add_user.sh
# Вставьте содержимое скрипта
```

### Шаг 4: Установка прав на выполнение

```bash
chmod +x passbolt_add_user.sh
```

### Шаг 5: Проверка команд (dry-run)

По умолчанию скрипт выводит команды, но не выполняет их. Это позволяет проверить правильность формирования команд:

```bash
./passbolt_add_user.sh
```

Вы увидите список команд вида:

```bash
/usr/bin/passbolt create user -u ivan.ivanov@example.ru -f Иван -l Иванов --userPassword '...' --serverAddress '...' --userPrivateKeyFile '...'
/usr/bin/passbolt create user -u petr.petrov@example.ru -f Петр -l Петров --userPassword '...' --serverAddress '...' --userPrivateKeyFile '...'
...
Summary created users: 3
```

### Шаг 6: Выполнение команд

**Вариант 1: Выполнение через xargs**

После проверки команд, выполните их:

```bash
./passbolt_add_user.sh | grep "^/usr/bin/passbolt" | bash
```

**Вариант 2: Модификация скрипта для автоматического выполнения**

Отредактируйте скрипт, заменив строку с `echo` на фактический вызов команды:

```bash
# Было:
echo "/usr/bin/passbolt create user -u ${user_names[$i]} -f ${first_names[$i]} -l ${last_names[$i]} --userPassword $pb_pass --serverAddress $pb_server --userPrivateKeyFile $pb_asc"

# Стало:
/usr/bin/passbolt create user -u ${user_names[$i]} -f ${first_names[$i]} -l ${last_names[$i]} --userPassword $pb_pass --serverAddress $pb_server --userPrivateKeyFile $pb_asc
```

## Как работает скрипт

1. **Инициализация переменных** - Настройка сервера, пароля и пути к ключу
2. **Парсинг таблицы пользователей** - Извлечение данных из массива `table`:
   - Фамилия (последнее слово)
   - Имя (второе слово)
   - Email (третье слово)
3. **Формирование массивов** - Создание отдельных массивов для имен, фамилий и email адресов
4. **Генерация команд** - Формирование команд `passbolt create user` для каждого пользователя
5. **Вывод статистики** - Показывает количество пользователей, для которых будут созданы команды

## Примеры использования

### Пример 1: Создание трех пользователей

```bash
table=(
    "Иванов Иван ivan.ivanov@example.ru"
    "Петров Петр petr.petrov@example.ru"
    "Сидоров Сидор sidor.sidorov@example.ru"
)
```

**Результат:**

```
/usr/bin/passbolt create user -u ivan.ivanov@example.ru -f Иван -l Иванов --userPassword '...' --serverAddress '...' --userPrivateKeyFile '...'
/usr/bin/passbolt create user -u petr.petrov@example.ru -f Петр -l Петров --userPassword '...' --serverAddress '...' --userPrivateKeyFile '...'
/usr/bin/passbolt create user -u sidor.sidorov@example.ru -f Сидор -l Сидоров --userPassword '...' --serverAddress '...' --userPrivateKeyFile '...'
Summary created users: 3
```

### Пример 2: Модифицированный скрипт с автоматическим выполнением

```bash
#!/bin/bash

pb_server="'https://passbolt.example.local'"
pb_pass="'your_password'"
pb_asc="'/path/to/key.asc'"

table=(
    "Иванов Иван ivan.ivanov@example.ru"
    "Петров Петр petr.petrov@example.ru"
)

user_names=()
first_names=()
last_names=()

for i in "${table[@]}"; do
    user_name=$(echo "$i" | awk '{print $3}')
    first_name=$(echo "$i" | awk '{print $2}')
    last_name=$(echo "$i" | awk '{print $1}')
    user_names+=("$user_name")
    first_names+=("$first_name")
    last_names+=("$last_name")
done

i=0
len=${#user_names[@]}
success_count=0
fail_count=0

while [ $i -lt $len ]; do
    echo "Creating user: ${user_names[$i]}"
    if /usr/bin/passbolt create user \
        -u "${user_names[$i]}" \
        -f "${first_names[$i]}" \
        -l "${last_names[$i]}" \
        --userPassword "$pb_pass" \
        --serverAddress "$pb_server" \
        --userPrivateKeyFile "$pb_asc"; then
        echo "✓ User ${user_names[$i]} created successfully"
        ((success_count++))
    else
        echo "✗ Failed to create user ${user_names[$i]}"
        ((fail_count++))
    fi
    let i++
done

echo "Summary:"
echo "  Total users: $len"
echo "  Successfully created: $success_count"
echo "  Failed: $fail_count"
```

## Важные замечания

1. **Безопасность:**
   - Не храните пароли в открытом виде в скрипте
   - Используйте переменные окружения или защищенные хранилища паролей
   - Файл `.asc` с приватным ключом должен быть защищен (права доступа `600`)
   - Рекомендуется использовать права доступа `600` для файла скрипта

2. **Проверка данных:**
   - Убедитесь, что все email адреса уникальны
   - Проверьте корректность формата email адресов
   - Убедитесь, что пользователи с такими email еще не существуют в системе

3. **Логирование:**
   - Рекомендуется сохранять вывод скрипта в лог-файл
   - Пример: `./passbolt_add_user.sh > user_creation.log 2>&1`

4. **Обработка ошибок:**
   - Скрипт по умолчанию выводит команды без выполнения
   - Это позволяет проверить корректность перед фактическим выполнением
   - При автоматическом выполнении добавьте обработку ошибок

5. **Формат данных:**
   - Каждая строка в массиве `table` должна содержать ровно 3 слова
   - Формат: `"Фамилия Имя Email"`
   - Email должен быть уникальным идентификатором пользователя

## Troubleshooting

### Ошибка: "command not found: passbolt"

**Решение:** Установите Passbolt CLI согласно инструкциям на [GitHub](https://github.com/passbolt/passbolt_cli)

### Ошибка: "Insufficient privileges"

**Причина:** У учетной записи нет прав администратора

**Решение:**
- Убедитесь, что используете учетную запись администратора
- Проверьте права доступа: `passbolt whoami`

### Ошибка: "User already exists"

**Причина:** Пользователь с указанным email уже существует в системе

**Решение:**
- Проверьте список существующих пользователей: `passbolt find user -u <email>`
- Удалите дубликаты из массива `table`
- Или пропустите существующих пользователей в скрипте

### Ошибка: "Invalid email format"

**Причина:** Email адрес имеет неверный формат

**Решение:**
- Проверьте формат email адресов в массиве `table`
- Убедитесь, что email содержит символ `@` и корректный домен

### Ошибка: "Cannot parse table entry"

**Причина:** Строка в массиве `table` содержит неправильное количество слов

**Решение:**
- Убедитесь, что каждая строка содержит ровно 3 слова: Фамилия, Имя, Email
- Проверьте, что нет лишних пробелов или символов

### Ошибка при парсинге данных

**Решение:**
- Добавьте проверку формата данных перед парсингом:

```bash
for i in "${table[@]}"; do
    word_count=$(echo "$i" | wc -w)
    if [ "$word_count" -ne 3 ]; then
        echo "Error: Invalid format in line: $i (expected 3 words, got $word_count)"
        continue
    fi
    # ... остальной код парсинга
done
```

## Улучшения скрипта

### Добавление проверки существования пользователя

```bash
# Перед созданием пользователя проверяем, существует ли он
if passbolt find user -u "${user_names[$i]}" --serverAddress $pb_server --userPrivateKeyFile $pb_asc >/dev/null 2>&1; then
    echo "User ${user_names[$i]} already exists, skipping..."
    continue
fi
```

### Использование переменных окружения для паролей

```bash
# Вместо хранения пароля в скрипте
pb_pass="${PASSBOLT_ADMIN_PASSWORD}"

# Установка переменной окружения:
export PASSBOLT_ADMIN_PASSWORD="your_password"
```

### Сохранение результатов в лог-файл

```bash
log_file="passbolt_user_creation_$(date +%Y%m%d_%H%M%S).log"
exec > >(tee -a "$log_file")
exec 2>&1
```

## Связанные заметки

- [[Passbolt to KeePassXC Export]] - Экспорт паролей из Passbolt в KeePassXC
- [[../Security Tools]] - Другие инструменты безопасности

