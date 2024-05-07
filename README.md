# graylog

## Installation Debian 12 Stand 07.05.2024

**Required packages**
> apt install wget gnupg2 pwgen

### Elastic Search

**Initial repo an install**
> wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor > /etc/apt/trusted.gpg.d/elasticsearch-7.gpg
> echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
> apt update && apt install -y elasticsearch=7.10.2

**Configure logpath**
> echo "cluster.name: graylog path.data: /var/lib/elasticsearch path.logs: /var/log/elasticsearch" | tee /etc/elasticsearch/elasticsearch.yml

**Configure Daemon**
> systemctl daemon-reload
> systemctl enable elasticsearch.service
> systemctl restart elasticsearch.service

**Check if running**
> systemctl status elasticsearch

### MongoDB

**Initial repo and install**
> wget -qO https://www.mongodb.org/static/pgp/server-7.0.asc | gpg --dearmor > /usr/share/keyrings/mongodb-server-7.0.gpg
> echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" | tee /etc/apt/sources.list.d/mongodb-org-7.0.list
> apt update && apt install mondodb-org

**Configure Daemon**
> systemctl daemon-reload
> systemctl enable mongod
> systemctl restart mongod

**Check if running**
> systemctl status mongod

### Graylog

**Download and install repos and package**
> wget https://packages.graylog2.org/repo/packages/graylog-5.1-repository_latest.deb
> dpkg -i graylog-5.1-repository_latest.deb
> apt update && apt install graylog-server

**Configure password and secret**
> cp /etc/graylog/server/server.conf /etc/graylog/server/server.conf.original

This Password will create for the admin user
> PW=$(pwgen -N 1 -s 15) && echo "Your Admin password (Note this\!): $PW" && PW=$(echo $PW | sha256sum | awk '{print $1}') && sed -i "/^root_password_sha2/c\root_password_sha2 = $PW" /etc/graylog/server/server.conf
This secret will create for the other graylog server
> PW=$(pwgen -N 1 -s 96) && sed -i "/^password_secret/c\password_secret = $PW" /etc/graylog/server/server.conf

**Configure Daemon**
> systemctl daemon-reload
> systemctl enable graylog-server
> systemctl restart graylog-server
