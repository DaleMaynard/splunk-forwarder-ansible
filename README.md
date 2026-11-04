# splunk-forwarder-ansible

An Ansible role that installs and configures the **Splunk Universal Forwarder** on Ubuntu, then proves it
connected to the indexer. Built as a lab to practice repeatable, idempotent infrastructure automation.

## What it does
- Creates a `splunk` service user and unpacks the forwarder from the Linux `.tgz`
- Seeds the forwarder admin account from an **Ansible Vault** secret (no plain-text passwords in the repo)
- Writes `outputs.conf` and `inputs.conf` from templates (indexer address and monitored files are variables)
- Enables start-on-boot with systemd and starts the service
- Restarts the forwarder only when configuration actually changed (handler)
- Fails the run unless the forwarder logs a connection to the indexer

## Requirements
- Ubuntu target (WSL2 with systemd works), Ansible 2.14+
- The Linux 64-bit forwarder `.tgz` from splunk.com saved as `/tmp/splunkforwarder.tgz` (or set `splunk_uf_tarball`)
- A Splunk Enterprise receiving port (default 9997) reachable from the target

## Run it
```bash
cp group_vars/all/vault.yml.example group_vars/all/vault.yml   # set a real password, then:
ansible-vault encrypt group_vars/all/vault.yml
ansible-playbook -i inventory.ini site.yml --ask-vault-pass --ask-become-pass
```
Run it a second time: a healthy, idempotent run reports `changed=0`.

## Key variables (roles/splunk_uf/defaults/main.yml)
| Variable | Default | Purpose |
|---|---|---|
| `splunk_uf_indexer_host` | default gateway | Where to send data (from WSL2, the Windows host) |
| `splunk_uf_indexer_port` | 9997 | Splunk receiving port |
| `splunk_uf_monitors` | dpkg.log, syslog | Files to watch, with index and sourcetype |
| `splunk_uf_enable_boot_start` | true | Use systemd; set false on hosts without it |

## Checks
`ansible-playbook --syntax-check` and `ansible-lint` run on every push (see `.github/workflows/lint.yml`).
