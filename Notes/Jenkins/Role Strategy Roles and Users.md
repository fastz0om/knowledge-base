---
created: 2025-01-27
tags:
  - jenkins
  - role-strategy
  - rbac
  - authorization
  - groovy
category: automation
---

# Получение списка ролей и назначенных пользователей в Role Strategy

## Обзор

Функция для получения полного перечня ролей и пользователей/групп, которым присвоена указанная роль в Jenkins с использованием плагина Role-Based Authorization Strategy.

## Функция для получения информации

### Полный скрипт

```groovy
import com.michelin.cio.hudson.plugins.rolestrategy.*
import jenkins.model.Jenkins

def authStrategy = Jenkins.instance.getAuthorizationStrategy()

if (!(authStrategy instanceof RoleBasedAuthorizationStrategy)) {
    println "⚠️ Текущая стратегия авторизации не является Role-Based Strategy."
    return
}

def rbas = (RoleBasedAuthorizationStrategy) authStrategy

// Перечень доменов: globalRoles, projectRoles, slaveRoles и т.п.
def roleMaps = [
    "globalRoles"  : rbas.getRoleMap(com.synopsys.arc.jenkins.plugins.rolestrategy.RoleType.Global),
    "projectRoles" : rbas.getRoleMap(com.synopsys.arc.jenkins.plugins.rolestrategy.RoleType.Project),
    "agentRoles"   : rbas.getRoleMap(com.synopsys.arc.jenkins.plugins.rolestrategy.RoleType.Slave)
]

roleMaps.each { domain, roleMap ->
    println "\n==== 📁 Domain: ${domain} ===="
    roleMap.getRoles().each { role ->
        def roleName = role.getName()
        def permissions = role.getPermissions()*.id.sort()
        def assignedUsers = roleMap.getSidsForRole(roleName).sort()
         
        println "\n🔹 Role: '${roleName}'"
        println "   ├─ Permissions:"
        permissions.each { p -> println "   │   - ${p}" }
        println "   └─ Assigned Users/Groups:"
        assignedUsers.each { u -> println "       - ${u}" }
    }
}
```

## Использование

### Запуск в Script Console

1. Открыть: **Jenkins → Manage Jenkins → Script Console**
2. Вставить код функции
3. Нажать **Run**
4. Результат будет выведен в консоль

### Пример вывода

```
==== 📁 Domain: globalRoles ====

🔹 Role: 'admin'
   ├─ Permissions:
   │   - hudson.model.Hudson.Administer
   │   - hudson.model.Item.Configure
   │   - hudson.model.Item.Delete
   └─ Assigned Users/Groups:
       - user1
       - group:developers

🔹 Role: 'developer'
   ├─ Permissions:
   │   - hudson.model.Item.Build
   │   - hudson.model.Item.Read
   └─ Assigned Users/Groups:
       - user2
       - user3

==== 📁 Domain: projectRoles ====
...
```

## Объяснение кода

### Проверка стратегии авторизации

```groovy
if (!(authStrategy instanceof RoleBasedAuthorizationStrategy)) {
    println "⚠️ Текущая стратегия авторизации не является Role-Based Strategy."
    return
}
```

Проверяет, что в Jenkins используется Role-Based Authorization Strategy. Если используется другая стратегия (например, Matrix Authorization Strategy), скрипт завершится с предупреждением.

### Типы ролей

```groovy
def roleMaps = [
    "globalRoles"  : rbas.getRoleMap(RoleType.Global),
    "projectRoles" : rbas.getRoleMap(RoleType.Project),
    "agentRoles"   : rbas.getRoleMap(RoleType.Slave)
]
```

- **Global Roles** - глобальные роли (действуют на весь Jenkins)
- **Project Roles** - роли проектов (действуют на конкретные проекты по паттерну)
- **Agent Roles** - роли агентов (действуют на узлы/агенты)

### Извлечение информации о роли

```groovy
def roleName = role.getName()                    // Имя роли
def permissions = role.getPermissions()*.id.sort()  // Список прав доступа
def assignedUsers = roleMap.getSidsForRole(roleName).sort()  // Назначенные пользователи/группы
```

- **roleName** - имя роли
- **permissions** - список идентификаторов прав (отсортированный)
- **assignedUsers** - список SID (Security Identifier) пользователей и групп

## Модификации функции

### Получение только глобальных ролей

```groovy
import com.michelin.cio.hudson.plugins.rolestrategy.*
import jenkins.model.Jenkins

def rbas = (RoleBasedAuthorizationStrategy) Jenkins.instance.getAuthorizationStrategy()
def globalRoleMap = rbas.getRoleMap(RoleType.Global)

println "==== Global Roles ===="
globalRoleMap.getRoles().each { role ->
    def roleName = role.getName()
    def assignedUsers = globalRoleMap.getSidsForRole(roleName).sort()
    
    println "\nRole: '${roleName}'"
    assignedUsers.each { u -> println "  - ${u}" }
}
```

### Поиск ролей конкретного пользователя

```groovy
import com.michelin.cio.hudson.plugins.rolestrategy.*
import jenkins.model.Jenkins

def targetUser = "user1"  // Имя пользователя
def rbas = (RoleBasedAuthorizationStrategy) Jenkins.instance.getAuthorizationStrategy()

def roleMaps = [
    "globalRoles"  : rbas.getRoleMap(RoleType.Global),
    "projectRoles" : rbas.getRoleMap(RoleType.Project),
    "agentRoles"   : rbas.getRoleMap(RoleType.Slave)
]

println "==== Roles for user: ${targetUser} ===="
roleMaps.each { domain, roleMap ->
    roleMap.getRoles().each { role ->
        def roleName = role.getName()
        def assignedUsers = roleMap.getSidsForRole(roleName)
        
        if (assignedUsers.contains(targetUser)) {
            println "${domain}: ${roleName}"
        }
    }
}
```

### Поиск пользователей с конкретным правом

```groovy
import com.michelin.cio.hudson.plugins.rolestrategy.*
import jenkins.model.Jenkins

def targetPermission = "hudson.model.Item.Delete"  // Имя права
def rbas = (RoleBasedAuthorizationStrategy) Jenkins.instance.getAuthorizationStrategy()

def roleMaps = [
    "globalRoles"  : rbas.getRoleMap(RoleType.Global),
    "projectRoles" : rbas.getRoleMap(RoleType.Project),
    "agentRoles"   : rbas.getRoleMap(RoleType.Slave)
]

println "==== Users with permission: ${targetPermission} ===="
def usersWithPermission = [] as Set

roleMaps.each { domain, roleMap ->
    roleMap.getRoles().each { role ->
        def permissions = role.getPermissions()*.id
        if (permissions.contains(targetPermission)) {
            def roleName = role.getName()
            def assignedUsers = roleMap.getSidsForRole(roleName)
            usersWithPermission.addAll(assignedUsers)
        }
    }
}

usersWithPermission.sort().each { u -> println "  - ${u}" }
```

### Экспорт в форматированный список

```groovy
import com.michelin.cio.hudson.plugins.rolestrategy.*
import jenkins.model.Jenkins

def rbas = (RoleBasedAuthorizationStrategy) Jenkins.instance.getAuthorizationStrategy()
def globalRoleMap = rbas.getRoleMap(RoleType.Global)

def output = []
output.add("=== Jenkins Roles and Users ===")
output.add("Generated: ${new Date()}")

globalRoleMap.getRoles().each { role ->
    def roleName = role.getName()
    def assignedUsers = globalRoleMap.getSidsForRole(roleName).sort()
    
    output.add("\nRole: ${roleName}")
    assignedUsers.each { u -> output.add("  - ${u}") }
}

// Вывод или сохранение
output.each { println it }
// Или сохранить в файл:
// new File('/tmp/jenkins-roles.txt').text = output.join('\n')
```

## Важные замечания

### Требования

- Плагин **Role-based Authorization Strategy** должен быть установлен и активен
- Текущая стратегия авторизации должна быть Role-Based Authorization Strategy

### Безопасность

- Скрипт работает в Script Console
- Требуются права на выполнение скриптов (обычно только для администраторов)
- Результаты могут содержать чувствительную информацию о правах доступа

### Ограничения

- Script Console имеет ограничения песочницы в некоторых конфигурациях
- Может потребоваться одобрение методов в **Manage Jenkins → In-process Script Approval**

## Troubleshooting

### Ошибка: "No such property: RoleBasedAuthorizationStrategy"

**Решение:** Убедитесь, что плагин Role-based Authorization Strategy установлен:
- Manage Jenkins → Manage Plugins → Installed plugins
- Проверьте наличие плагина

### Ошибка: "Current authorization strategy is not Role-Based"

**Решение:** Измените стратегию авторизации:
- Manage Jenkins → Configure Global Security
- Authorization → выберите "Role-Based Strategy"

### Скрипт не выполняется

**Решение:**
1. Проверьте права доступа (нужны права администратора)
2. Проверьте одобрение методов в Script Approval
3. Проверьте логи Jenkins на наличие ошибок

## Полезные ссылки

- [Role-based Authorization Strategy Plugin](https://plugins.jenkins.io/role-strategy/)
- [Jenkins Script Console Documentation](https://www.jenkins.io/doc/book/managing/script-console/)

## Связанные заметки

- [[Pipeline Scripts]] - Работа с Jenkins pipelines
- [[Role]] - Роли в Kubernetes (для сравнения концепций)

