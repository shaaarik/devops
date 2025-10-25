Развертывание ELK-стека (Elasticsearch, Logstash, Kibana) с Docker Compose

ELK-стек - это мощное решение для сбора, обработки, хранения и визуализации логов. Вот как его развернуть с помощью Docker Compose.
1. Подготовка окружения
Требования:

    Docker и Docker Compose установлены

    Минимум 4 ГБ оперативной памяти (рекомендуется 8+ ГБ)

    2+ ядра CPU

2. Базовая конфигурация ELK

Создайте структуру проекта:
Copy
```
elk-stack/
├── docker-compose.yml
├── elasticsearch/
│   └── config/
│       └── elasticsearch.yml
├── logstash/
│   ├── config/
│   │   └── logstash.yml
│   └── pipeline/
│       └── logstash.conf
├── kibana/
│   └── config/
│       └── kibana.yml
└── .env
```
Файл .env
```

# Общие настройки
ELK_VERSION=8.12.0
ELASTIC_PASSWORD=YourSecurePassword
KIBANA_PASSWORD=YourSecurePassword
LOGSTASH_INTERNAL_PASSWORD=YourSecurePassword

# Настройки сети
ELASTICSEARCH_HOST=elasticsearch
KIBANA_HOST=kibana
LOGSTASH_HOST=logstash
STACK_NETWORK=elk-network

# Настройки памяти
ES_JAVA_OPTS=-Xms1g -Xmx1g
LS_JAVA_OPTS=-Xms512m -Xmx512m
```
docker-compose.yml
```

version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION}
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=es-docker-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=${ES_JAVA_OPTS}
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - LOGSTASH_INTERNAL_PASSWORD=${LOGSTASH_INTERNAL_PASSWORD}
      - xpack.security.enabled=true
      - xpack.security.authc.api_key.enabled=true
    volumes:
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk
    ulimits:
      memlock:
        soft: -1
        hard: -1

  logstash:
    image: docker.elastic.co/logstash/logstash:${ELK_VERSION}
    container_name: logstash
    environment:
      - LS_JAVA_OPTS=${LS_JAVA_OPTS}
      - ELASTICSEARCH_HOST=${ELASTICSEARCH_HOST}
      - ELASTICSEARCH_PASSWORD=${LOGSTASH_INTERNAL_PASSWORD}
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"   # Beats input
      - "5000:5000/tcp"   # TCP input
      - "5000:5000/udp"   # UDP input
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:${ELK_VERSION}
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://${ELASTICSEARCH_HOST}:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es_data:
    driver: local

networks:
  elk:
    driver: bridge
```
1. Конфигурационные файлы
elasticsearch/config/elasticsearch.yml
```

cluster.name: "es-docker-cluster"
network.host: 0.0.0.0
xpack:
  security:
    authc:
      api_key:
        enabled: true
```
logstash/config/logstash.yml
```

http.host: "0.0.0.0"
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]
xpack.monitoring.elasticsearch.username: "logstash_internal"
xpack.monitoring.elasticsearch.password: "${LOGSTASH_INTERNAL_PASSWORD}"
```
logstash/pipeline/logstash.conf
```

input {
  tcp {
    port => 5000
    codec => json_lines
  }
  udp {
    port => 5000
    codec => json_lines
  }
  beats {
    port => 5044
  }
}

filter {
  # Пример фильтра для парсинга логов Nginx
  if [fileset][module] == "nginx" {
    if [fileset][name] == "access" {
      grok {
        match => { "message" => ["%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{DATA:[nginx][access][url]} HTTP/%{NUMBER:[nginx][access][http_version]}\" %{NUMBER:[nginx][access][response_code]} %{NUMBER:[nginx][access][body_sent][bytes]}( \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\"")?] }
        remove_field => "message"
      }
      mutate {
        add_field => { "read_timestamp" => "%{@timestamp}" }
      }
      date {
        match => [ "[nginx][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
        remove_field => "[nginx][access][time]"
      }
      useragent {
        source => "[nginx][access][agent]"
        target => "[nginx][access][user_agent]"
        remove_field => "[nginx][access][agent]"
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    user => "logstash_internal"
    password => "${LOGSTASH_INTERNAL_PASSWORD}"
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```
kibana/config/kibana.yml
```

server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://elasticsearch:9200"]
monitoring.ui.container.elasticsearch.enabled: true
xpack.security.enabled: true
```
1. Запуск ELK-стека

    Создайте необходимые директории:
```
mkdir -p {elasticsearch,logstash,kibana}/{config,pipeline} && touch {elasticsearch,logstash,kibana}/config/{elasticsearch,logstash,kibana}.yml logstash/pipeline/logstash.conf .env
```
    Заполните файлы конфигурации (как показано выше)

    Запустите стек:
```
docker-compose up -d
```
    Проверьте работу компонентов:

        Elasticsearch: curl -u elastic:YourSecurePassword -k http://localhost:9200

        Kibana: откройте в браузере http://localhost:5601 (логин: elastic, пароль из .env файла)

1. Настройка Filebeat для отправки логов (опционально)

    Создайте конфигурацию Filebeat:
```

filebeat.inputs:
- type: filestream
  enabled: true
  paths:
    - /var/log/*.log

output.logstash:
  hosts: ["localhost:5044"]
```
    Запустите Filebeat в Docker:
```

# Добавьте в docker-compose.yml
filebeat:
  image: docker.elastic.co/beats/filebeat:${ELK_VERSION}
  container_name: filebeat
  volumes:
    - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml
    - /var/log:/var/log:ro
  networks:
    - elk
  depends_on:
    - logstash
```
6. Полезные команды

    Проверить логи Elasticsearch: docker-compose logs -f elasticsearch

    Остановить стек: docker-compose down

    Остановить с удалением данных: docker-compose down -v

    Масштабировать Logstash: docker-compose up -d --scale logstash=3

7. Оптимизация для production

    Кластер Elasticsearch:

        Используйте несколько узлов

        Настройте разделение на master/data узлы

    Мониторинг:

        Включите X-Pack monitoring

        Настройте алерты

    Безопасность:

        Настройте TLS для всех компонентов

        Ограничьте доступ к портам

        Регулярно обновляйте пароли

    Резервное копирование:

        Настройте snapshot repository

        Создайте регулярные бэкапы

Это базовая конфигурация ELK-стека, которую можно расширять в зависимости от ваших потребностей.