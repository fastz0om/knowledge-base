---
created: 2025-01-27
tags:
  - prometheus
  - windows
  - monitoring
  - exporter
  - windows-exporter
category: monitoring
---

# Windows Exporter для Prometheus

## Описание

Windows Exporter - это экспортер метрик для Windows систем, который позволяет собирать и экспортировать метрики в формате Prometheus.

## Скачивание и установка

### Получение исполняемого файла

Для начала нам потребуется сам исполняемый файл Prometheus Windows Exporter, его можно достать отсюда:

- **Реleases:** [https://github.com/prometheus-community/windows_exporter/releases](https://github.com/prometheus-community/windows_exporter/releases)
- **Документация:** [https://github.com/prometheus-community/windows_exporter](https://github.com/prometheus-community/windows_exporter)

**Важно:** Нам нужен именно **EXE файл под 64 бита**, не MSI.

### Пример названия файла

```
windows_exporter-X.X.X-amd64.exe
```

где `X.X.X` - версия экспортера.

## Коллекторы

Windows Exporter поддерживает множество коллекторов для сбора различных метрик. Ниже представлена полная таблица доступных коллекторов:

| Name | Description | Enabled by default |
|------|-------------|-------------------|
| [ad](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.ad.md) | Active Directory Domain Services | |
| [adcs](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.adcs.md) | Active Directory Certificate Services | |
| [adfs](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.adfs.md) | Active Directory Federation Services | |
| [cache](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.cache.md) | Cache metrics | |
| [cpu](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.cpu.md) | CPU usage | ✓ |
| [cpu_info](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.cpu_info.md) | CPU Information | |
| [cs](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.cs.md) | "Computer System" metrics (system properties, num cpus/total memory) | ✓ |
| [container](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.container.md) | Container metrics | |
| [dfsr](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.dfsr.md) | DFSR metrics | |
| [dhcp](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.dhcp.md) | DHCP Server | |
| [dns](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.dns.md) | DNS Server | |
| [exchange](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.exchange.md) | Exchange metrics | |
| [fsrmquota](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.fsrmquota.md) | Microsoft File Server Resource Manager (FSRM) Quotas collector | |
| [hyperv](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.hyperv.md) | Hyper-V hosts | |
| [iis](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.iis.md) | IIS sites and applications | |
| [logical_disk](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.logical_disk.md) | Logical disks, disk I/O | ✓ |
| [logon](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.logon.md) | User logon sessions | |
| [memory](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.memory.md) | Memory usage metrics | |
| [mscluster_cluster](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mscluster_cluster.md) | MSCluster cluster metrics | |
| [mscluster_network](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mscluster_network.md) | MSCluster network metrics | |
| [mscluster_node](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mscluster_node.md) | MSCluster Node metrics | |
| [mscluster_resource](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mscluster_resource.md) | MSCluster Resource metrics | |
| [mscluster_resourcegroup](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mscluster_resourcegroup.md) | MSCluster ResourceGroup metrics | |
| [msmq](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.msmq.md) | MSMQ queues | |
| [mssql](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.mssql.md) | [SQL Server Performance Objects](https://docs.microsoft.com/en-us/sql/relational-databases/performance-monitor/use-sql-server-objects#SQLServerPOs) metrics | |
| [netframework_clrexceptions](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrexceptions.md) | .NET Framework CLR Exceptions | |
| [netframework_clrinterop](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrinterop.md) | .NET Framework Interop Metrics | |
| [netframework_clrjit](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrjit.md) | .NET Framework JIT metrics | |
| [netframework_clrloading](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrloading.md) | .NET Framework CLR Loading metrics | |
| [netframework_clrlocksandthreads](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrlocksandthreads.md) | .NET Framework locks and metrics threads | |
| [netframework_clrmemory](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrmemory.md) | .NET Framework Memory metrics | |
| [netframework_clrremoting](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrremoting.md) | .NET Framework Remoting metrics | |
| [netframework_clrsecurity](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.netframework_clrsecurity.md) | .NET Framework Security Check metrics | |
| [net](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.net.md) | Network interface I/O | ✓ |
| [os](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.os.md) | OS metrics (memory, processes, users) | ✓ |
| [process](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.process.md) | Per-process metrics | |
| [remote_fx](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.remote_fx.md) | RemoteFX protocol (RDP) metrics | |
| [scheduled_task](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.scheduled_task.md) | Scheduled Tasks metrics | |
| [service](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.service.md) | Service state metrics | ✓ |
| [smtp](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.smtp.md) | IIS SMTP Server | |
| [system](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.system.md) | System calls | ✓ |
| [tcp](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.tcp.md) | TCP connections | |
| [teradici_pcoip](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.teradici_pcoip.md) | [Teradici PCoIP](https://www.teradici.com/web-help/pcoip_wmi_specs/) session metrics | |
| [time](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.time.md) | Windows Time Service | |
| [thermalzone](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.thermalzone.md) | Thermal information | |
| [terminal_services](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.terminal_services.md) | Terminal services (RDS) | |
| [textfile](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.textfile.md) | Read prometheus metrics from a text file | ✓ |
| [vmware_blast](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.vmware_blast.md) | VMware Blast session metrics | |
| [vmware](https://github.com/prometheus-community/windows_exporter/blob/master/docs/collector.vmware.md) | Performance counters installed by the Vmware Guest agent | |

## Установка

### Шаг 1: Подготовка директории

1. Создайте директорию для экспортера (например, `C:\windows_exporter`)
2. Скопируйте туда исполняемый файл `windows_exporter-X.X.X-amd64.exe`

### Шаг 2: Конфигурационный файл

Для удобства дальнейшей настройки, в папку куда вы положили ваш исполняемый файл экспортера кладем следующую конфигурацию `config.yaml`:

```yaml
---
# Note this is not an exhaustive list of all configuration values
collectors:
  enabled: cpu,cs,logical_disk,net,os,service,system,ad,exchange,dns #коллекторы которые вам нужны
collector:
  service:
    services-where: Name='windows_exp' #Информация по вашему сервису (тут должно быть указано то как вы его назовете при создании)
log:
  level: warn #Уровень логирования
scrape:
  timeout-margin: 0.5 #Время сбора данных
telemetry:
  addr: ":9182" #Адрес и порт на котором будет слушать экспортер
  path: /metrics #Путь на котором метрики будут доступны
  max-requests: 5 #Ограничение по количеству запросов
```

### Шаг 3: Установка как Windows сервиса

Откройте PowerShell от имени администратора и установите сервис экспортера с помощью `sc.exe`:

```powershell
sc.exe create windows_exp binpath= 'C:\windows_exporter\windows_exporter-"ВАША_ВЕРСИЯ"-amd64.exe --config.file=C:\windows_exporter\config.yaml'
```

**Важно:** Замените `"ВАША_ВЕРСИЯ"` на актуальную версию экспортера (например, `0.24.0`).

**Пример:**

```powershell
sc.exe create windows_exp binpath= 'C:\windows_exporter\windows_exporter-0.24.0-amd64.exe --config.file=C:\windows_exporter\config.yaml'
```

### Шаг 4: Запуск сервиса

```powershell
sc.exe start windows_exp
```

### Шаг 5: Проверка работы

Проверьте, что сервис запущен и метрики доступны:

```powershell
# Проверка статуса сервиса
sc.exe query windows_exp

# Проверка доступности метрик
Invoke-WebRequest -Uri http://localhost:9182/metrics
```

## Сбор дополнительных метрик через файл

Для сбора дополнительных метрик, которые не предоставляются стандартными коллекторами, можно использовать коллектор `textfile`. Он позволяет читать метрики из текстовых файлов в формате Prometheus.

### PowerShell скрипт для сбора метрик Active Directory

Следующий скрипт собирает метрики по группам в AD:
- Количество пользователей в AD
- Количество людей в группах AD

Результат записывается в файл `ad_users.prom`:

```powershell
while( $true ){
	
$ad_users = $(get-aduser -filter *).count
$ad_groups = $(Get-ADGroup -Filter *).Name

Set-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value ""
Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "# HELP windows_ad_users_total Active directory total users.`n"
Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "# TYPE windows_ad_users_total counter`n"
Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "windows_ad_users_total ${ad_users}`n"
Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "# HELP windows_ad_group_users Users in group of Active Directory.`n"
Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "# TYPE windows_ad_group_users counter`n"
foreach ($group in $ad_groups){
	$groupName=$group.Replace(' ','_').Replace('-','_').ToLower()
	$groupMembers=$(get-adgroupmember $group).count
	if($groupMembers -ne $null){
		Add-Content -Path ad_users.prom -Encoding Ascii -NoNewline -Value "windows_ad_group_users{group=""${groupName}""} ${groupMembers}`n"
	}
}
sleep 60
}
```

### Настройка коллектора textfile

Чтобы собирать дополнительные метрики из файлов, при нестандартной установке экспортера, вам потребуется модифицировать строку установки сервиса.

**Значение по умолчанию:** `C:\Program Files\windows_exporter\textfile_inputs`

**Модифицированная команда установки:**

```powershell
sc.exe create windows_exp binpath= 'C:\windows_exporter\windows_exporter-"ВАША_ВЕРСИЯ"-amd64.exe --config.file=C:\windows_exporter\config.yaml --collector.textfile.directory=C:\windows_exporter\files'
```

**Важно:** Создайте директорию `C:\windows_exporter\files` для хранения файлов с метриками.

### Обновленный конфигурационный файл

Добавьте коллектор `textfile` в список включенных коллекторов:

```yaml
---
# Note this is not an exhaustive list of all configuration values
collectors:
  enabled: cpu,cs,logical_disk,net,os,service,system,ad,exchange,dns,textfile #коллекторы которые вам нужны
collector:
  service:
    services-where: Name='windows_exp' #Информация по вашему сервису (тут должно быть указано то как вы его назовете при создании)
log:
  level: warn #Уровень логирования
scrape:
  timeout-margin: 0.5 #Время сбора данных
telemetry:
  addr: ":9182" #Адрес и порт на котором будет слушать экспортер
  path: /metrics #Путь на котором метрики будут доступны
  max-requests: 5 #Ограничение по количеству запросов
```

**Примечание:** Коллектор `textfile` должен быть указан в списке `enabled`.

### Запуск скрипта сбора метрик AD как сервиса

Для автоматического сбора метрик AD можно запустить PowerShell скрипт как отдельный сервис или задачу планировщика.

**Пример создания задачи планировщика:**

```powershell
$action = New-ScheduledTaskAction -Execute 'PowerShell.exe' -Argument '-File "C:\windows_exporter\scripts\ad_metrics.ps1"'
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 1) -RepetitionDuration (New-TimeSpan -Days 365)
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "Windows Exporter AD Metrics" -Action $action -Trigger $trigger -Principal $principal -Description "Collect AD metrics for Prometheus"
```

## Формат файлов метрик для textfile коллектора

Файлы метрик должны иметь расширение `.prom` и соответствовать формату Prometheus:

```prometheus
# HELP metric_name Description of the metric
# TYPE metric_name counter
metric_name{label1="value1",label2="value2"} 123.45

# HELP another_metric Description
# TYPE another_metric gauge
another_metric{label="value"} 67.89
```

## Проверка работы

### Проверка доступности метрик

```powershell
# Проверка основного endpoint
Invoke-WebRequest -Uri http://localhost:9182/metrics

# Проверка конкретной метрики
(Invoke-WebRequest -Uri http://localhost:9182/metrics).Content | Select-String "windows_ad_users_total"
```

### Проверка работы textfile коллектора

```powershell
# Проверка наличия метрик из файлов
(Invoke-WebRequest -Uri http://localhost:9182/metrics).Content | Select-String "windows_ad"
```

## Управление сервисом

### Остановка сервиса

```powershell
sc.exe stop windows_exp
```

### Перезапуск сервиса

```powershell
sc.exe stop windows_exp
sc.exe start windows_exp
```

### Удаление сервиса

```powershell
sc.exe stop windows_exp
sc.exe delete windows_exp
```

## Настройка Prometheus для сбора метрик

Добавьте конфигурацию в `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'windows'
    static_configs:
      - targets: ['windows-server-1:9182', 'windows-server-2:9182']
```

## Troubleshooting

### Сервис не запускается

1. Проверьте логи Windows Event Viewer
2. Убедитесь, что путь к исполняемому файлу правильный
3. Проверьте, что конфигурационный файл существует и имеет правильный формат
4. Убедитесь, что порт 9182 не занят другим процессом

### Метрики не собираются

1. Проверьте, что нужные коллекторы включены в `config.yaml`
2. Убедитесь, что сервис запущен: `sc.exe query windows_exp`
3. Проверьте доступность endpoint: `Invoke-WebRequest -Uri http://localhost:9182/metrics`

### Метрики из textfile не появляются

1. Убедитесь, что файлы имеют расширение `.prom`
2. Проверьте формат метрик в файле
3. Убедитесь, что директория указана правильно в параметрах запуска
4. Проверьте права доступа к файлам (сервис должен иметь доступ на чтение)

### Проблемы с AD метриками

1. Убедитесь, что на сервере установлен модуль Active Directory PowerShell
2. Проверьте, что скрипт имеет права на чтение информации из AD
3. Убедитесь, что скрипт запущен от имени пользователя с необходимыми правами

## Best Practices

1. **Безопасность:**
   - Ограничьте доступ к endpoint метрик через файрвол
   - Используйте аутентификацию в Prometheus при необходимости
   - Ограничьте права сервиса Windows Exporter

2. **Производительность:**
   - Включайте только необходимые коллекторы
   - Настройте таймауты сбора данных
   - Используйте фильтры для service коллектора

3. **Мониторинг:**
   - Настройте алерты на недоступность экспортера
   - Мониторьте использование ресурсов самим экспортером
   - Логируйте ошибки сбора метрик

4. **Обслуживание:**
   - Регулярно обновляйте экспортер до последней версии
   - Проверяйте конфигурацию после обновлений
   - Делайте резервные копии конфигурационных файлов

## Связанные заметки

- [[Proxmox VE Exporter]] - Установка и настройка Proxmox VE Exporter для Prometheus
- [[../../Knowledge Base]] - Общая база знаний

