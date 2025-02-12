# Deploying Zabbix with MySQL Using Docker

This procedure outlines the steps to set up Zabbix with MySQL using Docker containers.

## Prerequisites

+ A Linux server (Ubuntu is used in this example).
+ Docker installed and running. If not installed, use:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

+ A dedicated directory for persistent MySQL data.

## Steps:

### Create a Docker Network

Create a custom Docker network (zabbix-net) with a specific subnet and IP range:

```bash
docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 zabbix-net
```

### Deploy MySQL Server

Run a MySQL 8.0 container with necessary environment variables and persistent storage:

```bash
docker run --name mysql-server -t \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix_pwd" \
  -e MYSQL_ROOT_PASSWORD="root_pwd" \
  --network=zabbix-net \
  --restart unless-stopped \
  -v /home/ubuntu/mysql-server-data:/var/lib/mysql \
  -d mysql:8.0-oracle \
  --character-set-server=utf8 \
  --collation-server=utf8_bin \
  --default-authentication-plugin=caching_sha2_password
```

### Deploy Zabbix Java Gateway

Run the Zabbix Java Gateway container:

```bash
docker run --name zabbix-java-gateway -t \
  --network=zabbix-net \
  --restart unless-stopped \
  -d zabbix/zabbix-java-gateway:alpine-6.4-latest
```

### Deploy Zabbix Server

Run the Zabbix server container, linking it to the MySQL database and Java Gateway:

```bash
docker run --name zabbix-server-mysql -t \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix_pwd" \
  -e MYSQL_ROOT_PASSWORD="root_pwd" \
  -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
  --network=zabbix-net \
  -p 10051:10051 \
  --restart unless-stopped \
  -d zabbix/zabbix-server-mysql:alpine-6.4-latest
```

### Deploy Zabbix Web Interface

Run the Zabbix web interface container using Nginx:

```bash
docker run --name zabbix-web-nginx-mysql -t \
  -e ZBX_SERVER_HOST="zabbix-server-mysql" \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="zabbix_pwd" \
  -e MYSQL_ROOT_PASSWORD="root_pwd" \
  --network=zabbix-net \
  -p 8010:8080 \
  --restart unless-stopped \
  -d zabbix/zabbix-web-nginx-mysql:alpine-6.4-latest
```

### Verify Deployment

Check Running Containers:

```bash
docker ps
```

Ensure all containers (mysql-server, zabbix-java-gateway, zabbix-server-mysql, and zabbix-web-nginx-mysql) are running.

Check Zabbix Logs (if needed):

```bash
docker logs -f zabbix-server-mysql
```

### Access Zabbix Web Interface

Open a browser and go to:

```bash
http://<server-ip>:8010
```

Default login credentials:

+ Username: Admin
+ Password: zabbix

