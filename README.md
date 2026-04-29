## ELK Stack + Docker Logging

Репозиторий содержит полный стек **ELK** (Elasticsearch, Logstash, Kibana) для централизованного сбора и анализа логов Docker-контейнеров.

### Компоненты

| Сервис | Назначение |
|--------|------------|
| **Elasticsearch (hot)** | Горячая нода для активных данных |
| **Elasticsearch (warm)** | Тёплая нода для долгосрочного хранения |
| **Logstash** | Приём, обработка и маршрутизация логов |
| **Kibana** | Визуализация и поиск по логам |
| **Filebeat** | Сбор логов со всех Docker-контейнеров |
| **log-generator** | Тестовое приложение-генератор логов (пример) |

---

### Возможности

- ✅ Автоматический сбор логов **всех** запущенных Docker-контейнеров
- ✅ Разделение на hot/warm ноды Elasticsearch для оптимизации хранения
- ✅ Готовая конфигурация Logstash для приёма логов от Filebeat
- ✅ Визуализация и поиск логов через Kibana
- ✅ Пример генератора логов для тестирования

## Быстрый старт

### Требования

- Docker Engine 20.10+
- Docker Compose 2.0+
- 4 GB свободной RAM

### Установка и запуск

```bash
# Клонировать репозиторий
git clone https://github.com/aleksey-dubrovin/elk-collector.git
cd elk

# Запустить все сервисы
docker-compose up -d

# Проверить работу контейнеров
docker ps
```

### Доступ к сервисам

| Сервис | URL | Логин/пароль |
|--------|-----|--------------|
| **Kibana** | http://localhost:5601 | не требуется |
| **Elasticsearch API** | http://localhost:9200 | не требуется |
| **Logstash (TCP)** | localhost:5044 (Beats) | — |
| **Logstash (JSON)** | localhost:5000 (TCP) | — |

---

## Настройка Kibana

После первого запуска:

1. Открыть http://localhost:5601
2. Перейти в **Stack Management → Index Patterns**
3. Нажать **Create index pattern**
4. Ввести `logstash-*` → Next
5. Выбрать `@timestamp` → Create
6. Перейти в **Discover** для просмотра логов

## Как это работает

1. **Filebeat** читает логи из `/var/lib/docker/containers/*/*.log`
2. Добавляет метаданные Docker (имя контейнера, образ и т.д.)
3. Отправляет логи в **Logstash** на порт `5044`
4. **Logstash** обрабатывает и отправляет в **Elasticsearch**
5. **Elasticsearch** индексирует логи (индекс `logstash-YYYY.MM.dd`)
6. **Kibana** позволяет искать и визуализировать логи

## Тестовый генератор логов

В составе стека запускается контейнер `log-generator`, который каждую секунду пишет случайное сообщение:

- `INFO: Hello there!!`
- `WARNING: Hmmm....something strange`
- `ERROR: OH NO!!!!!!`
- `EXCEPTION: this is exception`

Эти логи автоматически попадают в Elasticsearch и видны в Kibana.

### Остановка генератора

```bash
docker-compose stop log-generator
```

## Полезные команды

### Просмотр логов в реальном времени

```bash
# Логи всех контейнеров
docker-compose logs -f

# Логи только Filebeat
docker-compose logs -f filebeat

# Логи только Logstash
docker-compose logs -f logstash
```

### Проверка индексов в Elasticsearch

```bash
curl http://localhost:9200/_cat/indices?v
```

### Перезапуск стека

```bash
docker-compose restart
```

### Полная остановка и удаление данных

```bash
docker-compose down -v
```

## Устранение неполадок

| Проблема | Решение |
|----------|---------|
| Контейнеры не стартуют | Проверьте свободные порты: `9200`, `5601`, `5044`, `5000` |
| Нет логов в Kibana | Выполните `docker logs filebeat` — возможно, нет прав на `/var/lib/docker/containers` |
| Индекс не создаётся | Проверьте `docker logs logstash` — ошибки подключения к Elasticsearch |
| Permission denied | Убедитесь, что файлы конфигураций имеют права `chmod 644` |

---

## Лицензия

MIT

---