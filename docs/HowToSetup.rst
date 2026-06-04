OS Ubuntu 24.04


Prepare::

  sudo -s

  apt-get update && apt-get upgrade

  apt install -y python3.12 python3.12-venv python3.12-dev python3-pip python3-setuptools python3-multidict libleveldb-dev gcc g++ libsnappy-dev zlib1g-dev libbz2-dev libgflags-dev build-essential git

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
  DAEMON_URL = http://bitweb:bitwebpass@127.0.0.1:26332/
  SERVICES = tcp://:20001,rpc://:8000,ws://:20004
  EVENT_LOOP_POLICY = uvloop
  PEER_DISCOVERY = off
  INITIAL_CONCURRENT = 50
  COST_SOFT_LIMIT = 5000000
  COST_HARD_LIMIT = 50000000
  BANDWIDTH_UNIT_COST = 1000
  MAX_SESSIONS = 1000
  SESSION_TIMEOUT = 600


Give access to config::

  chown root:electrumx /opt/electrumx/electrumx.conf
  chmod 640 /opt/electrumx/electrumx.conf

Install server::

  python3.12 -m venv /opt/electrumx/venv
  source /opt/electrumx/venv/bin/activate
  pip install --upgrade pip setuptools wheel
  pip install .

go to nginx end create

electrumxs.example.com::

  server {
      listen 20003 ssl;
      server_name electrumx.example.com electrumx1.example.com electrumx2.example.com;
  
      ssl_certificate     /etc/letsencrypt/live/electrumx.example.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/electrumx.example.com/privkey.pem;
      ssl_protocols       TLSv1.2 TLSv1.3;
  
      location / {
          proxy_pass http://127.0.0.1:20004;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_read_timeout 600s;
      }
  }

next got to::

  user www-data;
  worker_processes 4;
  pid /run/nginx.pid;
  error_log /var/log/nginx/error.log;
  include /etc/nginx/modules-enabled/*.conf;

  events {
  	worker_connections 1024;
  	multi_accept on;
      use epoll;
  }

  stream {
      server {
          listen 20002 ssl;
          ssl_certificate     /etc/letsencrypt/live/electrumx.example.com/fullchain.pem;
          ssl_certificate_key /etc/letsencrypt/live/electrumx.example.com/privkey.pem;
          ssl_protocols       TLSv1.2 TLSv1.3;
          proxy_pass 127.0.0.1:20001;
      }
  }

  http {

      ##
      # Basic Settings
      ##

      sendfile on;
      tcp_nopush on;
      tcp_nodelay on;
      types_hash_max_size 2048;
      server_tokens off;

      # server_names_hash_bucket_size 64;
      # server_name_in_redirect off;

      include /etc/nginx/mime.types;
      default_type application/octet-stream;

      ##
      # SSL Settings
      ##

      ssl_protocols TLSv1.2 TLSv1.3;
      ssl_prefer_server_ciphers on;
      ssl_session_cache shared:SSL:10m;
      ssl_session_timeout 10m;

      # Keepalive
      keepalive_timeout 65;
      keepalive_requests 1000;

      ##
      # Logging Settings
      ##

      access_log off;
      error_log /var/log/nginx/error.log warn;

      ##
      # Gzip Settings
      ##

      gzip on;

      gzip_vary on;
      gzip_comp_level 4;
      gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

      proxy_buffer_size 16k;
      proxy_buffers 4 32k;
      proxy_busy_buffers_size 64k;

      ##
      # Virtual Host Configs
      ##

      include /etc/nginx/conf.d/*.conf;
      include /etc/nginx/sites-enabled/*;
  }

create token at cloudflare

then::

  apt install python3-certbot-dns-cloudflare -y
  nano /etc/letsencrypt/cloudflare.ini
  dns_cloudflare_api_token = token
  
  chmod 600 /etc/letsencrypt/cloudflare.ini
  
  certbot certonly \
    --dns-cloudflare \
    --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
    --dns-cloudflare-propagation-seconds 60 \
    --force-renewal \
    -d electrumx.example.com \
    -d electrumx1.example.com \
    -d electrumx2.example.com
  
if not exit add to::

  nano /etc/letsencrypt/renewal/electrumx.example.com.conf
  
  renew_hook = systemctl reload nginx

  nano /etc/letsencrypt/renewal/electrumx.example.com.conf

  dns_cloudflare_propagation_seconds = 60


Start::

  systemctl start electrumx

Stop::

  systemctl stop electrumx

Autorun::

  systemctl enable electrumx

Logs::

  journalctl -u electrumx -f
