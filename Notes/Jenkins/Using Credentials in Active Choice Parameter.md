---
created: 2025-01-27
tags:
  - jenkins
  - active-choice-parameter
  - credentials
  - api
  - groovy
  - vault
category: automation
---

# Использование секретов Jenkins в Active Choice Parameter

## Введение

Существуют ситуации, в которых необходимо динамически формировать перечень элементов при использовании Active Choice Parameter (включая его разновидности).

**Примеры использования:**
- Формирование перечня актуальных версий артефактов Java
- Перечень проектов из GitLab
- Список пространств из Confluence
- И другие динамические списки

Вся задача сводится к взаимодействию с API, для которого часто необходимо использовать токены или учетные данные. Хардкодить их - очень плохо.

Далее продемонстрирован пример получения ранее созданных секретов Jenkins с помощью использования метода `lookupCredentials` (подробнее в [JavaDoc](https://javadoc.jenkins.io/plugin/credentials/com/cloudbees/plugins/credentials/CredentialsProvider.html)).

## Полный пример

```groovy
properties([
    parameters ([
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_CHECKBOX',
            description: 'Укажите перечень тегов, которые необходимо собрать\nНазвания тегов формируются из названий Dockerfile, которые находятся в git-репозитории containers',
            filterLength: 1,
            filterable: false,
            name: 'Tags',
            script: [
                $class: 'GroovyScript',
                fallbackScript: [
                    classpath: [],
                    oldScript: '',
                    sandbox: false,
                    script: 'return ["error"]'
                ],
                script: [
                    classpath: [],
                    oldScript: '',
                    sandbox: false,
                    script: '''
                        import jenkins.model.*
                        import groovy.json.*

                        def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
                                com.datapipe.jenkins.vault.credentials.common.VaultStringCredential.class,
                                Jenkins.instance,
                                null,
                                null
                        );
                        def credential = jenkinsCredentials.findResult { it.id == 'git-token' ? it : null }
                        def gitlab_token = credential.secret

                        def api = 'https://gitlab.example.local/api/v4/projects/2125/repository/tree'
                        def conn = new URL(api).openConnection()
                        conn.setRequestProperty("PRIVATE-TOKEN", gitlab_token.toString())
                        def json = new groovy.json.JsonSlurper().parseText(conn.content.text)

                        def images = []
                        json.each{ elem ->
                            if(elem.path.contains('.dockerfile')){
                                images += elem.path.split('.dockerfile')[0]
                            }
                        }
                        return images
                    '''
                ]
            ]
        ]
    ])
])
```

## Реализация

Пример направлен на получение списка файлов репозитория с помощью API GitLab для формирования списка значений в Choice Parameter.

### Получение секрета

Ключевая часть для получения секрета из Jenkins:

```groovy
import jenkins.model.*
import groovy.json.*

def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.datapipe.jenkins.vault.credentials.common.VaultStringCredential.class,
        Jenkins.instance,
        null,
        null
);
def credential = jenkinsCredentials.findResult { it.id == 'git-token' ? it : null }
def gitlab_token = credential.secret
```

### Пошаговое объяснение

1. **В метод `lookupCredentials`** первым аргументом необходимо передать интересующий класс секрета
2. **В `findResult`** необходимо передать ID секрета (заменить `git-token` на ваш ID)

### Выбор класса секрета

В данном примере используется `com.datapipe.jenkins.vault.credentials.common.VaultStringCredential.class`. 

Поскольку он унаследован от стандартных классов Jenkins, можно использовать альтернативный класс: `org.jenkinsci.plugins.plaincredentials.StringCredentials.class`.

Токен хранится в **Vault Secret Text Credential** с названием `git-token`.

## Соответствие типов секретов и классов

| Название секрета | Название класса (включая аналоги) | Как получить данные |
|-----------------|-----------------------------------|---------------------|
| **Vault Secret Text Credential** | `com.datapipe.jenkins.vault.credentials.common.VaultStringCredential.class`<br>`org.jenkinsci.plugins.plaincredentials.StringCredentials.class` | `credential.secret` |
| **Vault Username-Password Credential** | `com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class` | `credential.username`<br>`credential.password` |
| **Vault SSH Username with private key Credential** | `com.datapipe.jenkins.vault.credentials.common.VaultSSHUserPrivateKey.class` | `credential.username`<br>`credential.passphrase`<br>`credential.privateKey`<br>`credential.privateKeys` |
| **Vault Secret File Credential** | `com.datapipe.jenkins.vault.credentials.common.VaultFileCredential.class`<br>`org.jenkinsci.plugins.plaincredentials.FileCredentials.class` | `credential.fileName`<br>`credential.content` (поток байтов) |
| **Username with Password** | `com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class` | `credential.username`<br>`credential.password` |
| **Secret text** | `org.jenkinsci.plugins.plaincredentials.StringCredentials.class` | `credential.secret` |
| **Secret file** | `org.jenkinsci.plugins.plaincredentials.FileCredentials.class` | `credential.fileName`<br>`credential.content` (поток байтов) |

## Поиск нужного класса

Если вам необходим другой тип учетных данных, нужно найти соответствующий класс:

1. Используйте Google для поиска класса
2. Проверьте исходники плагинов в GitHub
3. Используйте Jenkins Script Console для проверки доступных методов

**Пример поиска:** Название класса из примера было найдено в исходниках плагина [hashicorp-vault-plugin](https://github.com/jenkinsci/hashicorp-vault-plugin/tree/master).

![Image: Исходники hashicorp-vault-plugin](Attachments/Jenkins/hashicorp-vault-plugin-screen.png)

## Примеры использования разных типов секретов

### Secret Text

```groovy
def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        org.jenkinsci.plugins.plaincredentials.StringCredentials.class,
        Jenkins.instance,
        null,
        null
);
def credential = jenkinsCredentials.findResult { it.id == 'api-token' ? it : null }
def token = credential.secret
```

### Username-Password

```groovy
def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class,
        Jenkins.instance,
        null,
        null
);
def credential = jenkinsCredentials.findResult { it.id == 'db-credentials' ? it : null }
def username = credential.username
def password = credential.password
```

### SSH Private Key

```groovy
def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.datapipe.jenkins.vault.credentials.common.VaultSSHUserPrivateKey.class,
        Jenkins.instance,
        null,
        null
);
def credential = jenkinsCredentials.findResult { it.id == 'ssh-key' ? it : null }
def sshUser = credential.username
def privateKey = credential.privateKey
def passphrase = credential.passphrase
```

## Полный пример: Получение проектов из GitLab

```groovy
properties([
    parameters([
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_RADIO',
            description: 'Выберите проект для сборки',
            name: 'Project',
            script: [
                $class: 'GroovyScript',
                script: [
                    script: '''
                        import jenkins.model.*
                        import groovy.json.*

                        // Получение токена из секретов
                        def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
                                org.jenkinsci.plugins.plaincredentials.StringCredentials.class,
                                Jenkins.instance,
                                null,
                                null
                        );
                        def credential = jenkinsCredentials.findResult { it.id == 'gitlab-token' ? it : null }
                        def gitlab_token = credential.secret

                        // Запрос к GitLab API
                        def api = 'https://gitlab.example.local/api/v4/projects'
                        def conn = new URL(api).openConnection()
                        conn.setRequestProperty("PRIVATE-TOKEN", gitlab_token.toString())
                        def json = new groovy.json.JsonSlurper().parseText(conn.content.text)

                        // Формирование списка проектов
                        def projects = []
                        json.each { project ->
                            projects.add(project.name)
                        }
                        return projects.sort()
                    '''
                ]
            ]
        ]
    ])
])
```

## Примечания

### 1. Script Approval

Данный способ использует значительное количество функционала, который заблокирован в песочнице Jenkins. Поэтому необходимо одобрить методы в **Manage Jenkins → In-process Script Approval**.

**Методы, которые нужно одобрить:**
- `method com.cloudbees.plugins.credentials.CredentialsProvider lookupCredentials`
- `method jenkins.model.Jenkins getInstance`
- Методы работы с секретами (getSecret, getUsername, getPassword и т.д.)

### 2. Альтернативные классы

Если используется Vault плагин, можно использовать классы:
- `com.datapipe.jenkins.vault.credentials.common.*` (специфичные для Vault)
- Стандартные классы `org.jenkinsci.plugins.plaincredentials.*` (работают и для Vault, и для обычных секретов)

### 3. Безопасность

- Никогда не выводите секреты в логи
- Используйте правильный scope для секретов (Global или конкретный job)
- Регулярно ротируйте секреты

## Best Practices

1. **Используйте lookupCredentials правильно** - указывайте корректный класс для типа секрета
2. **Проверяйте наличие секрета** - добавляйте проверку на null
3. **Обрабатывайте ошибки** - используйте fallbackScript в Active Choice Parameter
4. **Кэшируйте результаты** - для снижения нагрузки на API
5. **Логируйте безопасно** - не логируйте секреты

### Пример с обработкой ошибок

```groovy
def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        org.jenkinsci.plugins.plaincredentials.StringCredentials.class,
        Jenkins.instance,
        null,
        null
);

def credential = jenkinsCredentials.findResult { it.id == 'git-token' ? it : null }

if (!credential) {
    return ["Ошибка: Секрет не найден"]
}

def token = credential.secret
// ... использование токена
```

## Связанные заметки

- [[Active Choice Parameter Logging]] - Настройка логгирования для отладки Active Choice Parameter
- [[Pipeline Scripts]] - Работа с Jenkins pipelines
- [[Role Strategy Roles and Users]] - Управление правами доступа

