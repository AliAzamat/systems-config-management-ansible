# Ad-hoc commands

Before writing playbooks, you can drive Ansible one command at a time. The shape
is always: which hosts, which module, what arguments.

    ansible <group> -m <module> -a "<args>"

## Check connectivity to the whole fleet

    ansible fleet -m ping

The `ping` module isn't ICMP ping — it connects over SSH, runs a tiny Python
module on the target, and confirms the whole pipeline works. A green "pong"
means Ansible can actually manage that host.

## Install a package on the trading nodes (ad-hoc)

    ansible trading_nodes -m apt -a "name=htop state=present" --become

`--become` runs the task as root (sudo). `state=present` is the desired state:
"htop should be installed." Run it twice — the second run says "ok", not
"changed", because htop is already there. That's idempotence, even ad-hoc.
