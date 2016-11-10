# Issues


## 'name not resolved'

Clear DNS cache on your Mac:

```
sudo killall -HUP mDNSResponder
```

Or add an alias:

```
alias dnsflush='sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder;echo dns cache flushed'
alias flushdns='dnsflush'
```


## I need my ssh key in my container

Add your ssh key as a volume:

```
app:
    image: php
    volumes:
        - ~/.ssh/id_rsa:/root/.ssh/id_rsa:ro
```
