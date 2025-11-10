---
created: 2025-01-27
tags:
  - jenkins
  - grype
  - security
  - vulnerability-scanning
  - agents
  - nexus
category: automation
---

# Конфигурация Grype для сканирования уязвимостей на Jenkins агентах

## Введение

Существуют ситуации, когда сборка приложений происходит непосредственно на сборочных агентах (вне кластера Kubernetes). В целях анализа возможных проблем безопасности следует регулярно проводить сканирование ОС на предмет возможных уязвимостей.

В данной статье описывается процесс конфигурации инструмента поиска уязвимостей [Grype](https://github.com/anchore/grype).

## Требования

- Необходимо, чтобы на сборочных агентах присутствовал доступ до [Nexus](https://nexus.example.local/)

## Установка Grype

### DEB-based системы (Debian/Ubuntu)

#### Шаг 1: Получение ссылки на пакет

Скопируйте ссылку на актуальную версию .deb пакета из репозитория [grype](https://nexus.example.local/#browse/browse:grype).

> **Примечание:**
> Это внутренний Nexus Hosted Raw Repository, который содержит лишь те версии, которые были загружены руками. Чтобы убедиться, что в репозитории корректная версия, сравните её с актуальными релизами из официальной поставки: [https://github.com/anchore/grype/releases](https://github.com/anchore/grype/releases).
> 
> Если есть более актуальные версии, то загрузите их в Nexus самостоятельно. Если у вас отсутствуют права на загрузку файлов, то необходимо обратиться к команде DevOps-инженеров через соответствующий тикет в Jira.

#### Шаг 2: Загрузка пакета

```bash
wget ${DOWNLOAD_URL}
```

**DOWNLOAD_URL** - ссылка, которую вы получили на предыдущем этапе. 

**Пример:** `https://nexus.example.local/repository/grype/0.77.4/grype_0.77.4_linux_amd64.deb`

#### Шаг 3: Установка пакета

```bash
# через apt-get
sudo apt-get install -y ./${PACKAGE_NAME}

# через apt
sudo apt install -y ./${PACKAGE_NAME}

# через dpkg
sudo dpkg -i ./${PACKAGE_NAME}
```

**PACKAGE_NAME** - имя пакета, который был скачан на предыдущем этапе. 

**Пример:** `grype_0.77.4_linux_amd64.deb`

#### Шаг 4: Проверка установки

```bash
grype --version
# пример корректного вывода
# grype 0.77.4
```

#### Шаг 5: Удаление файла пакета

```bash
rm -rf ./${PACKAGE_NAME}
```

**PACKAGE_NAME** - имя пакета, который был скачан на предыдущем этапе. 

**Пример:** `grype_0.77.4_linux_amd64.deb`

### RPM-based системы (CentOS/RHEL/RedOS)

#### Шаг 1: Получение ссылки на пакет

Скопируйте ссылку на актуальную версию .rpm пакета из репозитория [grype](https://github.com/anchore/grype/releases)
#### Шаг 2: Загрузка пакета

```bash
wget ${DOWNLOAD_URL}
```

**DOWNLOAD_URL** - ссылка, которую вы получили на предыдущем этапе. 

**Пример:** `https://nexus.example.local/repository/grype/0.77.4/grype_0.77.4_linux_amd64.rpm`

#### Шаг 3: Установка пакета

```bash
# через yum
sudo yum install -y ./${PACKAGE_NAME}

# через dnf
sudo dnf install -y ./${PACKAGE_NAME}

# через rpm
sudo rpm -i ./${PACKAGE_NAME}
```

**PACKAGE_NAME** - имя пакета, который был скачан на предыдущем этапе. 

**Пример:** `grype_0.77.4_linux_amd64.rpm`

#### Шаг 4: Проверка установки

```bash
grype --version
# пример корректного вывода
# grype 0.77.4
```

#### Шаг 5: Удаление файла пакета

```bash
rm -rf ./${PACKAGE_NAME}
```

**PACKAGE_NAME** - имя пакета, который был скачан на предыдущем этапе. 

**Пример:** `grype_0.77.4_linux_amd64.rpm`

## Конфигурация grype.yaml

Для дальнейшей работы `grype` необходимо настроить ряд опций, путем создания конфигурационного файла `.grype.yaml`.

Далее приведена минимальная конфигурация, которая позволяет использовать внутренний Nexus Proxy Repository ([grype-db](https://nexus.example.local/#browse/browse:grype-db)) для актуализации БД уязвимостей.

### Пример конфигурационного файла

```yaml
check-for-app-update: false

match-upstream-kernel-headers: false

db:
  auto_update: true
  update-url: "https://nexus.example.local/repository/grype/databases/listing.json"
  validate-age: true
  cache-dir: /opt/grype-cache
```

### Описание параметров

- **check-for-app-update** - проверка обновлений приложения
- **match-upstream-kernel-headers** - сопоставление с upstream заголовками ядра
- **db.auto_update** - автоматическое обновление базы данных
- **db.update-url** - URL для обновления базы данных (внутренний Nexus)
- **db.validate-age** - проверка возраста базы данных
- **db.cache-dir** - каталог для кэша базы данных

Описание текущих и возможных опций можно посмотреть на странице [официальной документации](https://github.com/anchore/grype).

### Расположение конфигурационного файла

Для того, чтобы `grype` использовал данный конфигурационный файл необходимо разместить его по одному из следующих путей:

- `.grype.yaml`
- `.grype/config.yaml`
- `~/.grype.yaml`
- `<XDG_CONFIG_HOME>/grype/config.yaml`

Либо выбрать путь на ваше усмотрение и использовать аргумент `-c (--config)` при запуске сканирования.

**Пример:**

```bash
grype dir:./ --distro centos:7 --config /opt/grype/.grype
```

## Синтаксис

Синтаксис инструмента, возможные аргументы и типы сканирования можно посмотреть на странице [официальной документации](https://github.com/anchore/grype).

## Использование в Jenkins Pipeline

### Выдача необходимых прав

В случае, если ваш Jenkins slave agent подключен к Jenkins master'у не под учетной записью root (суперпользователь) и он не состоит в группе sudo, требуется выдать ему привилегии на запуск grype от имени суперпользователя. В противном случае сканирование будет невозможно.

Выдать права можно путем добавления (редактирования) файла `/etc/sudoers.d/${USERNAME}`, где **USERNAME** - имя пользователя под которым был подключен сборочный агент.

**Пример конфигурации:**

```
jenkins ALL = NOPASSWD: /bin/grype
```

> **Примечание:**
> Данная строка позволит пользователю jenkins запускать `grype` с любыми аргументами и флагами. Если необходимо ограничить перечень возможных вызовов, требуется указать конкретную команду.
> 
> **Пример:** `/bin/grype dir:./ --distro centos:7 --config /opt/grype/.grype`

**ВАЖНО!**

В примере указан абсолютный путь до бинарного файла `grype`, данный путь может отличаться в зависимости от ОС.

Чтобы узнать корректный путь, требуется выполнить команду:

```bash
which grype
# пример вывода
# /usr/bin/grype
```

### Создание grype.yaml с помощью Config File Provider

Как говорилось ранее для корректной работы `grype` требуется конфигурационный файл `grype.yaml`.

Если нет возможности или желания создать данный файл непосредственно на ОС (по инструкции выше), то можно создать его в Jenkins. Для этого используется плагин [Config File Provider](https://plugins.jenkins.io/config-file-provider/).

#### Шаги настройки:

1. Перейти в **Manage Jenkins → Managed Files → Add a new Config**
2. Выбрать тип **Custom File**
3. В поле **ID** указать желаемый id файла (либо оставить без изменений)
4. Нажать **Next**
5. По желанию: Указать имя (**Name**) и комментарий (**Comment**)
6. Заполнить файл следующим содержимым:

```yaml
check-for-app-update: false

match-upstream-kernel-headers: false

db:
  auto_update: true
  update-url: "https://nexus.example.local/repository/grype/databases/listing.json"
  validate-age: true
  cache-dir: /opt/grype-cache
```

7. Нажать **Submit**

**Пример конфигурационного файла:**

![Image: Config File Provider пример](Attachments/Jenkins/grype-config-file-provider.png)

*Примечание: Замените изображение на актуальное*

### Использование в пайплайне

Выдержка из пайплайна, который запускает поиск уязвимостей:

```groovy
def os = 'ubuntu'
def distro = 'ubuntu:20.04'

pipeline {
    agent { 
        label "DevOps-dev" 
    }
    stages {
        stage("Get .grype.yaml config file") {
            steps {
                script {
                    configFileProvider([configFile(fileId: 'grype-config', variable: 'TMP_FILE_NAME')]) {
                        sh('cp -f $TMP_FILE_NAME ./grype.yaml')
                    }
                }
            }
        }
        stage("Scan rootfs for agent") {
            steps {
                script {
                    sh("sudo grype dir:/ --distro ${distro} --config ./grype.yaml --output json --file grype-${os}.json")
                }
            }
        }
        stage("Publish grype results for agent") {
            steps {
                script {
                    recordIssues(tools: [grype(id: "grype-${os}", name: "Grype: ${os}", pattern: "**/grype-${os}.json")])
                    archiveArtifacts allowEmptyArchive: true, artifacts: "grype-${os}.json", fingerprint: true
                }
            }
        }
    }
    post {
        cleanup {
            cleanWs()
        }
    }
}
```

**Примечание:** Это тестовый пример.

## Пояснение к примеру пайплайна

### Этап 1: Получение конфигурации

```groovy
configFileProvider([configFile(fileId: 'grype-config', variable: 'TMP_FILE_NAME')]) {
    sh('cp -f $TMP_FILE_NAME ./grype.yaml')
}
```

- Использует Config File Provider для получения файла конфигурации
- Копирует конфигурацию в рабочую директорию

### Этап 2: Сканирование

```groovy
sh("sudo grype dir:/ --distro ${distro} --config ./grype.yaml --output json --file grype-${os}.json")
```

- **`dir:/`** - сканирование корневой файловой системы
- **`--distro`** - указание дистрибутива ОС
- **`--config`** - путь к конфигурационному файлу
- **`--output json`** - вывод результатов в JSON формате
- **`--file`** - сохранение результатов в файл

### Этап 3: Публикация результатов

```groovy
recordIssues(tools: [grype(id: "grype-${os}", name: "Grype: ${os}", pattern: "**/grype-${os}.json")])
archiveArtifacts allowEmptyArchive: true, artifacts: "grype-${os}.json", fingerprint: true
```

- **recordIssues** - публикация результатов в Jenkins UI (требует плагин Warnings Next Generation)
- **archiveArtifacts** - архивирование JSON отчета

## Полезные команды

### Проверка подключения к Nexus

```bash
curl -I https://nexus.example.local/repository/grype/databases/listing.json
```

### Ручное обновление базы данных

```bash
sudo grype db update --config /path/to/grype.yaml
```

### Проверка версии базы данных

```bash
sudo grype db status --config /path/to/grype.yaml
```

### Сканирование конкретной директории

```bash
sudo grype dir:/opt/myapp --distro ubuntu:20.04 --config ./grype.yaml
```

## Best Practices

1. **Регулярное обновление базы данных** - настройте автоматическое обновление
2. **Используйте внутренний Nexus** - для быстрого доступа к базе данных
3. **Кэширование** - настройте cache-dir для ускорения работы
4. **Публикация результатов** - используйте recordIssues для визуализации
5. **Архивирование отчетов** - сохраняйте JSON отчеты для анализа трендов
6. **Права доступа** - настройте минимально необходимые права sudo

## Troubleshooting

### Ошибка доступа к Nexus

```
Error: failed to fetch vulnerability database
```

**Решение:**
- Проверьте доступность Nexus: `curl -I https://nexus.example.local`
- Проверьте настройки прокси (если используется)
- Убедитесь, что URL в конфигурации корректный

### Ошибка прав доступа

```
Permission denied
```

**Решение:**
- Проверьте настройки sudoers
- Убедитесь, что пользователь может выполнять `sudo grype --version`
- Проверьте права на каталог кэша

### База данных не обновляется

**Решение:**
- Проверьте `db.auto_update: true` в конфигурации
- Выполните ручное обновление: `grype db update`
- Проверьте доступность update-url

## Связанные заметки

- [[Security/Security Tools]] - Другие инструменты безопасности (Trivy)
- [[Nexus Component Download URL]] - Получение ссылок на компоненты из Nexus
- [[Pipeline Scripts]] - Другие полезные скрипты для Jenkins

