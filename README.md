# Fleet Baseline with Ansible

**An Ansible role that brings trading-fleet nodes to a known, idempotent baseline — standard packages, a locked-down service user, a per-group templated config, and a managed systemd service — with group variables tuning the same role differently for each tier of the fleet.**

Terraform provisions servers; configuration management decides what's on them. The core idea here is idempotence: a playbook describes the desired *state*, so a second run against a converged node should change nothing, and running it on a schedule corrects drift automatically.

## How it's organized

```
inventory.ini              trading_nodes (node01–03) + gateways (gw01), grouped under `fleet`
site.yml                   maps the trading_nodes group to the baseline role
group_vars/
  trading_nodes.yml        8 workers · 4096 connections · log level warn
  gateways.yml             2 workers · 1024 connections · log level info
roles/baseline/
  defaults/main.yml        lowest-priority defaults: packages, service user, tuning
  tasks/main.yml           the baseline itself
templates/app.conf.j2      config rendered per host from variables
docs/adhoc.md              ad-hoc commands (ping, one-off package installs)
docs/converge.md           what first and second runs should look like
```

## What the baseline does

| Task | Module | Desired state |
|---|---|---|
| Standard packages | `ansible.builtin.apt` | `htop`, `chrony`, `net-tools` present (cache refreshed) |
| Service user | `ansible.builtin.user` | `trader` exists as a system user with `nologin` shell and no home |
| App config | `ansible.builtin.template` | `/etc/trading/app.conf` rendered from `app.conf.j2`, owned by the service user; a change notifies a service restart |
| Service | `ansible.builtin.systemd` | `trading-gateway` enabled and started |

Variable precedence does the per-tier tuning: role defaults are overridden by `group_vars`, so the same template renders differently per group. For `node01` the rendered config is:

```ini
[service]
name = trading-gateway
bind_address = 10.0.1.11
worker_threads = 8
max_connections = 4096

[logging]
level = warn
```

(Rendered with `ansible-core` from this repo's inventory and group variables.)

## Running it

Requires `ansible-core` on the control machine and SSH access as `ops` to Ubuntu hosts with Python 3. Edit `inventory.ini` to point at real hosts, then:

```bash
ansible fleet -i inventory.ini -m ping                 # connectivity check
ansible-playbook -i inventory.ini site.yml --check     # dry run
ansible-playbook -i inventory.ini site.yml             # converge
```

The playbook passes `ansible-playbook --syntax-check`. Run it a second time to confirm idempotence — the recap should report `changed=0`.

## Limitations

These need fixing before a run against a fresh node succeeds:

- **The restart handler doesn't exist.** The template task notifies `Restart trading-gateway`, but the role has no `handlers/main.yml`, so Ansible fails when the config changes.
- **`/etc/trading` is never created**, so the template task fails on a fresh host.
- **The `trading-gateway` systemd unit isn't installed** by the role, so the service task assumes it already exists.
- `site.yml` only targets `trading_nodes`; the `gateways` group has variables but no play.
- The inventory uses placeholder `10.0.1.x` addresses. SSH key distribution and sysctl tuning, mentioned in earlier descriptions, aren't implemented.
- `docs/converge.md` shows the expected shape of a run, not captured output.
