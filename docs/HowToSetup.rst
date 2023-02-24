OS Ubuntu 20.04


Prepare::

  sudo -s

  apt-get update && apt-get upgrade

  apt-get install python3-setuptools python3-multidict python3.8 python3.8-dev libleveldb-dev python3-setuptools python3-multidict gcc g++ libsnappy-dev zlib1g-dev libbz2-dev libgflags-dev build-essential python3-pip git

  python3.8 -m pip install aiohttp pylru plyvel Cython uvloop quark_hash bitweb_yespower==1.0.5

  important after enable GPU algo , use version bitweb_yespower==1.0.5 ( YescryptR16 )

  git clone https://github.com/bitweb-project/electrumx /opt/electrumx
  
  or 
  
  git clone https://github.com/spesmilo/electrumx /opt/electrumx ( if merged )

  cd /opt/electrumx

  mkdir -p db

  groupadd -r electrumx

  useradd -r -m -d /var/lib/electrumx -k /dev/null -s /bin/false -g electrumx electrumx

  chown electrumx:electrumx /opt/electrumx/db

  cp contrib/systemd/electrumx.service /etc/systemd/system/

  ln -sf /opt/electrumx/electrumx_server.py /usr/local/bin/electrumx_server.py

  ln -sf /opt/electrumx/electrumx.conf /etc/electrumx.conf

Create SSL certificat::

  mkdir -p ssl

  cd ssl

  openssl genrsa -out server.key 2048
  openssl req -new -key server.key -out server.csr
  openssl x509 -req -days 1825 -in server.csr -signkey server.key -out server.crt

Give access to ssl folser and cert::

  chown -R electrumx:electrumx /opt/electrumx/ssl

  cd ..

Create and edit config::

  nano /opt/electrumx/electrumx.conf

Config Example::

  COIN = Bitweb
  DB_DIRECTORY = /opt/electrumx/db
  DAEMON_URL = http://RPCUSER:RPCPASSWORD@IP:RPCPORT/
  SERVICES = tcp://:20001,rpc://:8001,ssl://:20002
  EVENT_LOOP_POLICY = uvloop
  PEER_DISCOVERY = self
  INITIAL_CONCURRENT = 50
  COST_SOFT_LIMIT = 10000
  COST_HARD_LIMIT = 100000
  BANDWIDTH_UNIT_COST = 10000
  SSL_CERTFILE = /opt/electrumx/ssl/server.crt
  SSL_KEYFILE = /opt/electrumx/ssl/server.key

Give access to config::

  chown root:electrumx /opt/electrumx/electrumx.conf
  chmod 640 /opt/electrumx/electrumx.conf

Install server::

  python3.8 setup.py install


Start::

  systemctl start electrumx

Stop::

  systemctl stop electrumx

Autorun::

  systemctl enable electrumx

Logs::

  journalctl -u electrumx -f
