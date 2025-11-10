---
created: 2025-01-27
tags: [vault, security, approle, secrets-management]
category: infrastructure
---

# HashiCorp Vault - Управление AppRole

## Обзор

Создание и управление AppRole для аутентификации приложений в HashiCorp Vault.

## Создание AppRole

### Создание нового AppRole

Создать новый AppRole для аутентификации приложения:

```bash
vault write auth/approle/role/jenkins_addvisor \
  secret_id_ttl=0 \
  token_ttl=60m \
  token_max_ttl=120m \
  policies="jenkins"
```

**Параметры:**
- `secret_id_ttl=0` - Secret ID никогда не истекает
- `token_ttl=60m` - Время жизни токена (1 час)
- `token_max_ttl=120m` - Максимальное время жизни токена (2 часа)
- `policies` - Список политик через запятую

## Получение данных AppRole

### Получение Role ID

```bash
vault read auth/approle/role/jenkins_addvisor/role-id
```

**Вывод:**
```
Key        Value
---        -----
role_id    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Генерация Secret ID

```bash
vault write -f auth/approle/role/jenkins_addvisor/secret-id
```

**Вывод:**
```
Key                   Value
---                   -----
secret_id             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
secret_id_accessor    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## Процесс аутентификации

1. **Приложение получает Role ID и Secret ID**
2. **Приложение аутентифицируется:**
   ```bash
   vault write auth/approle/login \
     role_id=<role-id> \
     secret_id=<secret-id>
   ```
3. **Vault возвращает токен** с настроенным TTL и политиками

## Рекомендации по безопасности

1. **Ротация Secret ID** - Регулярно меняйте Secret ID
2. **Короткие TTL токенов** - Используйте короткое время жизни токенов
3. **Минимальные права** - Ограничивайте политики минимально необходимыми разрешениями
4. **Мониторинг** - Отслеживайте использование AppRole

## Полезные команды

### Список всех AppRole

```bash
vault list auth/approle/role
```

### Просмотр конфигурации AppRole

```bash
vault read auth/approle/role/jenkins_addvisor
```

### Удаление AppRole

```bash
vault delete auth/approle/role/jenkins_addvisor
```

## Связанные заметки

- [[../Jenkins/Pipeline Scripts]]

