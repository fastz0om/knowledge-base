---
created: 2026-04-24
tags: [vault, gitlab, ci, jwt, id-token, secrets-management]
category: infrastructure
---

# Интеграция Vault + GitLab CI (CE) через ID Tokens (JWT)

## Обзор

Для GitLab CI (Community Edition) можно аутентифицироваться в Vault через `id_tokens` (OIDC JWT), а затем:
1. читать секреты напрямую через HTTP API Vault;
2. использовать `vault` CLI (бинарник) с тем же JWT login.

Этот подход заменяет статические токены в CI и выдает короткоживущий Vault token на каждый job.

## Поток аутентификации

1. GitLab Runner выдает job JWT через `id_tokens`.
2. Job вызывает `auth/jwt/login` в Vault с ролью и JWT.
3. Vault проверяет issuer/audience/claims и выдает временный token.
4. Job читает секреты по policy роли.

## Предварительная настройка Vault JWT auth

Включить auth method (один раз):

```bash
vault auth enable jwt
```

Настроить JWT backend под ваш GitLab:

```bash
vault write auth/jwt/config \
  bound_issuer="https://gitlab.example.internal" \
  oidc_discovery_url="https://gitlab.example.internal"
```

> Для self-managed GitLab используйте ваш реальный URL инстанса.

## Создание роли для GitLab CI

### Базовый пример

```bash
vault write auth/jwt/role/gitlab-example - <<'EOF'
{
  "role_type": "jwt",
  "policies": ["gitlab-example"],
  "ttl": "30m",
  "token_explicit_max_ttl": "60m",
  "user_claim": "user_email",
  "bound_audiences": ["https://vault.test.internal"],
  "bound_claims_type": "glob",
  "bound_claims": {
    "project_path": "subgroup/*"
  }
}
EOF
```

### Важные параметры роли

- `role_type` - тип роли (`jwt` для GitLab ID token).
- `policies` - список Vault policy, которые получит CI job.
- `ttl` - начальный TTL Vault token (рекомендуется короткий, например `15m-30m`).
- `token_explicit_max_ttl` - жесткий верхний TTL (ограничивает продление).
- `user_claim` - claim, отображаемый в alias сущности (`user_email`, `user_login`).
- `bound_audiences` - допустимый `aud` в JWT; должен совпадать с `aud` в `id_tokens`.
- `bound_claims_type` - тип сравнения claim (`string` или `glob`).
- `bound_claims` - ограничение роли по claim (проект, ветка, окружение, ref type и т.д.).
- `token_policies` - альтернативный способ задать policy (часто используют вместо `policies` в новых конфигурациях).
- `token_num_uses` - лимит количества операций токена (доп. hardening).
- `token_no_default_policy` - отключить добавление default policy.

## Примеры `bound_claims`

### Только один проект и default ветка

```bash
vault write auth/jwt/role/gitlab-main - <<'EOF'
{
  "role_type": "jwt",
  "policies": ["gitlab-main-read"],
  "bound_audiences": ["https://vault.test.internal"],
  "bound_claims_type": "string",
  "bound_claims": {
    "project_path": "platform/app-config",
    "ref": "main",
    "ref_type": "branch"
  }
}
EOF
```

### Все проекты подгруппы, только теги релиза

```bash
vault write auth/jwt/role/gitlab-release - <<'EOF'
{
  "role_type": "jwt",
  "policies": ["gitlab-release"],
  "bound_audiences": ["https://vault.test.internal"],
  "bound_claims_type": "glob",
  "bound_claims": {
    "project_path": "subgroup/*",
    "ref_type": "tag",
    "ref": "v*"
  }
}
EOF
```

### Только protected refs

```bash
vault write auth/jwt/role/gitlab-protected - <<'EOF'
{
  "role_type": "jwt",
  "policies": ["gitlab-protected-read"],
  "bound_audiences": ["https://vault.test.internal"],
  "bound_claims_type": "string",
  "bound_claims": {
    "project_path": "subgroup/critical-service",
    "ref_protected": "true"
  }
}
EOF
```

## Вариант 1: GitLab CI + Vault API (без бинарника)

```yaml
stages: [test]

read_secret_via_api:
  stage: test
  image: alpine:3.20
  id_tokens:
    VAULT_ID_TOKEN:
      aud: https://vault.test.internal
  variables:
    VAULT_ADDR: https://vault.test.internal
    VAULT_AUTH_PATH: jwt
    VAULT_ROLE: gitlab-example
    SECRET_PATH: secret/data/ci/example
  before_script:
    - apk add --no-cache curl jq
  script:
    - |
      LOGIN_PAYLOAD="$(jq -nc \
        --arg role "$VAULT_ROLE" \
        --arg jwt "$VAULT_ID_TOKEN" \
        '{role:$role,jwt:$jwt}')"
    - |
      VAULT_TOKEN="$(curl -sS --fail \
        -H "Content-Type: application/json" \
        --request POST \
        --data "$LOGIN_PAYLOAD" \
        "$VAULT_ADDR/v1/auth/$VAULT_AUTH_PATH/login" \
        | jq -r '.auth.client_token')"
    - test -n "$VAULT_TOKEN" && test "$VAULT_TOKEN" != "null"
    - |
      curl -sS --fail \
        -H "X-Vault-Token: $VAULT_TOKEN" \
        "$VAULT_ADDR/v1/$SECRET_PATH" \
        | jq -r '.data.data'
```

## Вариант 2: GitLab CI + Vault CLI (бинарник)

```yaml
stages: [test]

read_secret_via_cli:
  stage: test
  image: hashicorp/vault:1.16
  id_tokens:
    VAULT_ID_TOKEN:
      aud: https://vault.test.internal
  variables:
    VAULT_ADDR: https://vault.test.internal
    VAULT_AUTH_PATH: jwt
    VAULT_ROLE: gitlab-example
  script:
    - export VAULT_TOKEN="$(vault write -field=token auth/$VAULT_AUTH_PATH/login role="$VAULT_ROLE" jwt="$VAULT_ID_TOKEN")"
    - vault kv get -format=json secret/ci/example | jq -r '.data.data'
```

## Минимальная policy для примеров

```hcl
path "secret/data/ci/example" {
  capabilities = ["read"]
}
```

## Рекомендации по безопасности

1. Используйте минимальные `bound_claims` (project/ref/ref_type/ref_protected).
2. Выставляйте короткий TTL и max TTL.
3. Не логируйте JWT и Vault token в job output.
4. Разделяйте роли по окружениям (`dev/stage/prod`) и проектам.
5. Храните секреты в `kv-v2` и ограничивайте path policy.

## Диагностика

```bash
# Проверить конфигурацию JWT backend
vault read auth/jwt/config

# Проверить роль
vault read auth/jwt/role/gitlab-example

# Проверить claims у JWT локально (без валидации подписи)
python3 - <<'PY'
import os, json, base64
t=os.environ["VAULT_ID_TOKEN"].split(".")[1]
t += "=" * ((4-len(t)%4)%4)
print(json.dumps(json.loads(base64.urlsafe_b64decode(t)), indent=2))
PY
```

## Связанные заметки

- [[AppRole Management]] - Альтернативная machine-to-machine аутентификация в Vault
- [[../Jenkins/Using Credentials in Active Choice Parameter]] - Практики работы с секретами в CI
