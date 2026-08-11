# Config Management with Ansible: Bring a Fleet to a Known State

A systems project that utilizes configuration management the way ops teams actually run it. Where Terraform provisions the infrastructure, Ansible CONFIGURES the servers — and the whole discipline turns on one idea, idempotence. Starts from an inventory of nodes, runs ad-hoc command, then builds a playbook that installs standardized packages, creates a service user and SSH key, drops a config file from a Jinja2 template, manages a systemd service with a handler, and applies kernel/network sysctl tuning. Factors it into a reusable role and finishes by bringing a fresh node to a trusted baseline. The whole project is one realization — run the playbook twice and the second run reports changed=0, because configuration management describes a desired STATE, not a script of steps. Real, runnable Ansible YAML throughout.

## Stack
- Python
- Ansible
- YAML
- Jinja2
- systemd
- Ubuntu
