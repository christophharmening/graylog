# graylog

In this tutorial i used the tls encryption from rsyslog to send my logs over rsyslog to my graylog encrypted.

## Installation Debian 12 Stand 07.05.2024

**Required packages**
```
apt install wget gnupg2 pwgen rsyslog rsyslog-gnutls gnutls-bin
```

### Elastic Search

**Initial repo an install**
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor > /etc/apt/trusted.gpg.d/elasticsearch-7.gpg
```
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```
```
apt update && apt install -y elasticsearch=7.10.2
```

**Configure logpath**
```
echo "cluster.name: graylog path.data: /var/lib/elasticsearch path.logs: /var/log/elasticsearch" | tee /etc/elasticsearch/elasticsearch.yml
```

**Configure Daemon**
```
systemctl daemon-reload && systemctl enable elasticsearch.service &&systemctl restart elasticsearch.service
```

**Check if running**
```
systemctl status elasticsearch
```

### MongoDB

**Initial repo and install**
```
wget -qO https://www.mongodb.org/static/pgp/server-7.0.asc | gpg --dearmor > /usr/share/keyrings/mongodb-server-7.0.gpg
```
```
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" | tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
```
apt update && apt install mondodb-org
```

**Configure Daemon**
```
systemctl daemon-reload && systemctl enable mongod && systemctl restart mongod
```

**Check if running**
```
systemctl status mongod
```

### Graylog

**Download and install repos and package**
```
wget https://packages.graylog2.org/repo/packages/graylog-5.1-repository_latest.deb
```
```
dpkg -i graylog-5.1-repository_latest.deb
```
```
apt update && apt install graylog-server
```

**Configure password and secret**
```
cp /etc/graylog/server/server.conf /etc/graylog/server/server.conf.original
```

This Password will create for the admin user
```
PW=$(pwgen -N 1 -s 15) && echo "Your Admin password (Note this): $PW" && PW=$(echo $PW | sha256sum | awk '{print $1}') && sed -i "/^root_password_sha2/c\root_password_sha2 = $PW" /etc/graylog/server/server.conf
```
This secret will create for the other graylog server
```
PW=$(pwgen -N 1 -s 96) && sed -i "/^password_secret/c\password_secret = $PW" /etc/graylog/server/server.conf
```

**Configure Daemon**
```
systemctl daemon-reload && systemctl enable graylog-server &&systemctl restart graylog-server
```

## Enable TLS with rsyslog
### Install Certs and CA

Create bin for certs
```
mkdir /etc/rsyslog.d/ssl
cd /etc/rsyslog.d/ssl
```

Create CA
```
certtool --generate-privkey --outfile ca-rsyslog.key
```
```
certtool --generate-self-signed --load-privkey ca-rsyslog.key --outfile ca-rsyslog.pem
```

Create cert for rsyslog server
```
certtool --generate-privkey --outfile rsyslog-server.key --bits 2048
```
```
certtool --generate-request --load-privkey rsyslog-server.key --outfile rsyslog-server.req
```
```
certtool --generate-certificate --load-request rsyslog-server.req --outfile rsyslog-server.pem --load-ca-certificate ca-rsyslog.pem --load-ca-privkey ca-rsyslog.key
```

Create config file for rsyslog server
```
nano /etc/rsyslog.d/60-server.conf
```
```
global(
DefaultNetstreamDriver="gtls"
DefaultNetstreamDriverCAFile="/etc/rsyslog.d/ssl/ca-rsyslog.pem"
DefaultNetstreamDriverCertFile="/etc/rsyslog.d/ssl/rsyslog-server.pem"
DefaultNetstreamDriverKeyFile="/etc/rsyslog.d/ssl/rsyslog-server.key"
)

# load TCP listener
module(
load="imtcp"
StreamDriver.Name="gtls"
StreamDriver.Mode="1"
StreamDriver.Authmode="anon"
)

# start up listener at port 6514
input(
type="imtcp"
port="6514"
)
```

create second file for graylog
```
nano /etc/rsyslog.d/20-graylog.conf
```
```
*.*@127.0.0.1:5148;RSYSLOG_SyslogProtocol23Format
```

Restart server
```
systemctl restart rsyslog
```

## Configure rsyslog clients
On clients install rsyslog
```
apt install rsyslog rsyslog-gnutls
```

Copy CA cert from server to client
```
scp /etc/rsyslog.d/ca-rsyslg.pem root@CLIENT:/etc/rsyslog.d/
```

Create Client config file
```
nano /etc/rsyslog.d/60-remote-rsyslog.conf
```
```
# certificate files - just CA for a client
global(DefaultNetstreamDriverCAFile="/etc/rsyslog.d/ca-rsyslog.pem")

# set up the action for all messages
action(type="omfwd" protocol="tcp" target="SERVER" port="6514"
       StreamDriver="gtls" StreamDriverMode="1" StreamDriverAuthMode="anon")
```

Restart rsyslog
```
systemctl restart rsyslog
```

## Configure Graylog Stream

Open Browser and login http://SERVER:9000

Goto System/Inputs -> Inputs
Select Input "Syslog UDP" and "Launch Input"

Config Input like this

![ScreenShot](https://github.com/christophharmening/graylog/blob/main/GraylogRsyslogInput.png)

