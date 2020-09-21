# Ansible playbooks for Ubuntu 18.04 Bionic
Deploy gateway and booker services.


## Install
### Linux (Ubuntu 18.04)

##### Requirements on management server
* Ansible

##### Requirements on service server
* Git
* Docker
* Docker Compose

Install Ansible
```bash
apt update
apt -y install python
apt -y install ansible
apt -y install python-docker python-psycopg2 python-pip python3-docker python3-psycopg2 python3-pip
```

Check that python3 installed:
```bash
python -V
Python 3.6.9
python3 -V
Python 3.6.9
```

ssh-keygen -b 4096
cat ~/.ssh/id_rsa.pub
ssh-copy-id -i ~/.ssh/id_rsa user@host

### Pre-deploy phase

Defaults:
```bash
Redis port: 6379
PostgreSQL port: 5432
Booker port: 8080
Ethereum Gateway port: 8089
```

1. Change port for Redis in (if needed):
   - redis.yml
   ```bash
   redis_db_port: 6379
   ```
   - ethereum_gateway.yml
   ```bash
   MEMORY_DB_PORT: "6379"
   ```
2. Change configs for PosgreSQL in (if needed):
   > *Custom PostgreSQL port not supported*
   - db.yml ```bash postgres_db_port: 5432```
   - booker/.env 
   ```bash 
   DB_PORT=5432
   DB_HOST=127.0.0.1
   DB_USER=booker
   DB_PASSWORD=booker
   DB_DATABASE=booker
   ```
   - ethereum_gateway.yml 
   ```bash 
   DB_HOST: "127.0.0.1"
   DB_USER: "payment-gateway"
   DB_PASSWORD: payment-gateway"
   DB_DATABASE: "payment-gateway"
   ```
3. Change configs for booker in:
   - booker.yml
   ```bash
   HTTP_PORT: "8080"
   HTTP_PORT_HOST: "8080"
   ```
   - booker/.env
   ```bash
   HTTP_PORT=8080
   HTTP_PORT_HOST=8080
   ```
4. Fill inventory_dev with your server IPs or DNS names.






Edit /etc/default/ufw and add this line at end of file
```bash
IPV6=yes
```

Edit /etc/default/docker and add at end of file
```bash
DOCKER_OPTS="--iptables=false"
```

Enable firewall on database server:
```bash
ufw status
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
ufw allow from <app-server-ip> to any port <postgresql-port>
ufw allow from <app-server-ip> to any port <redis-port>
```

Check firewall status
```bash
# ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
<postgresql-port>          ALLOW       <app-server-ip>
<redis-port>               ALLOW       <app-server-ip>
22/tcp (v6)                ALLOW       Anywhere (v6)
```


Install common software on all hosts:
```bash
ansible-playbook common.yml --extra-vars "target=all" -i inventory_dev
```

Install postgresql on database server:
```bash
ansible-playbook db.yml -i inventory_dev
```

Install redis on database server:
```bash
ansible-playbook redis.yml -i inventory_dev
```

Aftes deploy on database server you see this:
```bash
# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
60f3ba4ff59d        redis:5.0.7-buster     "docker-entrypoint.s…"   11 seconds ago      Up 9 seconds        0.0.0.0:6379->6379/tcp   app_memory_db01
35706b4fc30f        postgres:12.3-alpine   "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes        0.0.0.0:5432->5432/tcp   app_db01
````





