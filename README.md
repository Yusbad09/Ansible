# 🐧 Multi-Distro Linux Server Patch Management — Ansible Automation

An Ansible playbook that automates package updates across multiple Linux distributions simultaneously — detecting each server's OS family at runtime and applying the correct update method automatically.

Built from real-world experience managing a heterogeneous server estate across Ubuntu, Kali Linux, and CentOS environments.

---

## 📋 What It Does

| Step | Action |
|------|--------|
| 1 | Connects to all servers in parallel and gathers OS facts |
| 2 | Detects OS family (Debian-based or RHEL-based) automatically |
| 3 | Refreshes the package cache for each distro |
| 4 | Checks and reports how many packages are available for update |
| 5 | Upgrades all installed packages to their latest available version |
| 6 | Saves a per-server update log with date, OS, and result |

---

## 🖥️ Supported Distributions

| Distribution | Package Manager | OS Family |
|---|---|---|
| Ubuntu (all LTS versions) | `apt` | Debian |
| Kali Linux | `apt` | Debian |
| CentOS 7 / 8 | `yum` | RedHat |
| RHEL | `yum` | RedHat |
| Debian | `apt` | Debian |
| Amazon Linux | `yum` | RedHat |

> Adding support for a new distro is as simple as adding it to the inventory — the playbook auto-detects the OS family using Ansible facts.

---

## 🗂️ Project Structure

```
linux-patch-management/
├── ansible.cfg               # Ansible configuration (parallelism, logging)
├── update_servers.yaml       # Main playbook
├── inventory/
│   └── hosts                 # Server inventory grouped by distro
└── .gitignore
```

---

## ⚙️ Prerequisites

| Requirement | Notes |
|-------------|-------|
| Ansible 2.9+ | `pip install ansible` |
| SSH access | Key-based auth recommended (no password prompts) |
| sudo privileges | Required on all target servers |
| Python 3 | Must be installed on all target servers |

---

## 🚀 Setup & Usage

### 1. Clone the repository

```bash
git clone https://github.com/yusbad09/linux-patch-management.git
cd linux-patch-management
```

### 2. Configure your inventory

Edit `inventory/hosts` and replace the example IPs with your actual server addresses:

```ini
[ubuntu_servers]
ubuntu-srv-01 ansible_host=YOUR_SERVER_IP

[centos_servers]
centos-srv-01 ansible_host=YOUR_SERVER_IP
```

### 3. Set up SSH key authentication

```bash
# Generate a dedicated Ansible SSH key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_rsa -C "ansible service key"

# Copy to each target server
ssh-copy-id -i ~/.ssh/ansible_rsa.pub ansible_svc@YOUR_SERVER_IP
```

### 4. Test connectivity

```bash
ansible all_servers -m ping
```

Expected output:
```
ubuntu-srv-01 | SUCCESS => { "ping": "pong" }
centos-srv-01 | SUCCESS => { "ping": "pong" }
...
```

### 5. Run the playbook

```bash
# Full run — check and update all servers
ansible-playbook update_servers.yaml

# Check only — see what updates are available without applying
ansible-playbook update_servers.yaml --tags check

# Update only a specific group
ansible-playbook update_servers.yaml --limit ubuntu_servers

# Dry run (no changes made)
ansible-playbook update_servers.yaml --check
```

---

## 📊 Sample Output

```
TASK [Display detected OS information]
ok: [ubuntu-srv-01] => {
    "msg": "Host: ubuntu-srv-01 | OS Family: Debian | Distro: Ubuntu 22.04"
}
ok: [centos-srv-01] => {
    "msg": "Host: centos-srv-01 | OS Family: RedHat | Distro: CentOS 7.9"
}

TASK [[Debian] Show available update count]
ok: [ubuntu-srv-01] => { "msg": "14 package(s) available for update" }

TASK [Update complete — final status]
ok: [ubuntu-srv-01] => { "msg": "✔ ubuntu-srv-01 (Ubuntu) — update complete." }
ok: [centos-srv-01] => { "msg": "✔ centos-srv-01 (CentOS) — update complete." }
```

---

## ⏰ Scheduling with Cron

To run weekly patch updates automatically (every Sunday at 1:00 AM):

```bash
crontab -e
```

Add:
```
0 1 * * 0 cd /path/to/linux-patch-management && ansible-playbook update_servers.yaml >> /var/log/patch_management.log 2>&1
```

---

## 📈 Scalability

The playbook was implemented on a 10-server estate but is designed to scale with zero changes to the playbook itself:

- **Add servers** — drop them into the correct group in `inventory/hosts`
- **Add a new distro** — add it to inventory; if it's Debian or RHEL based it works immediately
- **Increase parallelism** — adjust `forks` in `ansible.cfg` (currently set to 10)
- **Target subsets** — use `--limit` to run against specific groups or individual hosts

---

## 🔧 Key Design Decisions

| Decision | Reason |
|---|---|
| `gather_facts: true` | Enables OS auto-detection via `ansible_os_family` |
| `cache_valid_time: 3600` | Avoids redundant apt cache refreshes within the same hour |
| `forks = 10` | All 10 servers update in parallel, not sequentially |
| Per-server log files | Easy audit trail — one log per host per day |
| `--tags` support | Allows check-only runs without applying changes |

---

## 📌 Real-World Context

This playbook was developed to manage patch compliance across a heterogeneous Linux server estate in an enterprise network security environment. It eliminated the need for manual per-server updates and ensured consistent patch levels across all systems regardless of distribution.

---

## 🛡️ Related Skills

`Ansible` · `Linux Administration` · `Patch Management` · `System Hardening` · `Ubuntu` · `CentOS` · `Kali Linux` · `Security Automation` · `DevSecOps`

---

## 📄 License

MIT License — free to use and adapt with attribution.

---

## 👤 Author

**Yusuf Akinkunmi Badrudeen**  
Cybersecurity & Cloud Security Engineer  
[LinkedIn](https://www.linkedin.com/in/badrudeen-yusuf-akinkunmi-6692b819b/) · [Portfolio](https://yusbad09.github.io/)
