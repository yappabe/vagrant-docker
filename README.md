# A lightweight Docker VM

## Getting started

### Requirements

* Brew
* Brew Cask
* Virtualbox
* Vagrant
* Docker

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

#### Dnsmasq

```
brew install dnsmasq
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

### Add dns resolver domain

```
sudo vim /etc/resolver/docker
```

Add the following line:

```
nameserver 172.17.8.101
```

### Flush DNS cache

```
sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder
```

## Test Docker

```
docker ps
```

When the following line appears:

```
d370b4d9e575        tonistiigi/dnsdock    "/go/bin/dnsdock"        X hours ago        Up X hours          0.0.0.0:53->53/udp   dnsdock
```

You can now visit `http://dnsdock.docker/services`.
