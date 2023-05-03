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
  version: v1.1.0
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
  - { name: "exporter_down", condition: "up == 0", duration: "5m", severity: "major", summary: "Exporter down" }

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
```
