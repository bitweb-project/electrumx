OS Ubuntu 24.04


Prepare::

  sudo -s

  apt-get update && apt-get upgrade

  apt install -y python3.12 python3.12-venv python3.12-dev python3-pip python3-setuptools python3-multidict libleveldb-dev gcc g++ libsnappy-dev zlib1g-dev libbz2-dev libgflags-dev build-essential git librocksdb-dev build-essential pkg-config

  git clone https://github.com/bitweb-project/electrumx /opt/electrumx
  
  cd /opt/electrumx

  mkdir -p db

  groupadd -r electrumx

  useradd -r -m -d /var/lib/electrumx -k /dev/null -s /bin/false -g electrumx electrumx

  chown electrumx:electrumx /opt/electrumx/db

  cp contrib/systemd/electrumx.service /etc/systemd/system/

  ln -sf /opt/electrumx/electrumx.conf /etc/electrumx.conf

Create and edit config::

  nano /opt/electrumx/electrumx.conf

Config Example::


COIN = Bitweb
DB_DIRECTORY = /opt/electrumx/db
DAEMON_URL = http://rpcuser:rpcpassword@127.0.0.1:26332/
DB_ENGINE = rocksdb
SERVICES = tcp://:20001,rpc://:8000,ssl://20002,wss://:20003
EVENT_LOOP_POLICY = uvloop
PEER_DISCOVERY = off
INITIAL_CONCURRENT = 50
COST_SOFT_LIMIT = 5000000
COST_HARD_LIMIT = 50000000
BANDWIDTH_UNIT_COST = 1000
MAX_SESSIONS = 1000
SESSION_TIMEOUT = 600
SSL_CERTFILE = /opt/electrumx/ssl/fullchain.pem
SSL_KEYFILE = /opt/electrumx/ssl/privkey.pem



Give access to config::

  chown root:electrumx /opt/electrumx/electrumx.conf
  chmod 640 /opt/electrumx/electrumx.conf

Install server::

  python3.12 -m venv /opt/electrumx/venv
  source /opt/electrumx/venv/bin/activate
  pip install --upgrade pip setuptools wheel
  pip install .

create token at cloudflare

then::

  apt install python3-certbot-dns-cloudflare -y
  nano /etc/letsencrypt/cloudflare.ini
  dns_cloudflare_api_token = token
  chmod 600 /etc/letsencrypt/cloudflare.ini

  mkdir -p /etc/letsencrypt/renewal-hooks/deploy
  nano /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
  cat /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh

  bash#!/bin/bash
  systemctl reload nginx

  chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh

  certbot certonly \
    --dns-cloudflare \
    --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
    --dns-cloudflare-propagation-seconds 60 \
    --force-renewal \
    -d electrumx.example.com \
    -d electrumx1.example.com \
    -d electrumx2.example.com
  
add to nginx::

    /etc/nginx/sites-available/default-drop

    server {
        listen 80 default_server;
        listen 443 ssl default_server;
        
        ssl_certificate     /etc/letsencrypt/live/electrumx.bitwebcore.net/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/electrumx.bitwebcore.net/privkey.pem;
        
        server_name _;
        
        return 444;
    }



Start::

  systemctl start electrumx

Stop::

  systemctl stop electrumx

Autorun::

  systemctl enable electrumx

Logs::

  journalctl -u electrumx -f
