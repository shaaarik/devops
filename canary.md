# Canary-развертывание с балансировкой нагрузки в Nginx и Docker Compose

Для реализации canary-развертывания с постепенным увеличением нагрузки (10%, 30%, 50%, 100%) на одном из двух контейнеров с помощью Nginx и Docker Compose, вы можете использовать следующую конфигурацию:
1. Структура проекта
```
project/
├── docker-compose.yml
├── nginx/
│   └── nginx.conf
├── app-v1/
│   └── (файлы версии 1)
└── app-v2/
    └── (файлы версии 2)
```
1. Docker Compose файл
```
version: '3.8'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app-v1
      - app-v2

  app-v1:
    image: your-app-image:v1
    expose:
      - "8000"

  app-v2:
    image: your-app-image:v2
    expose:
      - "8000"
```
1. Конфигурация Nginx (nginx.conf)
```
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    upstream backend {
        # Основная версия (v1) - получает 90% трафика
        server app-v1:8000 weight=90;
        
        # Canary версия (v2) - получает 10% трафика
        server app-v2:8000 weight=10;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```
4. Постепенное увеличение нагрузки на canary-версию

Для изменения соотношения трафика между версиями вам нужно:
```
    10% на canary (как в примере выше):
    nginx
    Copy

    server app-v1:8000 weight=90;
    server app-v2:8000 weight=10;

    30% на canary:
    nginx
    Copy

    server app-v1:8000 weight=70;
    server app-v2:8000 weight=30;

    50% на canary:
    nginx
    Copy

    server app-v1:8000 weight=50;
    server app-v2:8000 weight=50;

    100% на canary (полный переход):
    nginx
    Copy

    server app-v2:8000;
    # или
    server app-v1:8000 weight=0;
    server app-v2:8000 weight=100;
```
5. Обновление конфигурации

После изменения nginx.conf выполните:
```
docker-compose exec nginx nginx -s reload
```
Это перезагрузит конфигурацию Nginx без простоя.
Альтернативный вариант: с использованием переменных окружения

Если вы хотите динамически управлять распределением трафика, можно использовать переменные окружения:
Измените nginx.conf:
```
upstream backend {
    server app-v1:8000 weight=${WEIGHT_V1};
    server app-v2:8000 weight=${WEIGHT_V2};
}
```
Обновите docker-compose.yml для nginx:
```

environment:
  - WEIGHT_V1=90
  - WEIGHT_V2=10
```
Для изменения соотношения просто обновите переменные окружения и перезапустите nginx.

Это решение позволяет вам плавно переводить трафик с одной версии на другую, минимизируя риски при развертывании новых версий приложения.