
# Ansible Monitoring Lab

A lightweight Infrastructure-as-Code (IaC) project for deploying and managing a monitoring environment using Ansible.

## Overview

This repository provisions a monitoring lab consisting of:

* **Prometheus** – Metrics collection and storage
* **Node Exporter** – Host metrics exporter
* **Grafana** – Visualization and dashboards
* **Elastic Stack** *(planned)* – Logs and observability

The goal of this project is to build a fully automated monitoring stack without relying on Docker initially, in order to understand each component's installation and configuration process.

---

## Project Structure

```text
.
├── ansible.cfg
├── inventories
│   └── dev
│       ├── group_vars
│       │   └── all.yml
│       └── hosts.yml
├── playbooks
│   └── monitoring.yml
├── roles
│   ├── common
│   ├── prometheus
│   ├── node_exporter
│   ├── grafana
│   └── elastic_stack
└── README.md
```

---

## Architecture

```text
                    +----------------------+
                    |    Control Node      |
                    |----------------------|
                    | Prometheus           |
                    | Grafana              |
                    | Elastic (planned)    |
                    +----------+-----------+
                               |
                Scrape Metrics |
                               |
        +----------------------+----------------------+
        |                                             |
+---------------+                           +---------------+
| Node 01       |                           | Node 02       |
|---------------|                           |---------------|
| Node Exporter |                           | Node Exporter |
+---------------+                           +---------------+
```

---

## Requirements

Control machine:

* Ansible
* SSH access to all managed hosts
* Passwordless sudo 

Managed nodes:

* Ubuntu Server 24.04
* Python 3 installed
* SSH enabled (PubkeyAuthentication)

---

## Inventory Example

```yaml
all:
  children:
    controls:
      hosts:
        control01:
          ansible_host: 192.168.122.100

    nodes:
      hosts:
        node01:
          ansible_host: 192.168.122.110
```

---

## Variables

Shared variables are stored in:

```text
inventories/dev/group_vars/all.yml
```

Example:

```yaml
ansible_user: ansible

prometheus_port: 9090
node_exporter_port: 9100
grafana_port: 3000
```

---

## Running the Playbook

Deploy the entire monitoring environment:

```bash
ansible-playbook playbooks/monitoring.yml
```

Run against a specific inventory:

```bash
ansible-playbook \
    -i inventories/dev/hosts.yml \
    playbooks/monitoring.yml
```

---

## Roles

### common

Shared configuration for every managed node.

Responsibilities:

* Update APT cache
* Configure common system settings
* Install prerequisite packages

---

### prometheus

Installs and configures Prometheus.

Responsibilities:

* Create Prometheus user
* Download Prometheus
* Install binaries
* Deploy configuration
* Configure systemd
* Enable and start the service

---

### node_exporter

Installs and configures Prometheus Node Exporter.

Responsibilities:

* Create Node Exporter user
* Download Node Exporter
* Install binary
* Configure systemd
* Enable and start the service

---

## grafana

Installs Grafana and automatically provisions:

- Prometheus as the default data source.
- Dashboard providers.
- Pre-configured dashboards (Node Exporter Full).
- Grafana server service (enabled and started).

Once deployed, Grafana is immediately ready to visualize metrics collected by Prometheus without requiring any manual configuration.

---

### elastic_stack *(planned)*

Will deploy:

* Elasticsearch
* Kibana
* Elastic Agent
* Fleet Server

---

## Design Principles

This project emphasizes:

* Learning automation via Ansible
* Idempotent playbooks
* Small, reusable roles
* Consistent variable naming
* Minimal hardcoded values
* Clean project organization
* Infrastructure as Code

Each role follows the same lifecycle whenever possible:

```text
Create users
        ↓
Create directories
        ↓
Download package
        ↓
Extract
        ↓
Install binaries
        ↓
Deploy configuration
        ↓
Deploy service
        ↓
Enable service
        ↓
Cleanup
```

---

## Future Improvements

* Dynamic Prometheus service discovery
* Grafana dashboards provisioning
* Alertmanager integration
* TLS encryption
* Ansible Vault for secrets
* Multi-environment inventories (dev, staging, production)
* Docker deployment option
* CI/CD using GitHub Actions

---

## License

MIT License
