**Note:**: The following documentation is for frappe installation in windows using wsl. And I've used Karani's documentation for the frappe installation.

# Guide To Frappe Verison 15 Installation In Windows WSL

## WSL Setup

1. To Install WSL open Powershell as administrator
2. Run the following command in the powershell:
```bash
   wsl --install
```
3. After the installation is completed if it logs you to enter the linux username and password then enter it otherwise restart your pc once you have rebooted it will ask you for linux username and password.
4. **(Not Mandatory)** Download Windows Terminal from Microsoft Store and In the Windows Terminal settings in the **Startup** select default profile to Ubuntu with the penguine logo.

## Frappe Installation

### Update and Upgrade Packages
```bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
```

### Give Bench user permissions
```bash
    sudo usermod -aG sudo [frappe-user]
    sudo su [frappe-user]
    cd /home/[frappe-user]
```

### Install Required Packages

#### Install Git
```bash
    sudo apt-get install git -y
```

#### Install Python Dependencies
```bash
    sudo apt-get install python3-dev -y
    sudo apt-get install python3-setuptools python3-pip -y
    sudo apt install python3.12-venv -y
```

#### Install Maria DB
```bash
    sudo apt-get install software-properties-common -y
    sudo apt install mariadb-server -y
    sudo mysql_secure_installation
```
- When you run this command, the server will show the following prompts. Please follow the steps as shown below to complete the setup correctly.
- Enter current password for root: (Enter your SSH root user password)
- Switch to unix_socket authentication [Y/n]: Y
- Change the root password? [Y/n]: Y (It will ask you to set new MySQL root password at this step. This can be different from the SSH root user password.)
- Remove anonymous users? [Y/n] Y
- Disallow root login remotely? [Y/n]: N (This is set as N because we might want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.)
- Remove test database and access to it? [Y/n]: Y
- Reload privilege tables now? [Y/n]: Y

**Edit MYSQL default config file**
```bash
    sudo nano /etc/mysql/my.cnf
```

**Add the following block of code to the end of the file, exactly as is:**
```bash
    [mysqld] character-set-client-handshake = FALSE character-set-server = utf8mb4 collation-server = utf8mb4_unicode_ci [mysql] default-character-set = utf8mb4
```

**Restart the MYSQL Server**
```bash
    sudo service mysql restart
```

**Install Redis Server**
```bash
    sudo apt-get install redis-server -y
```

#### Instal CURL, Node, NPM and Yarn
```bash
    sudo apt install curl
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash source ~/.profile nvm install 18
    sudo apt-get install npm -y
    sudo npm install -g yarn -y
```

#### Install wkhtmltopdf
```bash
    sudo apt-get install xvfb libfontconfig wkhtmltopdf -y
```

#### Install Frappe Bench & Ansible
```bash
    sudo -H pip3 install frappe-bench --break-system-packages
    sudo -H pip3 install ansible --break-system-packages
```

### Intialize Frappe Bench
```bash
    bench init frappe-bench --frappe-branch version-15
    cd frappe-bench
```

### Change user directory permissions
```bash
    chmod -R o+rx /home/[frappe-user]
```

### Create a New Site
```bash
    bench new-site [site-name]
```

### Setting ERPNext for Production
**Select (Y) To all the options**
```bash
    bench --site [site-name] enable-scheduler
    bench --site [site-name] set-maintenance-mode off
    sudo bench setup production [frappe-user]
    bench setup nginx
    sudo supervisorctl restart all sudo bench setup production [frappe-user]
```

### Install ERPNext and other Apps
``` bash
    bench get-app payments
    bench get-app --branch version-15 erpnext
    bench get-app hrms
    bench --site [site-name] install-app erpnext
    bench --site [site-name] install-app hrms
    bench migrate
    sudo supervisorctl restart all
```
