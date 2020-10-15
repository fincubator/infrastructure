# Ansible playbooks for Ubuntu 18.04 Bionic
Deploy gateway and booker services.


## Requirements
### Linux (Ubuntu 18.04)

##### Requirements for management server
* Ansible 2.5 or above

##### Requirements for service server
* Git
* Docker 19.03.12
* Docker Compose 1.26.2


## Prepare management server

Install necessary components
```bash
apt update
apt -y install python
apt -y install python3
apt -y install python-docker python-psycopg2 python-pip
apt -y install python3-docker python3-psycopg2 python3-pip
apt -y install git
apt -y install ansible
```
> Note: docker and docker-compose will be installed later via ansible playbook


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
Python 2.7.17
# python3 -V
Python 3.6.9
```

Prepare management ssh keys:
```bash
ssh-keygen -b 4096
```

Copy key for service server (repeat for each service server):
> Skip this step when you place all services and database on one server.
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

> When you place all services and database on one server don't use 127.0.0.1 or localhost. 
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
   - ethereum_gateway/.env
   ```bash 
   DB_HOST=127.0.0.1
   MEMORY_DB_HOST=127.0.0.1
   ```
2. Replace 127.0.0.1 to your bitshares gateway server IP:
   - booker/gateways.yml
   ```bash
	  target:
		- 127.0.0.1 # host
   ```
3. Replace 127.0.0.1 to your ethereum gateway server IP:
   - booker/gateways.yml
   ```bash
	  native:
		- 127.0.0.1 # host
   ```   
4. Replace <your-prefix> with your exchange prefix:
   > EXCHANGE_PREFIX and gateway_prefix must match in booker and all gateways
   - booker/.env
   ```bash
   EXCHANGE_PREFIX=<your-prefix>
   ``` 
5. Replace 127.0.0.1 to your booker server IP:
   - bitshares_gateway/.env
   ```bash
   BOOKER_HOST=127.0.0.1
   ```
6. Replace <your-prefix> with your exchange prefix:
   > EXCHANGE_PREFIX and gateway_prefix must match in booker and all gateways
   - bitshares_gateway/gateway.yml
   ```bash
   gateway_prefix: <your-prefix>
   ```
7. Replace all variables with your bitshares data:
   - bitshares_gateway/gateway.yml
   ```bash
	account: <your-account>

	keys:
	  active: <your-active-key>
	  memo: <your-memo-key>
   ```
8. Replace <your-prefix> with your exchange prefix:
   > EXCHANGE_PREFIX and gateway_prefix must match in booker and all gateways
   - ethereum_gateway.yml
   ```bash
	EXCHANGE_PREFIX: "<your-prefix>"
   ```
   - ethereum_gateway/.env
   ```bash
	EXCHANGE_PREFIX=<your-prefix>
   ```   
9. Replace all variables with your ethereum data:
   - ethereum_gateway/.env
   ```bash
    ETHEREUM_USDT_ADDRESS=<your-asset-address>
    ETHEREUM_COLD_KEY=<your-cold-key>
    ETHEREUM_SIGN_KEY=<your-sign-key>
   ```  
10. Replace 127.0.0.1 to your booker server IP:
    - ethereum_gateway/.env
    ```bash
     BOOKER_PROVIDER=ws://127.0.0.1:8080/ws-rpc
    ```  
11. Fill inventory file with your server IPs or DNS names.
    - inventory_dev (example)
    ```bash
    [db_hosts]
    128.197.40.38
    ```
12. (Optional) Change port for Redis in:
    - redis.yml
    ```bash
    redis_db_port: 6379
    ```
    - ethereum_gateway.yml
    ```bash
    MEMORY_DB_PORT: "6379"
    ```
13. (Optional) Change configs (database server IP, username, password and database name) for PosgreSQL in:
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
    DB_PASSWORD: "payment-gateway"
    DB_DATABASE: "payment-gateway"
    ```
14. (Optional) Change configs for booker in:
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
15. (Optional) Change configs for bitshares gateway in:
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
16. (Optional) Change configs for ethereum gateway in:
    - ethereum_gateway.yml
    ```bash
    HOST_PORT: 8089
    PORT: 8089
    ```
    - ethereum_gateway/.env
    ```bash
    HOST_PORT=8089
    PORT=8089
     
    MEMORY_DB_PORT=6379
    ``` 

---

## Prepare database server

Edit `/etc/default/ufw` and add this line at end of file
```bash
IPV6=yes
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


Edit `/etc/default/ufw` and add this line at end of file
```bash
IPV6=yes
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
# ansible-playbook common.yml --extra-vars "target=all" -i inventory_dev

PLAY RECAP **********************************************************************************
<server-ip>              : ok=9    changed=0    unreachable=0    failed=0
```

On database server and service server complete this steps:
1. Еdit `/etc/default/docker` and add at end of file
   ```bash
   DOCKER_OPTS="--iptables=false"
   ```
   
2. Edit `/etc/docker/daemon.json` and add at end of file
   > This file may be missing. In this case, you need to create it.
   ```bash
   { "iptables" : false }
   ```

3. Restart docker service
   ```bash
   systemctl restart docker
   ```

4. Enable private subnets masquerade
   ```bash
   iptables -t nat -A POSTROUTING ! -o docker0 -s 172.16.0.0/12 -j MASQUERADE
   ```

4. Edit `/etc/default/ufw`. Replace line
   ```bash
   DEFAULT_FORWARD_POLICY="DROP"
   ```
   to
   ```bash
   DEFAULT_FORWARD_POLICY="ACCEPT"
   ```

5. Edit `/etc/ufw/before.rules` and add this lines before `*filter` line
   ```bash
   # NAT table rules
   *nat
   :POSTROUTING ACCEPT [0:0]
   -A POSTROUTING -s 172.16.0.0/12 ! -o docker0 -j MASQUERADE
   COMMIT
   ```
6. Restart UFW
   ```bash
   ufw reload
   ```

### Deploy databases

Install postgresql on database server:
```bash
# ansible-playbook db.yml -i inventory_dev

PLAY RECAP **********************************************************************************
<server-ip>              : ok=13   changed=1    unreachable=0    failed=0

```

Install redis on database server:
```bash
# ansible-playbook redis.yml -i inventory_dev

PLAY RECAP **********************************************************************************
<server-ip>              : ok=3    changed=1    unreachable=0    failed=0
```

Aftes deploy on database server you see this:
```bash
# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
60f3ba4ff59d        redis:5.0.7-buster     "docker-entrypoint.s…"   11 seconds ago      Up 9 seconds        0.0.0.0:6379->6379/tcp   app_memory_db01
35706b4fc30f        postgres:12.3-alpine   "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes        0.0.0.0:5432->5432/tcp   app_db01
````

### Deploy services

#### Install bitshares gateway on service server
```bash
# ansible-playbook bitshares_gateway.yml -i inventory_dev

PLAY RECAP ********************************************************************************************************************************************************************************************************
<server-ip>              : ok=5    changed=5    unreachable=0    failed=0
```

#### Install ethereum gateway on service server
```bash
# ansible-playbook ethereum_gateway.yml -i inventory_dev

PLAY RECAP ********************************************************************************************************************************************************************************************************
<server-ip>              : ok=4    changed=4    unreachable=0    failed=0
```

#### Install booker on service server
```bash
# ansible-playbook booker.yml -i inventory_dev

PLAY RECAP ********************************************************************************************************************************************************************************************************
<server-ip>              : ok=7    changed=3    unreachable=0    failed=0
```


Aftes all deploys on service server you see this:
```bash
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                            NAMES
900f8c900353        ethereum_gateway_app        "docker-entrypoint.s…"   23 minutes ago      Up 33 minutes       0.0.0.0:8089->8089/tcp                           ethereum_gateway_app_1
a54a3159939b        bitshares_gateway_gateway   "bash -c 'pipenv run…"   23 minutes ago      Up 2 hours          0.0.0.0:8889->8889/tcp                           gateway
883c5240898b        booker_booker               "bash -c 'pipenv run…"   20 minutes ago      Up 2 hours          0.0.0.0:8080->8080/tcp                           booker
````

## Check logs

Check booker logs
```bash
# docker logs booker
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
2020-10-01 13:14:44,207|BookerApp|INFO|Database connected to <database-server-ip>:5432
2020-10-01 13:14:44,211|BookerApp|INFO|Booker server started on 0.0.0.0:8080
2020-10-01 13:14:44,211|BookerApp|INFO|Setup USDT gateways clients connections...
2020-10-01 13:14:44,211|BookerApp|INFO|nativeUSDT client created and ready to connect ws://<ethereum-gateway-server-ip>:8089/ws-rpc
2020-10-01 13:16:54,780|BookerApp|INFO|targetUSDT client created and ready to connect ws://<bitshares-gateway-server-ip>:8889/ws-rpc
2020-10-01 13:16:54,780|OrderProcessing|INFO|Start to process orders...
```

Check bitshares gateway logs
```bash
# docker logs gateway
Loading .env environment variables…
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
Loading .env environment variables…
2020-10-01 15:16:35,905|Config build|INFO|Successfully loaded user's .env configuration
2020-10-01 15:16:35,912|Config build|INFO|Using unencrypted keys from gateway.yml file
2020-10-01 15:16:35,912|Config build|INFO|Successfully loaded user's gateway.yml configuration
/root/.local/share/virtualenvs/app-4PlAip0Q/lib/python3.8/site-packages/graphenecommon/chain.py:115: RuntimeWarning: coroutine 'dict.__new__' was never awaited
  self.account_class(account)
RuntimeWarning: Enable tracemalloc to get the object allocation traceback
2020-10-01 15:16:36,774|Gateway|INFO|Account localtest-finteh-gateway-usdt-wallet is new. Let's retrieve data from blockchain and record it in database!
2020-10-01 15:16:36,842|Gateway|INFO|Retrieve account localtest-finteh-gateway-usdt-wallet data from database
2020-10-01 15:16:36,847|Gateway|INFO|Start from operation 43602191, block number 40385877
2020-10-01 15:16:36,858|Gateway|INFO|BookerClient ready to connect  ws://<your-booker-server-ip>:8080/
2020-10-01 15:16:36,861|Gateway|INFO|Started websockets rpc server on ws://0.0.0.0:8889/ws-rpc
2020-10-01 15:16:36,863|Gateway|INFO|
     Run LOCALFINTEH USDT BitShares gateway
     Distribution account: localtest-finteh-gateway-usdt-wallet
     Connected to BitShares API node: wss://testnet.dex.trading/
     Connected to database: True

2020-10-01 15:16:36,864|Gateway|INFO|Watching localtest-finteh-gateway-usdt-wallet for new operations started
2020-10-01 15:16:36,867|Gateway|INFO|Watching unconfirmed operations
2020-10-01 15:16:36,868|Gateway|INFO|Parsing blocks...
```

Check ethereum gateway logs
```bash
# docker logs ethereum_gateway_app_1

> @ migrate /app
> sequelize db:migrate


Sequelize CLI [Node: 14.11.0, CLI: 6.2.0, ORM: 6.3.5]

Loaded configuration file "dist/config/config.db.js".
Using environment "development".
== 20191130072647-create-wallets-derived-wallets-and-transactions: migrating =======
(sequelize) Warning: PostgresSQL does not support 'INTEGER' with LENGTH, UNSIGNED or ZEROFILL. Plain 'INTEGER' will be used instead.
>> Check: http://www.postgresql.org/docs/9.4/static/datatype.html
== 20191130072647-create-wallets-derived-wallets-and-transactions: migrated (3.118s)


> @ serve /app
> node dist/index.js

(sequelize) Warning: PostgresSQL does not support 'INTEGER' with LENGTH, UNSIGNED or ZEROFILL. Plain 'INTEGER' will be used instead.
>> Check: http://www.postgresql.org/docs/9.4/static/datatype.html
Connection to Redis has been established successfully.
Connection to Redis has been established successfully.
[WARN] Redis server does not require a password, but a password was supplied.
[WARN] Redis server does not require a password, but a password was supplied.
[WARN] Redis server does not require a password, but a password was supplied.
Executing (default): SELECT 1+1 AS result
Connection to database has been established successfully.
```


---

## Experemential features

#### More securely installation of bitshares gateway 
```bash
# ansible-playbook bitshares_gateway_prod.yml -i inventory_dev

PLAY RECAP ********************************************************************************************************************************************************************************************************
<server-ip>              : ok=5    changed=5    unreachable=0    failed=0
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
Type your password and press Enter.

