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
![Alt text](docker up.png?raw=true "Docker up")


Далее надо сгенерировать ssh-ключ:
ssh-keygen

После чего выполнить следующие команды для пользователя admin1, аналогично можно и для developer1:
sudo mkdir -p /home/admin1/.ssh
echo "ssh-rsa key" (тут ключ) | sudo tee /home/admin1/.ssh/authorized_keys
sudo chown -R admin1:admins /home/admin1/.ssh
sudo chmod 700 /home/admin1/.ssh
sudo chmod 600 /home/admin1/.ssh/authorized_keys

Добавим пользователей:
sudo useradd -m -s /bin/bash -g admins -G docker admin1
sudo useradd -m -s /bin/bash -g developers -G docker developer1
