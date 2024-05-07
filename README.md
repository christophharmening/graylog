# graylog

## Installation Debian 12 Stand 07.05.2024

**Gundlegende Pakete**
> apt install wget gnupg2

### Elastic Search

**Repo einrichten und installieren**
> wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor > /etc/apt/trusted.gpg.d/elasticsearch-7.gpg
> echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
> apt update && apt install -y elasticsearch=7.10.2

**Logpfad einstellen**
> echo "cluster.name: graylog path.data: /var/lib/elasticsearch path.logs: /var/log/elasticsearch" | tee /etc/elasticsearch/elasticsearch.yml

**Dienst Verwalten**
> systemctl daemon-reload
> systemctl enable elasticsearch.service
> systemctl restart elasticsearch.service

**Prüfen**
> systemctl status elasticsearch
