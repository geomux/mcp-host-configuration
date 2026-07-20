# mcp-host-setup

Ansible playbook that configures an existing host with a remote MCP server and an nginx reverse proxy. Ideal for a box you already own (an EC2 instance, other established VM) that you want accesible by MCP tools in one command, no containers involved.

*Intended for hosts you control and can SSH into. This repo builds no infrastructure, images/containers, it only configures what already exists. If you want a containerized MCP tool option instead, see mcp-sandbox-setup repo.*

**Control Node OS: Linux, macOS, WSL.**

**Target Node OS: Linux (Ubuntu/Debian)**

```
System:  {user} <--> mcp-client-console <--> internet <--> {host:80} <--> nginx <--> mcp-server-remote <--> tools
Stack:   site.yaml <--> roles/mcp_server + roles/nginx <--> templates/*.j2 <--> group_vars/all.yaml
```

## Repo Layout

| File / Folder             | Purpose                                                               |
| ------------------------- | --------------------------------------------------------------------- |
| `site.yaml`                | The playbook, runs both roles against your inventory                  |
| `ansible.cfg`             | Points Ansible at the inventory, contains defaults                    |
| `inventory/hosts.ini`     | The hosts to configure (IP, SSH user, key)                            |
| `group_vars/all.yaml`      | Your settings: server name, port, auth token (edit before first run)  |
| `roles/mcp_server/`       | Installs mcp-server-remote, writes config.toml, adds a systemd unit   |
| `roles/nginx/`            | Installs nginx and proxies host port 80 to the server port            |

## User Guide | Installation

Requires Ansible on your workstation and SSH access to the target host. The target host box just needs Python installed.

```bash
git clone https://github.com/geomux/mcp-host-setup.git
cd mcp-host-setup
# edit inventory/hosts.ini and group_vars/all.yaml first (see Configuration below)
ansible-playbook site.yaml
```
**Install for Ansible prior (if needed)**
```bash
pipx install --include-deps ansible
```

## User Guide | Configuration

Point the inventory at your host:

```ini
[mcp_hosts]
box1 ansible_host=203.0.113.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/mykey.pem
```

Then set your values in `group_vars/all.yaml`:

```yaml
mcp_server_name: "Box_1"   # Label for this machine
mcp_server_port: 9000      # Port nginx proxies to
mcp_server_path: "/mcp"    # Leave this alone. /mcp is default for dependencies.
mcp_auth_token: ""         # openssl rand -hex 32, same drill as the server repo
```

The server binds 127.0.0.1 on the host, only nginx faces the internet. Rerun the playbook after any changes, Ansible will only touch what actually changed.

## User Guide | Operation

| Command                                   | What it does                                          |
| ----------------------------------------- | ------------------------------------------------------ |
| `ansible all -m ping`                     | Confirm SSH connectivity before doing anything else    |
| `ansible-playbook site.yaml --check --diff`| Dry run, shows what would change without changing it   |
| `ansible-playbook site.yaml`               | Configure the host (safe to rerun anytime)             |
| `ansible-playbook site.yaml --tags nginx`  | Rerun just the nginx role                              |

Two ways to use this repo: 
1) SSH option... (run the playbook from your workstation against a live host)
2) Terraform option... (a recipe creates the host, then calls this playbook to configure it)

## Related / Required Repos

- [mcp-server-remote](https://github.com/geomux/mcp-server-remote)
- [mcp-client-console](https://github.com/geomux/mcp-client-console)
- [mcp-sandbox-setup](https://github.com/geomux/mcp-sandbox-setup)

## Project Status

- [x] Create host setup repo
- [x] Write inventory and group_vars
- [x] Write mcp_server role (install, config.toml, systemd unit)
- [x] Write nginx role (reverse proxy to the server port)
- [ ] Run end to end against a live EC2 host
- [ ] Call the playbook from a Terraform recipe
- [ ] Harden (TLS on nginx, firewall rules)
