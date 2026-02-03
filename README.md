# strigalev.n-itmo-megaschool-devops-2026

Добавим пользователей:
    sudo groupadd developers
    sudo groupmod developers -g 1001
    sudo groupadd admins -g 1002
Добавим им автоматическое выполнение:

sudo visudo

И там в группе admins добавим строчку:
%admins ALL=(ALL) NOPASSWD: ALL

Установим докер (последнюю версию):

24.10 — «Oracul»;
24.04 — «Noble»;
22.04 — «Jammy».

sudo apt update
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble (тут версия убунту) stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce -y
sudo systemctl status docker
![Alt text](/docker_up.png?raw=true "Optional Title")


Далее надо сгенерировать ssh-ключ:
ssh-keygen

После чего выполнить следующие команды для пользователя admin1, аналогично можно и для developer1:
sudo mkdir -p /home/admin1/.ssh
echo "ssh-rsa key" (тут ключ) | sudo tee /home/admin1/.ssh/authorized_keys
sudo chown -R admin1:admins /home/admin1/.ssh
sudo chmod 700 /home/admin1/.ssh
sudo chmod 600 /home/admin1/.ssh/authorized_keys

Добавим пользователей в группу docker:
sudo usermod -aG docker admins
sudo usermod -aG docker devlopers
sudo useradd -m -s /bin/bash -g admins -G docker admin1
sudo useradd -m -s /bin/bash -g developers -G docker developer1

И поставим автостарт докера:

sudo systemctl enable docker

Далее поставим nexus:
docker-compose-nexus.yml
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: unless-stopped
    ports:
      - "8081:8081"
      - "8082:8082"
    volumes:
      - nexus-data:/nexus-data

volumes:
  nexus-data:
  
И сам мониторинг:

cat docker-compose-monitoring.yml
services:
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123

  node-exporter:
    image: prom/node-exporter:latest
    ports: ["9100:9100"]
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports: ["8080:8080"]
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
    privileged: true

cat prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node_exporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

   Добавляем датасорсы:

   Configuration - Data Sources - Add data source

Choose Prometheus
URL: http://prometheus:9090
Save & targetsImport dsashboard
DashboardS - Import
ID: 1860 (Node Exporter Full) - Load
Prometheus - targetsImport
Repeat for ID: 893 (Docker Monitoring)

Запускаем nexus:
docker compose -f docker-compose-nexus.yml up -d

Находим пароль для nexus в файле контейнера:
docker exec nexus
cat /nexus-data/admin.password

Логинимся, после чего создаем репозиторий с такими настройками:
HTTP: Port: 8082
Remote storage: https://registry-1.docker.io
Allow anonymous docker pull
Press Create repository

Перед этим говорим, чтобы докер доверял локальному нексусу:
sudo nano /etc/docker/daemon.json
{
"insecure-registries": ["localhost:8082"],
"registry-mirrors": ["http://localhost:8082"]
}

Запускаем мониторинг:
docker compose -f docker-compose-monitoring.yml down
docker compose -f docker-compose-monitoring.yml up -d

Заходим в grafana, выбираем data source:




