# Ansible playbooks for Ubuntu 18.04 Bionic
Deploy gateway and booker services.


## Requirements
### Linux (Ubuntu 18.04)

##### Requirements for management server
* Ansible 2.6 or above

##### Requirements for service server
* Git
* Docker 19.03.12
* Docker Compose 1.26.2


## Prepare management server

Install Ansible
```bash
apt update
apt -y install python
apt -y install python3
apt -y install python-docker python-psycopg2 python-pip
apt -y install python3-docker python3-psycopg2 python3-pip
apt -y install git
apt -y install ansible
```

Check that all installed:
```bash
# ansible --version
ansible 2.5.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Jul 20 2020, 15:37:01) [GCC 7.5.0]
# python -V
Python 2.7.18
# python3 -V
Python 3.6.9
```

Prepare management ssh keys:
```bash
ssh-keygen -b 4096
```

> Skip this step when you place all services and database on one server.

Copy key for service server (repeat for each service server):
```bash
ssh-copy-id -i ~/.ssh/id_rsa user@host
```

Clone fincubator infrastructure repo:
```bash
cd /opt/
git clone https://github.com/fincubator/infrastructure.git fincubator_infra
cd ./fincubator_infra/ansible/ubuntu/
```

Default ports:
```bash
Redis port: 6379
PostgreSQL port: 5432
Booker port: 8080
Bitshares gateway port: 8889
Bitshares gateway websocket port: 7082
Ethereum gateway port: 8089
```

> When you place all services and database on one server don't user 127.0.0.1 or localhost. 
> Use your server EXTERNAL IP.

Edit defaults:
1. Replace 127.0.0.1 to your database server IP:
   - booker/.env 
   ```bash 
   DB_HOST=127.0.0.1
   ```
   - booker/docker-compose.yml
   ```bash 
   DB_HOST: "127.0.0.1"
   ```   
   - bitshares_gateway/.env
   ```bash
   DATABASE_HOST=127.0.0.1
   ```
   - bitshares_gateway.yml
   ```bash
    DATABASE_HOST: "127.0.0.1"
   ```
   - ethereum_gateway.yml 
   ```bash 
   DB_HOST: "127.0.0.1"
   MEMORY_DB_HOST: "127.0.0.1"
   ```
   - bitshares_gateway/gateway.yml
   ```bash 
   host: 127.0.0.1
   ```   
2. Replace 127.0.0.1 to your bitshares gateway server IP:
   - booker/gateways.yml
   ```bash
       - 127.0.0.1 # host
   ```
3. Replace 127.0.0.1 to your ethereum gateway server IP:
   - bitshares_gateway/gateway.yml
   ```bash 
   host: 127.0.0.1
   ```      
4. Replace 127.0.0.1 to your booker server IP:
   - bitshares_gateway/.env
   ```bash
   BOOKER_HOST=127.0.0.1
   ```
5. Fill invetory file with your server IPs or DNS names.
   - inventory_dev (example)
   ```bash
   [db_hosts]
   128.197.40.38
   ```


(Optional) Change ports default:
1. Change port for Redis in:
   - redis.yml
   ```bash
   redis_db_port: 6379
   ```
   - ethereum_gateway.yml
   ```bash
   MEMORY_DB_PORT: "6379"
   ```
2. Change configs (database server IP, username, password and database name) for PosgreSQL in:
   > *Custom PostgreSQL port not supported*
   - db.yml
   ```bash
   postgres_db_port: 5432
   ```
   - booker/.env 
   ```bash 
   DB_PORT=5432
   DB_USER=booker
   DB_PASSWORD=booker
   DB_DATABASE=booker
   ```
   - bitshares_gateway/.env
   ```bash
   DATABASE_PORT=5432
   DATABASE_USERNAME=bitshares-gateway
   DATABASE_PASSWORD=bitshares-gateway
   DATABASE_NAME=bitshares-gateway
   ```
   - bitshares_gateway.yml
   ```bash
    DATABASE_PORT: 5432
    DATABASE_USERNAME: "bitshares-gateway"
    DATABASE_PASSWORD: "bitshares-gateway"
    DATABASE_NAME: "bitshares-gateway"
   ```
   - ethereum_gateway.yml 
   ```bash 
   DB_USERNAME: "payment-gateway"
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
4. Change configs for bitshares gateway in:
   - bitshares_gateway.yml
   ```bash
    HTTP_HOST: "0.0.0.0"
    HTTP_PORT: 8889
    WS_HOST: "0.0.0.0"
    WS_PORT: 7082
   ```
   - bitshares_gateway/.env
   ```bash
   HTTP_HOST=0.0.0.0
   HTTP_PORT=8889

   WS_HOST=0.0.0.0
   WS_PORT=7082
   
   BOOKER_PORT=8080
   ```
5. Change configs for ethereum gateway in:
   - ethereum_gateway.yml
   ```bash
   PORT: 8089
   ```
   - ethereum_gateway/docker-compose.yaml
   ```bash
    ports:
      - "0.0.0.0:8089:8089"
   ```

---

## Prepare databse server

Edit /etc/default/ufw and add this line at end of file
```bash
IPV6=yes
```

Edit /etc/default/docker and add at end of file
```bash
DOCKER_OPTS="--iptables=false"
```

Enable firewall on server:
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

---

## Prepare service server


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
ufw allow <booker-port>
ufw allow <bitshares-gateway-port>
ufw allow <bitshares-gateway-ws-port>
ufw allow <ethereum-gateway-port>
```

Check firewall status
```bash
# ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
<booker-port>              ALLOW       Anywhere
<bitshares-gateway-ws-port>>     ALLOW       Anywhere
<bitshares-gateway-port>   ALLOW       Anywhere
<ethereum-gateway-port>    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

---

## Deploy

Install common software on all hosts:
```bash
ansible-playbook common.yml --extra-vars "target=all" -i inventory_dev
```

### Deploy databases

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

### Deploy services

#### Install booker on service server
```bash
ansible-playbook booker.yml -i inventory_dev
```

#### [Test enviroment] Install bitshares gateway on service server
```bash
ansible-playbook bitshares_gateway.yml -i inventory_dev
```

#### [Production enviroment] Install bitshares gateway on service server
```bash
ansible-playbook bitshares_gateway_prod.yml -i inventory_dev
```

Next, connect to service server via ssh. 
```bash
# cd /opt/bitshares-gateway
# docker-compose run gateway
ACCOUNT is new account. Let's add and encrypt keys
Please Enter ACCOUNT active private key
```
Enter your active key and press ENTER

```bash
ACCOUNT is new account. Let's add and encrypt keys
Please Enter ACCOUNT memo private key
```
Enter your memokey and press ENTER

```bash
Now enter password to encrypt keys
```
Enter your password twice.

> No limit on characters in the password.
> No way to recover a forgotten password.

If container restarted you need to run it with command:
```bash
# docker run gateway
Account ACCOUNT found. Enter password to decrypt keys
```
Enter your password.


#### Install ethereum gateway on service server
```bash
ansible-playbook ethereum_gateway.yml -i inventory_dev
```



Aftes all deploys on service server you see this:
```bash
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                            NAMES
900f8c900353        ethereum_gateway_app        "docker-entrypoint.s…"   33 minutes ago      Up 33 minutes       0.0.0.0:8089->8089/tcp                           ethereum_gateway_app_1
a54a3159939b        bitshares_gateway_gateway   "bash -c 'pipenv run…"   23 hours ago        Up 2 hours          0.0.0.0:7082->7082/tcp, 0.0.0.0:8889->8889/tcp   gateway
883c5240898b        booker_booker               "bash -c 'pipenv run…"   25 hours ago        Up 2 hours          0.0.0.0:8080->8080/tcp                           booker
````
