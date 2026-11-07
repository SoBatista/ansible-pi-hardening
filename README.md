# pi-hardening

Ansible playbook that hardens a Raspberry Pi running **Debian 13 (trixie)**:

| Role | What it does |
|------|--------------|
| `preflight` | Refuses to run on the wrong OS, with leftover placeholders, or if the firewall would lock you out |
| `updates` | `apt update` + `apt full-upgrade` |
| `admin_user` | Ensures your admin user exists, is in `sudo`, and has your SSH public key |
| `ssh` | Disables password/keyboard-interactive login and root login |
| `firewall` | nftables, default-deny inbound; SSH only from your admin IP on one interface |
| `services` | Purges `avahi-daemon`, masks `rpcbind` |
| `unattended_upgrades` | Automatic security updates |
| `logging` | Persistent journald logs |

## ⚠️ What you must fill in

The inventories ship with placeholders like `"<INSERT YOUR PI's IP HERE>"`.
**Replace every `<INSERT ...>` value** before running. Everything else can stay as it is.

| Variable | What to put there |
|----------|-------------------|
| `ansible_host` | IP address of your Pi (or the Vagrant VM for dev) |
| `ansible_user` | User Ansible logs in as (prod: the user you created in Raspberry Pi Imager) |
| `admin_user_name` | Admin user to manage (usually the same as `ansible_user`) |
| `admin_user_ssh_key` | Your SSH **public** key, i.e. the contents of `~/.ssh/id_ed25519.pub` |
| `firewall_ssh_allowed_ipv4` | IP of the machine you SSH **from**. It's the only IP allowed to reach SSH |
| `firewall_ssh_iface` | `wlan0` if the Pi is on Wi-Fi, `eth0` if it's on cable |

> Never put a **private** key in these files, only the `.pub` one.

### Recommended: keep your values out of git

Instead of editing `hosts.yml`, create a host_vars file. It overrides the placeholders and
is already in `.gitignore`:

```yaml
# inventories/prod/host_vars/prod-pi.yml
---
ansible_host: 192.168.1.50
ansible_user: myuser
admin_user_name: myuser
admin_user_ssh_key: "ssh-ed25519 AAAA... me@laptop"
firewall_ssh_allowed_ipv4: 192.168.1.20
```

Use `inventories/dev/host_vars/dev-pi.yml` for the dev VM.

## Usage

Requirements: `ansible-core` on your machine. For dev you also need Vagrant + libvirt.

```bash
# Dev: test against a throwaway VM first (inventories/dev is the default)
vagrant up
vagrant ssh-config        # shows the VM's IP for ansible_host
ansible-playbook site.yml

# Prod: the real Pi
ansible-playbook -i inventories/prod site.yml --check --diff   # dry run
ansible-playbook -i inventories/prod site.yml
```

Keep an existing SSH session open to the Pi while you apply the `ssh` and `firewall` roles for the first time.

## Linting

```bash
yamllint .
ansible-lint
ansible-playbook --syntax-check site.yml
```
