# monitoring

## Role installation

### create requirements file

``` bash
cd ANSIBLE_FOLDER
mkdir requirements
touch requirements/monitoring.yml
```

### file content

``` yaml
---

# ansible-galaxy install -p vss_galaxy_roles --force -r requirements/monitoring.yml
- src: "https://github.com/virsas/mod-ansible-rpm-monitoring"
  scm: git
  version: v1.6.0
  name: monitoring
  path: vss_galaxy_roles
```

## Install role

### Installation

``` bash
ansible-galaxy install -p galaxy_roles --force -r requirements/monitoring.yml
```

### Best practices

If you are using git for your playbooks and sites configuration, add vss_galaxy_roles to your .gitignore file. You are free to download the role directly to your roles directory, but it will be then pushed to your repo too.

## Role access

### Direct access

``` bash
$ cd vss_galaxy_roles
$ ls
monitoring
$ cd monitoring
$ ls -1
defaults
handlers
LICENSE
meta
README.md
tasks
templates
$
```

### Access from ansible (ansible.cfg)

``` bash
[defaults]
gathering = smart
roles_path=./roles:../roles:../../roles:./vss_galaxy_roles
log_path=./ansible_run.log
retry_files_enabled=False
[ssh_connection]
scp_if_ssh=True
host_key_checking=False
```

## Files

### Playbook (./playbooks/monitoring.yml)

``` yaml
---

- hosts: monitoring
  remote_user: ec2-user
  become: yes
  roles:
    - monitoring
```

### Inventory (./sites/NAME/inventory)

``` txt
[monitoring]
monitoring_01
```

### Host vars (./sites/NAME/host_vars/monitoring_01.yml)

``` yaml
---

ansible_ssh_host: 1.1.1.1
```

### Group vars (./sites/NAME/group_vars/monitoring.yml)

Please see the variables below to get the complete list of possible modifications. The YAML file below is just a basic example of a minimal configuration.

``` yaml
PROMETHEUS_JOBS:
  -
    name: node_exporter
    scheme: http
    path: /metrics
    targets:
      - { name: node_exporter, targets: ['10.0.0.4:9100'] }


ALERTMANAGER_EMAIL_HOST: "smtp.gmail.com"
ALERTMANAGER_EMAIL_PORT: "587"
ALERTMANAGER_EMAIL_USER: "api@example.com"
ALERTMANAGER_EMAIL_PASS: "asdasdasdasdasd"
ALERTMANAGER_EMAIL_FROM: "api@example.com"
ALERTMANAGER_EMAIL_TO: "system@example.com"
```

## Usage

``` bash
ansible-playbook -i sites/NAME/inventory playbooks/monitoring.yml --diff
```

## Variables

``` yml
GRAFANA_VERSION: "8.4.2-1"
GRAFANA_ARCH: "aarch64"

GRAFANA_PROTOCOL: "http"
GRAFANA_PORT: "3000"
GRAFANA_ADMIN_USER: "admin"
GRAFANA_ADMIN_PASS: "admin"
GRAFANA_ADMIN_EMAIL: "system@example.org"
# allowed options "sqlite3", "postgres", "mysql"
# see optional configuration in case you choose postgres or mysql
GRAFANA_DB_TYPE: "sqlite3"

PROMETHEUS_VERSION: "2.33.4"
PROMETHEUS_ARCH: "arm64"
PROMETHEUS_IP: "0.0.0.0"
PROMETHEUS_PORT: "9090"
PROMETHEUS_URL: "http://localhost"
PROMETHEUS_RETENTION_TIME: "30d"
PROMETHEUS_RETENTION_SIZE: "0"
PROMETHEUS_INTERVAL: "60s"
PROMETHEUS_JOBS:
  -
    name: node_exporter
    scheme: http
    path: /metrics
    targets:
      - { name: node_exporter, targets: ['10.0.0.4:9100'] }
  -
    name: golang
    scheme: http
    path: /metrics
    targets:
      - { name: service_one, targets: ['10.0.0.4:2001', '10.0.0.5:2001'] }
      - { name: service_two, targets: ['10.0.0.4:2002', '10.0.0.5:2002'] }

PROMETHEUS_RULES:
  -
    group: "exporter"
    rules:
      -
        alert: "exporter_down"
        description: "Exporter down"
        severity: "major"
        duration: "5m"
        condition: "up == 0"
        labels:
          - { key: "team", value: "sysops" }
        annotations:
          - { key: "grafana", value: "https://monitoring.example.org:3000/d/abcdefghi/node-exporter-full?orgId=1" }

ALERTMANAGER_VERSION: "0.23.0"
ALERTMANAGER_ARCH: "arm64"
ALERTMANAGER_IP: "127.0.0.1"
ALERTMANAGER_PORT: "9093"
ALERTMANAGER_URL: "http://localhost"
ALERTMANAGER_EMAIL_HOST: "smtp.gmail.com"
ALERTMANAGER_EMAIL_PORT: "587"
ALERTMANAGER_EMAIL_USER: ""
ALERTMANAGER_EMAIL_PASS: ""
ALERTMANAGER_EMAIL_TLS: true
ALERTMANAGER_EMAIL_FROM: "alertmanager@localhost"
ALERTMANAGER_EMAIL_TO: "root@localhost"
ALERTMANAGER_TIMEOUT: "5m"
ALERTMANAGER_INTERVAL: "1m"
ALERTMANAGER_WAIT: "2m"
ALERTMANAGER_REPEAT: "24h"
```

## Optional variables

``` yml
# If you specify PROMETHEUS_STORAGE, the device will be formatted to ext4 and mounted to /var/lib/prometheus for data scraping
PROMETHEUS_STORAGE: "/dev/sdb"
# If you specify GRAFANA_STORAGE, the device will be formatted to ext4 and mounted to /var/lib/grafana for sqlite database
GRAFANA_STORAGE: "/dev/sdc"
# mysql / postgres
GRAFANA_DB_HOST: "localhost"
GRAFANA_DB_PORT: "5432"
GRAFANA_DB_DATABASE: "grafana"
GRAFANA_DB_USER: "grafana"
GRAFANA_DB_PASS: "asdasd"
# sqlite3
GRAFANA_DB_PATH: "/var/lib/grafana.db"
# additional receivers if used by additional routes
ALERTMANAGER_ADDITIONAL_RECEIVERS:
  -
    name: "frontend_email"
    type: "email_configs"
    config:
      - { key: "to", value: "frontend@example.org" }
# additional routes for alert manager
ALERTMANAGER_ADDITIONAL_ROUTES:
  -
    receiver: "frontend_email"
    matchers:
      - { label: "team", condition: "=~", value: "frontend" }
```
