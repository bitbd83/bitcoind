
# docker-bitcoind

[![Docker Stars](https://img.shields.io/docker/stars/nexbitio/bitcoind.svg)](https://hub.docker.com/r/nexbitio/bitcoind/)
[![Docker Pulls](https://img.shields.io/docker/pulls/nexbitio/bitcoind.svg)](https://hub.docker.com/r/nexbitio/bitcoind/)

A Docker configuration with sane defaults for running a fully-validating
Bitcoin node, based on [Alpine Linux](https://alpinelinux.org/).

**Warning:** older versions of this Dockerfile were based on Ubuntu and
included all build dependencies, but due to the ridiculous size of the
resulting image I've moved to binaries downloaded to Alpine Linux. If I've
screwed up your development workflow somehow, file an issue and we'll get it
fixed.

## Quick start

Requires that [Docker should be installed](https://docs.docker.com/install/) on the host machine.

```bash
# Create some directory where your bitcoin data will be stored.
$ mkdir /home/youruser/bitcoin_data

$ docker run --name bitcoind -d \
   --env 'BTC_RPCUSER=foo' \
   --env 'BTC_RPCPASSWORD=password' \
   --env 'BTC_TXINDEX=1' \
   --volume /home/youruser/bitcoin_data:/root/.bitcoin \
   -p 127.0.0.1:8332:8332 \
   --publish 8333:8333 \
   nexbitio/bitcoind

$ docker logs -f bitcoind
[ ... ]
```


## Configuration

A custom `bitcoin.conf` file can be placed in the mounted data directory.
Otherwise, a default `bitcoin.conf` file will be automatically generated based
on environment variables passed to the container:

| name | default |
| ---- | ------- |
| BTC_RPCUSER | btc |
| BTC_RPCPASSWORD | changeme |
| BTC_RPCPORT | 8332 |
| BTC_RPCALLOWIP | ::/0 |
| BTC_RPCCLIENTTIMEOUT | 30 |
| BTC_DISABLEWALLET | 1 |
| BTC_TXINDEX | 0 |
| BTC_TESTNET | 0 |
| BTC_DBCACHE | 0 |
| BTC_ZMQPUBHASHTX | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBHASHBLOCK | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBRAWTX | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBRAWBLOCK | tcp://0.0.0.0:28333 |


## Daemonizing

The smart thing to do if you're daemonizing is to use Docker's [builtin
restart
policies](https://docs.docker.com/config/containers/start-containers-automatically/#use-a-restart-policy),
but if you're insistent on using systemd, you could do something like

Create a "bitcoind.service"
```bash
$ nano /etc/systemd/system/bitcoind.service
$ systemctl start bitcoind
$ systemctl enable bitcoind
```

# bitcoind.service 
```bash
[Unit]
Description=Bitcoind
After=docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker kill bitcoind
ExecStartPre=-/usr/bin/docker rm bitcoind
ExecStartPre=/usr/bin/docker pull nexbitio/bitcoind
ExecStart=/usr/bin/docker run \
    --name bitcoind \
    -p 8333:8333 \
    -p 127.0.0.1:8332:8332 \
    -v /data/bitcoind:/root/.bitcoin \
    nexbitio/bitcoind
ExecStop=/usr/bin/docker stop bitcoind
```

# Service file ends
# Run : ["docker logs -f bitcoind" to ensure that bitcoind continues to run.]
# Implement NGINX
```bash
~#apt-get install nginx
~#nano /etc/nginx/nginx.conf
```
# Basic NGINX conf below (can use ssl also)
```bash
worker_processes  auto;
daemon off;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  10024;
}
http {
    sendfile      off;
    access_log    off;
    server_tokens off;
    keepalive_timeout  65;
    server {
        charset utf-8;
        client_max_body_size 128M;
        listen 80;
	location / {
		proxy_pass http://127.0.0.1:8332;
		proxy_redirect off;
	}
    }
}
```




Thank you. For more support and products please visit: https://nexbit.io
Mail me : developer@nexbit.io
