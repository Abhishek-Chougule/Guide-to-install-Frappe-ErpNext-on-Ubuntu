
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

# Table of Contents

1. Server Setup  
2. Install Required Packages  
3. Configure MariaDB  
4. Install Node.js, NPM & Yarn  
5. Initialize Frappe Bench  
6. Create Site & Install Frappe  
7. Setup Production Environment  
8. Install ERPNext & Apps  
9. SSL & Custom Domain Setup  
10. Optional Modules  
11. Useful Commands  

---

# 1. Server Setup

## Update System

```bash
sudo apt update -y
sudo apt upgrade -y
```

## Set Timezone

```bash
date
timedatectl set-timezone "Asia/Kolkata"
```

## Create User

```bash
sudo adduser frappe
sudo usermod -aG sudo frappe
su frappe
```

---

# 2. Install Required Packages

## Install Dependencies

```bash
sudo apt install git curl redis-server mariadb-server mariadb-client python3-dev python3-pip python3-setuptools python3-venv xvfb libfontconfig libmysqlclient-dev pkg-config software-properties-common net-tools nginx supervisor -y
```

## Install Python 3.14 using UV

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

uv python install 3.14 --default
```

Check Versions:

```bash
python3 --version
uv --version
```

---

## Install wkhtmltopdf

### ARM64

```bash
sudo wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb
```

### AMD64

```bash
sudo wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```

Install Package:

```bash
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo apt-get -f install -y
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```

Verify:

```bash
wkhtmltopdf --version
```

---

# 3. Configure MariaDB

## Secure MariaDB

```bash
sudo mysql_secure_installation
```

## Configure MariaDB UTF8

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

---

## Alternative MariaDB Config Path

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Find:

```ini
collation-server = utf8mb4_general_ci
```

Replace With:

```ini
collation-server = utf8mb4_unicode_ci
```

Save & Exit:

- CTRL + X
- Press Y
- Press Enter

Restart MariaDB:

```bash
sudo service mysql restart
```

---

# 4. Install Node.js, NPM & Yarn

## Install NVM & Node.js 24

```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile

nvm install 24
nvm use 24
```

Verify:

```bash
node --version
```

## Install NPM

```bash
sudo apt install npm -y
npm --version
```

## Install Yarn

```bash
sudo npm install -g yarn
yarn --version
```

---

# 5. Initialize Frappe Bench

## Install Bench

```bash
uv tool install frappe-bench
```

## Initialize Bench

```bash
bench init --frappe-branch version-16 frappe-bench
```

## Set Permissions

```bash
cd frappe-bench
sudo chmod -R o+rx /home/frappe/
```

---

# 6. Create Site & Install Frappe

## Create New Site

```bash
bench new-site site1.local
```

## Enable Scheduler

```bash
bench --site site1.local enable-scheduler
```

## Disable Maintenance Mode

```bash
bench --site site1.local set-maintenance-mode off
```

---

# 7. Setup Production Environment

## Install Ansible

```bash
sudo apt install ansible -y
```

## Setup Production

```bash
sudo env "PATH=$PATH" bench setup production frappe
```

## Setup NGINX

```bash
bench setup nginx
```

## Restart Services

```bash
sudo supervisorctl restart all
sudo supervisorctl status
```

If supervisor config missing:

```bash
sudo ln -sf /home/frappe/frappe-bench/config/supervisor.conf /etc/supervisor/conf.d/frappe-bench.conf

sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart all
```

---

# 8. Install ERPNext & Apps

## ERPNext

```bash
bench get-app --branch version-16 erpnext
bench --site site1.local install-app erpnext
```

## HRMS

```bash
bench get-app --branch version-16 hrms
bench --site site1.local install-app hrms
```

Verify:

```bash
bench version --format table
```

---

# 9. SSL & Custom Domain Setup

## Enable DNS Multitenancy

```bash
bench config dns_multitenant on
```

## Add Domain

```bash
bench setup add-domain www.mrabhi.com --site mrabhi.com
```

## Install SSL Dependencies

```bash
#dont use this if using uv
sudo pip install -U pyOpenSSL cryptography
sudo pip install certbot

#use
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot

#can do this for ssl certificate setup
sudo certbot --nginx

#but i recommend below
sudo -H bench setup lets-encrypt site_name
```

## Generate SSL

```bash
sudo -H bench setup lets-encrypt mrabhi.com --custom-domain www.mrabhi.com
```

## Firewall Fix

```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
sudo netfilter-persistent save
sudo systemctl restart nginx
```

---

# Optional Troubleshooting

## Fix PermissionError [Errno 13]

```bash
sudo addgroup supervisor
sudo usermod -aG supervisor frappe
```

Edit:

```bash
sudo nano /etc/supervisor/supervisord.conf
```

Update:

```ini
[unix_http_server]
file=/var/run/supervisor.sock
chmod=0760
chown=frappe:frappe
```

Restart:

```bash
sudo service supervisor restart
sudo supervisorctl stop all
sudo bench setup production frappe
```

---

## Fix CSS & JS Not Loading

```bash
sudo chmod o+x /home/$USER
```

## Fix Spawn Errors

```bash
bench setup socketio && bench setup supervisor && bench setup redis && sudo supervisorctl reload
```

---

# 10. Optional Modules

## India Compliance

```bash
bench get-app --branch version-16 india_compliance https://github.com/resilient-tech/india-compliance
bench --site site1.local install-app india_compliance
```

## Chat

```bash
bench get-app chat
bench --site site1.local install-app chat
```

## Builder

```bash
bench get-app builder
bench --site site1.local install-app builder
```

## Persona

```bash
bench get-app persona https://github.com/iptelephony/persona
bench --site site1.local install-app persona
```

## Raven

```bash
bench get-app raven https://github.com/The-Commit-Company/Raven
bench --site site1.local install-app raven
```

---

# 11. Useful Commands

```bash
bench restart
bench update
bench migrate
bench logs
bench doctor
```

---

# Credits

Maintained and formatted by Abhishek Chougule.

Reference Sources:

- https://discuss.frappe.io/t/erpnext-v16-installation-guide-on-ubuntu-24-04/162801
- https://github.com/Abhishek-Chougule/Guide-to-install-Frappe-ErpNext-on-Ubuntu
