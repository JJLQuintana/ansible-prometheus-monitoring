# Ansible Prometheus Monitoring

Ansible role that installs Prometheus on both Ubuntu and CentOS servers, as part of a lab on performance monitoring with Infrastructure as Code.

## What this covers
- Installing Prometheus with a role-based playbook
- Distribution-specific install paths: the `apt` package on Ubuntu, and `snap` on CentOS
- Preparing CentOS with the `epel-release` and `snapd` prerequisites and enabling the snapd socket
- Running system updates in `pre_tasks` before the role

## Lab environment
- Control node: Ubuntu workstation running Ansible
- Managed nodes: one Ubuntu server and one CentOS server (VirtualBox, host-only network)

## Repository structure
```
.
├── ansible.cfg
├── inventory
├── prom.yml            # Main playbook (pre_tasks + role)
└── roles/
    └── remote_server/
        └── tasks/
            └── main.yml
```

## How it works
`prom.yml` runs two plays against all hosts:
1. `pre_tasks` update the package index and installed packages.
2. The `remote_server` role installs Prometheus:
   - **Ubuntu:** installs the `prometheus` package with `apt`.
   - **CentOS:** installs `epel-release` and `snapd`, enables `snapd.socket`, then runs `snap install prometheus --classic`.

## Usage
```bash
ansible-playbook --ask-become-pass prom.yml
```

## Verification
- Ubuntu: `prometheus --version` reports 2.31.2 (distro package), and the web UI loads at `http://<host>:9090`.
- CentOS: the Prometheus web UI loads at `http://<host>:9090`.

## Notes and next steps
- The snap install uses the `command` module, which is not idempotent. The `community.general.snap` module would be a better fit.
- Scrape targets are not configured, so the UI shows no data. Adding a `prometheus.yml` with targets (for example `node_exporter`) is the natural next step.
- Pair with Grafana for dashboards.
