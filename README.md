# Zabbix Infrastructure Automation

This project automates the full setup of a Zabbix monitoring environment using Ansible.  
It deploys:
- A **custom internal YUM repository** shared over HTTP  
- A **Zabbix server** (backend + frontend + DB config)  
- A **Zabbix agent** on monitored hosts  

Everything is reproducible with one command.

---

## 🧩 Requirements

- RHEL 9 / Rocky 9 / AlmaLinux 9
- Ansible installed on control machine
- RPMs placed in `rpms/` directory
- SSH access to all hosts

---

## 📦 What This Repository Solves

- No internet required for installation  
- Fully automated Zabbix deployment  
- Repeatable and consistent across environments  
- Clear separation of tasks using proper Ansible roles  

---

## 🚀 How the Automation Works

### **1️⃣ Prepare Internal Repository**
The first playbook turns the Localhost into a mini web server hosting an internal YUM repo:

- Installs Apache (`httpd`)
- Creates directory `/var/www/html/repo`
- Copies all RPMs into it
- Runs `createrepo` to generate metadata
- Fixes SELinux labels
- Opens firewall port 80

All other machines install Zabbix from this internal repo — completely offline.

---

### **2️⃣ Install Zabbix Server & Agent**
The second playbook:

- Installs Zabbix server + frontend packages on the server
- Installs Zabbix agent on client VMs
- Opens needed firewall ports
- Ensures services start on boot

At this stage, all binaries are installed but **not configured yet**.

---

### **3️⃣ Configure Zabbix Database, Frontend & Services**
The third playbook:

- Ensures MariaDB is running
- Creates the `zabbix` database and user
- Imports initial Zabbix schema
- Deploys `zabbix_server.conf`
- Configures PHP timezone
- Restarts all relevant services (MariaDB, Zabbix Server, Zabbix Agent, Apache)

At this point, Zabbix backend + frontend are fully operational.

---

## ▶️ How to Run the Automation

### Install Required Ansible Collections (IMPORTANT)

This project uses modules that are not included in ansible-core.
You must install the following collections on your Ansible control node before running the playbooks:
```
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.general
```
These collections provide:

ansible.posix.firewalld → opening firewall ports

community.general.sefcontext → managing SELinux context mappings

If these collections are missing, you may encounter errors like:

```vbnet
ERROR! couldn't resolve module/action 'firewalld'
ERROR! community.general.sefcontext was not found

```
Once installed, the playbooks and roles will use these modules normally.

### **. Edit your inventory**
`inventory.txt` must contain something like:

```
[zabbix-server]
192.168.1.10

[zabbix-agent]
192.168.1.11
```

### **. Run the full setup**
```bash
ansible-playbook -i inventory.txt site.yml -K
```

`-K` asks for sudo password.

---

## 🌐 Accessing the Zabbix Frontend

Once the playbooks finish:

### **If using NAT Network**  
Set port forwarding for your VM:
```
|     Host       |     Guest       |
|--------------------------------- |
| 127.0.0.1:8080 | 192.168.1.10:80 |
```
Then open:

```
http://localhost:8080/zabbix
```

You will see the Zabbix setup wizard.

---

## 📝 Author

Project by **Amr Qamhieh** — automated with Ansible to provide a complete offline Zabbix infrastructure setup.

