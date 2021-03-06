# Magento LTS

Version 2.4

## Requirements

| Service           | Version |
| ----------------- | ------- |
| OS Unix           | *       |
| Docker            | 20      |
| Docker-Compose    | 1.28    |

## Image (rootless)

| Service | Version       |
|---------|---------------|
| OS      | bullseye-slim |
| Nginx   | 1.18          |
| PHP     | 7.4           |

## Install

Open hosts

```
sudo nano /etc/hosts
```

Copy rules

```
127.0.0.1       www.traefik.lan
127.0.0.1       www.phpmyadmin.lan
127.0.0.1       www.rabbitmq.lan
127.0.0.1       www.elasticsearch.lan
127.0.0.1       www.magento.lan
```

Install composer

```shell
# @see https://getcomposer.org/download/
wget -q https://getcomposer.org/download/latest-stable/composer.phar; \
mv composer.phar docker/bin-composer
```

Install mailHog (local)

```shell
# @see https://github.com/mailhog/mhsendmail/releases
wget -q https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64; \
mv mhsendmail_linux_amd64 docker/bin-mhsendmail
```

Update composer auth.json

```shell
cp auth.sample.json src/auth.json # Change the entries into src/auth.json
```

Build image

```shell
docker build . -t aurelienandre/magento-lts:latest \
--build-arg UID=$(id -u) \
--build-arg GID=$(id -g)
```

Start containers

```shell
docker-compose up
```

Initialize magento

```shell
rm -rf src/generated/* \
&& docker-compose exec magento bin/magento app:config:import \
&& docker-compose exec magento bin/magento setup:upgrade \
&& docker-compose exec magento bin/magento setup:di:compile \
&& docker-compose exec magento bin/magento indexer:reindex \
&& docker-compose exec magento bin/magento cache:clean \
&& docker-compose exec magento bin/magento cache:flush
```

Website

Identification : admin @  admin123

| Host                         |
|------------------------------|
| http://www.traefik.lan       |
| http://www.phpmyadmin.lan    |
| http://www.rabbitmq.lan      |
| http://www.elasticsearch.lan |
| http://www.magento.lan       |

## PHP

### Configuration

To configure php, use the environment variable.

Eg. set memory_limit = -1

PHP_MEMORY_LIMIT = -1

Eg. set opcache.enable = 1

PHP_OPCACHE__ENABLE = 1

the "PHP_" prefix is removed and the double underscores are replaced by dots

The entire configuration overlay is in the docker image

```shell
cat /etc/php/7.4/90-php.ini
```

## Supervisor

```shell
supervisorctl help

# default commands (type help <topic>):
# =====================================
# add    exit      open  reload  restart   start   tail   
# avail  fg        pid   remove  shutdown  status  update 
# clear  maintail  quit  reread  signal    stop    version
```

```shell
supervisorctl 

# boot                             RUNNING   pid 92036, uptime 0:00:24
# crontab:crontab-default_00       RUNNING   pid 92039, uptime 0:00:24
# crontab:crontab-index_00         RUNNING   pid 90682, uptime 0:09:59
# server:server-fpm_00             RUNNING   pid 1952, uptime 10:32:06
# server:server-nginx_00           RUNNING   pid 1951, uptime 10:32:06

```

```shell
supervisorctl restart server:*

# server:server-nginx_00: stopped
# server:server-fpm_00: stopped
# server:server-nginx_00: started
# server:server-fpm_00: started
```

```shell
supervisorctl stop crontab:*

# crontab:crontab-index_00: stopped
# crontab:crontab-default_00: stopped
```
