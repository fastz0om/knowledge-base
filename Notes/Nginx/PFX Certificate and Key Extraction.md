---
created: 2026-04-24
tags: [nginx, ssl, tls, openssl, pfx, certificate, private-key]
category: infrastructure
---

# Получение сертификата и приватного ключа из PFX

## Обзор

`PFX`/`PKCS#12` контейнер обычно содержит:
- сертификат сервера (или клиента);
- приватный ключ;
- цепочку CA сертификатов.

Для настройки `Nginx` чаще всего нужны отдельные файлы:
- сертификат (`.crt`/`.pem`);
- приватный ключ (`.key`/`.pem`);
- (опционально) цепочка CA (`ca-bundle.crt`).

## Быстрый сценарий для Nginx

```bash
# 1) Извлечь сертификат сервера
openssl pkcs12 -in nginx.pfx -clcerts -nokeys -out server.crt

# 2) Извлечь приватный ключ (в зашифрованном PEM)
openssl pkcs12 -in nginx.pfx -nocerts -out private.key

# 3) Снять passphrase с ключа (если нужен ключ без пароля)
openssl rsa -in private.key -out private-unencrypted.key
```

## Команды извлечения (актуализированные)

### Приватный ключ

```bash
# Извлечь ключ без шифрования в выходном файле
openssl pkcs12 -in nginx.pfx -nocerts -nodes -out private.key

# Вариант, если у контейнера пустой пароль
openssl pkcs12 -in nginx.pfx -nocerts -nodes -passin pass:'' -out private.key
```

> Используйте `-nodes` только в защищенной среде. Такой ключ хранится в открытом виде.

### Ключ без passphrase (из уже зашифрованного ключа)

```bash
openssl rsa -in private.key -out private-unencrypted.key
```

### Сертификат (leaf/client cert)

```bash
# Извлечь сертификат без ключа
openssl pkcs12 -in nginx.pfx -clcerts -nokeys -out server.crt

# Вариант для пустого пароля и "чистого" блока CERTIFICATE
openssl pkcs12 -in nginx.pfx -clcerts -nokeys -passin pass:'' \
  | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > server.crt
```

### CA bundle (цепочка)

```bash
openssl pkcs12 -in nginx.pfx -cacerts -nokeys -out ca-bundle.crt
```

## Проверка извлеченных файлов

```bash
# Проверка сертификата
openssl x509 -in server.crt -noout -subject -issuer -dates

# Проверка приватного ключа
openssl rsa -in private-unencrypted.key -check -noout

# Сравнение modulus сертификата и ключа (должны совпадать)
openssl x509 -noout -modulus -in server.crt | openssl md5
openssl rsa  -noout -modulus -in private-unencrypted.key | openssl md5
```

## Использование в Nginx

```nginx
ssl_certificate     /etc/nginx/ssl/server.crt;
ssl_certificate_key /etc/nginx/ssl/private-unencrypted.key;
```

Если требуется полная цепочка, объедините `server.crt` и `ca-bundle.crt` в один файл (`fullchain.crt`) в правильном порядке.

## Безопасность

1. Не храните `private-unencrypted.key` в репозитории.
2. Ограничьте права доступа:
   ```bash
   chmod 600 private-unencrypted.key
   chmod 644 server.crt ca-bundle.crt
   ```
3. Передавайте секреты через защищенные хранилища (например, Vault), а не через историю shell.

## Связанные заметки

- [[Default Server Configuration]] - Базовая SSL конфигурация и default server в Nginx
