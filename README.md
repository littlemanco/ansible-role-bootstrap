# Ansible MOTD Role

This is the Ansible bootstrap role. It's designed for consumption by playbooks, not for consumption by itself.
It provisions a node with the tools to further self-provision.

## Requirements

- Systemd services
- Systemd timers

## Justification

This doesn't feel like a good idea, inherently. The problem is that if you're provisioning the server from itself:

- You cannot easily debug it
- You don't know when it goes wrong
- You don't know where it goes wrong

and so on. However, there are certain machines (such as monitoring displays) that don't need to have as much of a
guarantee around uptime, and if they break, can be trivially reprovisioned.

## Addressing the above flaws

### Debugging

The ansible playbooks are run by systemd, as a systemd service with a systemd timer. Logs will be pushed to the 
systemd journal, where log aggregation (or just journalctl) can read and introspect them. 

### Going Wrong

The service will express the status of the playbook to the location the Prometheus node exporter expects text metrics 
to exists, so that status can be consumed by monitoring.

## Usage

Include this in another ansible playbook. Generally, it should be one of the few roles in this playbook (the others
being monitoring)

Consider a generic server playbook:

```yaml
---
# $PLAYBOOK_ROOT/server.yaml
- name: "server"
  hosts: all
  become: true
  become_user: "root"
```

Add the reference for the role:

```yaml
# $PLAYBOOK_ROOT/server.yaml
# ...
become_user: "root"
roles
  - "bootstrap"
```

This will allow the role to be discovered. Then, add this repo as a submodule:

```bash
$ cd path/to/playbook/root
$ mkdir roles/
$ git submodule add https://github.com/littlemanco/ansible-role-bootstrap roles/bootstrap
```

This won't work just yet -- you will need to add some configuration:

```yaml
# $PLAYBOOK_ROOT/group_vars/$GROUP.yaml
---
# The URL that the playbook is accessible at. Auth is deliberately not handled by this role; do it in another.
bootstrap_playbook_url: "https://github.com/littlemanco/ansible-role-a-playbook"

# The frequency, expressed as a systemd timer, that you wish to reprovision. Suggestion would be every 15 minutes.
bootstrap_provision_frequency: ""
```

That's it! Run the playbook, and that machine should take care of itself.

## Contact

| Type     | Address                       |
|----------|-------------------------------|
| Contact: | hello@andrewhowden.com        |
| Web:     | https://www.andrewhowden.com/ |
