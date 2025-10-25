создаём директорию, в которой будем хранить данные
```
$ mkdir -p ~/minio/data 
```
minio-compose.yml
```
version: '3.7'

services:
  minio:
    image:  quay.io/minio/minio:RELEASE.2023-07-11T21-29-34Z
    container_name: minio
    command: server /data --console-address ":9001"
    volumes:
      - /home/student/minio/data:/data
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

nginx.conf
```
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log warn;
pid       /var/run/nginx.pid;

events {
    worker_connections 4096;
}

http {
    include       /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_refer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log main;
    sendfile    on;
    keepalive_timeout  65;

    # include /etc/nginx/conf.d/*.conf

    upstream minio {
        server minio:9000;
    }

    upstream console {
        server minio:9001;
    }

    server {
        listen       9000;
        server_name  localhost;

        # To allow special chatacters in headers
        ignore_invalid_headers off;
        # Allow any size file to be upload.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;
        proxy_request_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 300;
            # Default is HTTP/1, keepalive is only enabled on HTTP/1.1
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            chunked_transfer_encoding off;
            proxy_pass http://minio;
        }
    }

    server {
        listen       9001;
        server_name  localhost;

        # To allow special chatacters in headers
        ignore_invalid_headers off;
        # Allow any size file to be upload.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;
        proxy_request_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-NginX-Proxy true;

            # This is necessary to pass the correct IP to be hached
            real_ip_header X-Real-IP;

            proxy_connect_timeout 300;

            # To support websocker
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            chunked_transfer_encoding off;
            proxy_pass http://console;
        }
    }
}
```
change minio-compose.yml
```
version: '3.8'

services:
  minio:
    image:  quay.io/minio/minio:RELEASE.2023-07-11T21-29-34Z
    container_name: minio
    command: server /data --console-address ":9001"
    volumes:
      - /home/student/minio/data:/data
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
  nginx:
    image: nginx:1.19.2-alpine
    container_name: nginx
    hostname: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "9000:9000"
      - "9001:9001"
    depends_on:
      - minio    
```

```
docker compose -f minio-compose.yml up -d 

curl https://rclone.org/install.sh | sudo bash

$ vi ~/.config/rclone/rclone.conf
[yandex-s3]
type = s3
provider = AWS
access_key_id = <Access Key Яндекс облака>
secret_access_key = <Secret Key Яндекс облака>
region = ru-central1
endpoint = storage.yandexcloud.net

[minio]
type = s3
provider = Minio
env_auth = false
access_key_id = <Access Key MinIO>
secret_access_key = <Secret Key MinIO>
region = ru-central1
endpoint = http://127.0.0.1:9000


$ rclone sync yandex-s3:<имя бакета в Яндекс Облаке> minio:<имя бакета в MinIO>
$ rclone ls <minio или yandex-s3>:<имя бакета>
$ crontab -e
0 * * * * rclone sync yandex-s3:<имя бакета в Яндекс Облаке> minio:<имя бакета в MinIO>
```