# 🔒 Palo Alto Firewall Configuration Backup — Ansible Automation

An Ansible playbook that automates the secure download, validation, and remote transfer of running configurations from Palo Alto firewalls using the PAN-OS REST API and SMB.

Built from real-world experience managing enterprise firewall infrastructure at scale.

---

## 📋 What It Does

| Step | Action |
|------|--------|
| 1 | Creates a local backup directory with secure permissions |
| 2 | Authenticates to the Palo Alto firewall via PAN-OS API and downloads the running configuration |
| 3 | Validates the downloaded file is well-formed XML |
| 4 | Formats the XML for human readability using `xmllint` |
| 5 | Transfers the backup to a remote SMB file share |
| 6 | Cleans up local backups older than 7 days |

---

## 🗂️ Project Structure

```
palo-alto-backup/
├── ansible.cfg                  # Ansible configuration
├── download_config.yaml         # Main playbook
├── inventory/
│   └── hosts                    # Inventory (localhost)
├── vars/
│   └── secrets.yaml.example     # Secrets template (safe to commit)
│   └── secrets.yaml             # Your encrypted secrets (git-ignored)
└── .gitignore
```

---

## 🔐 Security Design

All sensitive values — firewall IP, API key, SMB credentials — are stored in an **Ansible Vault encrypted file** and never hardcoded in the playbook.

The `no_log: true` directive on the SMB transfer task prevents credentials from appearing in Ansible output or logs.

Backup XML files are excluded from version control via `.gitignore`.

---

## ⚙️ Prerequisites

| Requirement | Notes |
|-------------|-------|
| Ansible 2.9+ | `pip install ansible` |
| `xmllint` | `apt install libxml2-utils` / `yum install libxml2` |
| `smbclient` | `apt install smbclient` / `yum install samba-client` |
| PAN-OS API key | Generated from your Palo Alto firewall |
| SMB share access | Write permissions on the destination share |

---

## 🚀 Setup & Usage

### 1. Clone the repository

```bash
git clone https://github.com/yusbad09/palo-alto-backup.git
cd palo-alto-backup
```

### 2. Configure your secrets

```bash
cp vars/secrets.yaml.example vars/secrets.yaml
```

Edit `vars/secrets.yaml` and fill in your values:

```yaml
vault_firewall_ip: "10.0.0.1"
vault_api_key: "YOUR_PANOS_API_KEY"
vault_smb_share: "//192.168.1.100/Backups"
vault_smb_user: "DOMAIN\\username"
vault_smb_password: "YOUR_PASSWORD"
vault_smb_remote_path: "Backups/PaloAlto/2024"
```

### 3. Encrypt your secrets with Ansible Vault

```bash
ansible-vault encrypt vars/secrets.yaml
```

You will be prompted to set a vault password. **Keep this password safe — without it you cannot decrypt your secrets.**

### 4. Generate a PAN-OS API key

```bash
curl -k -X GET \
  "https://<FIREWALL_IP>/api/?type=keygen&user=<USERNAME>&password=<PASSWORD>"
```

Copy the `<key>` value from the XML response into `vault_api_key`.

### 5. Run the playbook

```bash
ansible-playbook download_config.yaml
```

You will be prompted for your Vault password. To run non-interactively (e.g. via cron):

```bash
ansible-playbook download_config.yaml --vault-password-file ~/.vault_pass
```

### 6. Run specific steps using tags

```bash
# Only download
ansible-playbook download_config.yaml --tags download

# Only transfer
ansible-playbook download_config.yaml --tags transfer

# Skip cleanup
ansible-playbook download_config.yaml --skip-tags cleanup
```

---

## ⏰ Scheduling with Cron

To run daily at 2:00 AM:

```bash
crontab -e
```

Add:

```
0 2 * * * cd /path/to/palo-alto-backup && ansible-playbook download_config.yaml --vault-password-file ~/.vault_pass >> /var/log/pa_backup.log 2>&1
```

---

## 🔧 Improvements Over Basic Implementation

| Original Approach | This Playbook |
|-------------------|---------------|
| Hardcoded credentials in script | Ansible Vault encrypted secrets |
| `curl` shell command | Native `uri` module (idempotent, no shell injection risk) |
| No credential masking | `no_log: true` on sensitive tasks |
| No XML validation | `xmllint --noout` validates before formatting |
| No cleanup | Automatically removes backups older than 7 days |
| No tagging | Tagged tasks allow partial runs |

---

## 📌 Real-World Context

This playbook was developed and used in a production enterprise environment managing Palo Alto PA-5450 firewalls across a large-scale government network infrastructure. It was part of a broader security operations automation initiative to ensure daily configuration backups without manual intervention.

---

## 🛡️ Related Skills

`Ansible` · `Palo Alto PAN-OS` · `Network Security` · `Security Automation` · `Python` · `Linux` · `SMB` · `Infrastructure-as-Code` · `Firewall Management`

---

## 📄 License

MIT License — free to use and adapt with attribution.

---

## 👤 Author

**Yusuf Akinkunmi Badrudeen**  
Cybersecurity & Cloud Security Engineer  
[LinkedIn](https://www.linkedin.com/in/badrudeen-yusuf-akinkunmi-6692b819b/) · [Portfolio](https://yusbad09.github.io/)
