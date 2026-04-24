---
created: 2026-04-24
tags: [java, jfr, heap-dump, troubleshooting, performance]
category: infrastructure
---

# Снятие JFR и дампа памяти у Java сервисов

## Обзор

Для диагностики проблем Java-сервисов обычно снимают:
- `JFR` (Java Flight Recorder) - профиль CPU, lock, allocation, GC и событий JVM во времени;
- `Heap dump` (`.hprof`) - срез памяти JVM для анализа утечек и больших объектов.

## Предварительные требования

1. Доступ к хосту с Java процессом.
2. Достаточно свободного места на диске (heap dump может быть размером с heap).
3. Установленные инструменты JDK (`jcmd`, `jmap`, `jps`).

Проверка:

```bash
java -version
jcmd -h
```

## Поиск PID Java процесса

```bash
# Рекомендуемый способ
jcmd -l

# Альтернатива
jps -lv
```

## Снятие JFR

### Быстрый профиль (5 минут)

```bash
PID=12345
OUT_DIR=/tmp/java-dumps
mkdir -p "$OUT_DIR"

jcmd "$PID" JFR.start \
  name=on_demand \
  settings=profile \
  duration=5m \
  filename="$OUT_DIR/service_$(date +%F_%H-%M-%S).jfr"
```

Проверить активные записи:

```bash
jcmd "$PID" JFR.check
```

### Долгая запись с ручной остановкой

```bash
jcmd "$PID" JFR.start name=incident settings=profile filename=/tmp/java-dumps/incident.jfr
# ... подождать воспроизведения проблемы ...
jcmd "$PID" JFR.stop name=incident
```

### Снять snapshot без остановки

```bash
jcmd "$PID" JFR.dump name=incident filename=/tmp/java-dumps/incident_snapshot.jfr
```

## Снятие Heap Dump

### Рекомендуемый способ (`jcmd`)

```bash
PID=12345
OUT_DIR=/tmp/java-dumps
mkdir -p "$OUT_DIR"

jcmd "$PID" GC.heap_dump "$OUT_DIR/heap_$(date +%F_%H-%M-%S).hprof"
```

### Альтернатива (`jmap`)

```bash
jmap -dump:live,format=b,file=/tmp/java-dumps/heap_live.hprof 12345
```

> `live` включает только живые объекты и может уменьшить размер дампа.

## Минимальный набор артефактов для инцидента

```bash
PID=12345
OUT_DIR=/tmp/java-dumps/incident_$(date +%F_%H-%M-%S)
mkdir -p "$OUT_DIR"

jcmd "$PID" VM.version > "$OUT_DIR/vm_version.txt"
jcmd "$PID" VM.flags > "$OUT_DIR/vm_flags.txt"
jcmd "$PID" Thread.print > "$OUT_DIR/thread_dump.txt"
jcmd "$PID" GC.heap_info > "$OUT_DIR/heap_info.txt"
jcmd "$PID" JFR.start name=incident settings=profile duration=3m filename="$OUT_DIR/incident.jfr"
jcmd "$PID" GC.heap_dump "$OUT_DIR/heap.hprof"
```

## Анализ

- `JFR`: JDK Mission Control (JMC), IntelliJ Profiler, async-profiler converters.
- `Heap dump`: Eclipse MAT, VisualVM, IntelliJ Profiler.

## Рекомендации для production

1. Делайте дампы в каталог с контролем прав доступа (`chmod 700`).
2. Не держите дампы на сервере дольше необходимого (могут содержать секреты в памяти).
3. Сжимайте перед передачей:
   ```bash
   gzip -9 /tmp/java-dumps/*.jfr /tmp/java-dumps/*.hprof
   ```
4. На постоянной основе включите дамп при OOM:
   ```bash
   -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/java-dumps
   ```

## Частые ошибки

1. `AttachNotSupportedException` - процесс запущен под другим пользователем или запрещен attach.
2. `No space left on device` - недостаточно места под `.hprof`.
3. `Operation not permitted` - не хватает прав, используйте того же пользователя, что и JVM.

## Связанные заметки

- [[../Linux/System Administration]] - Базовые операции Linux
- [[../Monitoring/Prometheus/Windows Exporter]] - Мониторинг хостов (смежная тема observability)
