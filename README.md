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

Composer

```shell
# Download last version of composer
wget -q https://getcomposer.org/download/latest-stable/composer.phar; \
mv composer.phar docker/bin-composer
```

Phing

```shell
# Download last version of phing
wget -q https://www.phing.info/get/phing-latest.phar; \
mv phing-latest.phar docker/bin-phing
```

MailHog

```shell
# Download last version of mhsendmail for mailhog
wget -q https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64; \
mv mhsendmail_linux_amd64 docker/bin-mhsendmail
```

Start all

```shell
# Build image
docker build . -t aurelienandre/magento-lts:latest
```

```shell
# Start containers
docker-compose up
```

```shell
# Run deploy local
docker-compose exec -u rootless magento bin-phing -f /deploy/build-local.xml
```

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

# crontab:crontab-default_00       RUNNING   pid 57, uptime 0:00:13
# crontab:crontab-index_00       RUNNING   pid 8, uptime 0:03:25
# server:server-fpm_00            RUNNING   pid 11, uptime 0:03:25
# server:server-nginx_00          RUNNING   pid 9, uptime 0:03:25

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
