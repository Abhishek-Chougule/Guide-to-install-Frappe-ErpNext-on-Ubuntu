# ERPNext v16 Installation Guide on Ubuntu 24.04

## Maintained by Abhishek Chougule

Production-ready ERPNext v16 installation guide for Ubuntu 24.04 including:

- Frappe Framework v16
- ERPNext v16
- HRMS
- SSL with NGINX & Let's Encrypt
- Supervisor & Production Setup
- Optional Apps & Troubleshooting

---

## Quick Install Overview

### Update System

```bash
sudo apt update -y
sudo apt upgrade -y
```

### Create User

```bash
sudo adduser frappe
sudo usermod -aG sudo frappe
su frappe
```

### Install Required Packages

```bash
sudo apt install git curl redis-server mariadb-server mariadb-client \
python3-dev python3-pip python3-setuptools python3-venv \
xvfb libfontconfig libmysqlclient-dev pkg-config -y
```

### Install UV & Python 3.14

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv python install 3.14 --default
```

### Configure MariaDB

Edit:

```bash
sudo nano /etc/mysql/my.cnf
```

Add:

```ini
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Restart:

```bash
sudo service mysql restart
```



## Alternative MariaDB Configuration Path

Some Ubuntu installations use the following MariaDB configuration file instead:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Locate the following line:

```ini
collation-server = utf8mb4_general_ci
```

Modify it to:

```ini
collation-server = utf8mb4_unicode_ci
```

Save and exit:

- CTRL + X
- Press Y
- Press Enter


### Install Node.js 24

```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile

nvm install 24
nvm use 24
```

### Install Yarn

```bash
sudo npm install -g yarn
```

### Install Bench

```bash
uv tool install frappe-bench
```

### Initialize Bench

```bash
bench init --frappe-branch version-16 frappe-bench
cd frappe-bench
```

### Create Site

```bash
bench new-site site1.local
```

### Setup Production

```bash
sudo apt install ansible -y
sudo env "PATH=$PATH" bench setup production frappe
bench setup nginx
sudo supervisorctl restart all
```

### Install ERPNext

```bash
bench get-app --branch version-16 erpnext
bench --site site1.local install-app erpnext
```

---

# SSL Installation (NGINX)

## Enable DNS Multitenancy

```bash
bench config dns_multitenant on
```

## Install SSL Dependencies

```bash
sudo pip install -U pyOpenSSL cryptography
sudo pip install certbot
```

## Add Domain

```bash
bench setup add-domain www.mrabhi.com --site mrabhi.com
```

## Generate SSL

```bash
sudo -H bench setup lets-encrypt mrabhi.com --custom-domain www.mrabhi.com
```

---

# Firewall Fix

```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
sudo netfilter-persistent save
sudo systemctl restart nginx
```

---

# Optional Apps

## HRMS

```bash
bench get-app hrms --branch version-16
bench --site site1.local install-app hrms
```

## India Compliance

```bash
bench get-app --branch version-16 india_compliance https://github.com/resilient-tech/india-compliance
bench --site site1.local install-app india_compliance
```

## Builder

```bash
bench get-app builder
bench --site site1.local install-app builder
```

## Raven

```bash
bench get-app raven https://github.com/The-Commit-Company/Raven
bench --site site1.local install-app raven
```

---

# Useful Commands

```bash
bench restart
bench update
bench migrate
bench logs
```

---

# Credits

Maintained and formatted by Abhishek Chougule.

Reference:
https://discuss.frappe.io/t/erpnext-v16-installation-guide-on-ubuntu-24-04/162801
https://github.com/Abhishek-Chougule/Guide-to-install-Frappe-ErpNext-on-Ubuntu
