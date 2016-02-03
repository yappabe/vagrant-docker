# A lightweight Docker VM

## Getting started

### Requirements

* Brew
* Brew Cask
* Virtualbox
* Vagrant
* Docker
* Docker Compose

### Install Requirements

#### Vagrant and Virtualbox

```
brew cask install vagrant
brew cask install virtualbox
```

#### Docker

```
brew install docker
```


### Add dns resolver domain

```
sudo vim /etc/resolver/docker
```

Add the following line:

```
nameserver 172.17.8.101
```

### Vagrant up

```
git clone https://github.com/yappabe/vagrant-docker.git
cd vagrant-docker
vagrant up
```

An issue with port-forwarding, reload to fix this.

```
vagrant reload
```

### Add route

*Do this after each reboot! [Or automate it!](https://www.jverdeyen.be/mac/persistent-static-routes-mac-os-x/)*

```
sudo route -n add -net 172.18.0.0  172.17.8.101
```

### Flush DNS cache

```
sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder
```

### Reboot

You may need to reboot your device to ensure `/etc/resolver/docker` is being used.

### Add the `DOCKER_HOST` env var

```
export DOCKER_HOST=tcp://localhost:2375
```

You can add this in `.bashrc` or any file that runs every interactive shell launch.


## Test Docker

```
docker ps
```

When the following line appears:

```
d370b4d9e575        tonistiigi/dnsdock    "/go/bin/dnsdock"        X hours ago        Up X hours          0.0.0.0:53->53/udp   dnsdock
```

You can now visit `http://dnsdock.docker/services`.

## Install Docker Compose

```
brew install docker-compose
```


## Example docker-compose.yml for Symfony2 development in PHP 7.0.

```yml
app:
    image: busybox
    volumes:
        - .:/var/www/app
        - /vendor
        - /tmpfs
    tty: true
    mem_limit: 1000000000

nginx:
    image: yappabe/nginx
    links:
        - php
        - mailcatcher
    volumes_from:
        - app
    environment:
        DOCUMENT_ROOT: /var/www/app/web
        INDEX_FILE: app_dev.php
        PHP_FPM_SOCKET: php:9000
        DNSDOCK_ALIAS: project.docker

mysql:
    image: yappabe/mysql
    environment:
        MYSQL_PASS: dev
        MYSQL_USER: dev
        ON_CREATE_DB: project
        DNSDOCK_ALIAS: mysql.project.docker

php:
    image: yappabe/php:7.0
    working_dir: /var/www/app
    volumes_from:
        - app
    links:
        - mysql
        - mailcatcher

mailcatcher:
    image: yappabe/mailcatcher
    environment:
        DNSDOCK_ALIAS: mailcatcher.project.docker
```

```
docker-compose up -d
```
