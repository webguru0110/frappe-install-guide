# install_frape15
Install instructions to frappe/erpnext 15 on ubuntu 22.04

sudo timedatectl set-timezone "America/Guatemala"

sudo apt-get update -y

sudo apt-get upgrade -y

sudo adduser frappe

sudo usermod -aG sudo frappe

su frappe

cd /home/frappe/


sudo apt-get install git -y

sudo apt-get install python3-dev python3.10-dev python3-setuptools python3-pip python3-distutils -y

sudo apt-get install python3.10-venv -y

sudo apt-get install software-properties-common -y

sudo apt install mariadb-server mariadb-client -y

mariadb --version

sudo apt-get install redis-server -y

sudo apt-get install xvfb libfontconfig wkhtmltopdf -y

sudo apt-get install libmysqlclient-dev -y

sudo service mysql restart

sudo mysql_secure_installation


#
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# change line to utf8mb4_unicode_ci or fail next steps

#mariadb.conf.d

sudo nano /etc/mysql/my.cnf

[mysqld]

character-set-client-handshake = FALSE

character-set-server = utf8mb4

collation-server = utf8mb4_unicode_ci

[mysql]

default-character-set = utf8mb4

# close and save

sudo service mysql restart


sudo apt install curl -y

curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash

source ~/.profile

nvm install 16.15.0

sudo apt-get install npm -y

sudo npm install -g yarn -y


sudo pip3 install frappe-bench 

bench init --frappe-branch version-14 frappe-bench

npx browserslist@latest --update-db

npm install -g npm@9.1.1

cd frappe-bench/

chmod -R o+rx /home/frappe/

bench new-site misitio.gt

bench get-app --branch version-14 erpnext

bench get-app payments

bench get-app hrms

bench get-app ecommerce_integrations --branch main


bench --site misitio.gt install-app erpnext

bench --site misitio.gt install-app payments

bench --site misitio.gt install-app hrms

bench --site misitio.gt install-app ecommerce_integrations


bench --site  misitio.gt enable-scheduler

bench --site  misitio.gt set-maintenance-mode off


sudo bench setup production frappe

bench setup nginx

sudo supervisorctl restart all

sudo bench setup production frappe

# *** Optional POST install ***

# Change version pdf 

sudo apt-get remove wkhtmltopdf

wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb

sudo apt-get install -y xfonts-75dpi

sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb

wkhtmltopdf -V

# Add Firewall support

ufw allow 22,25,143,80,443,3306,3022,8000/tcp

ufw enable


# Add multi-site DNS option

bench config dns_multitenant on

bench new-site misitio2.gt

bench setup nginx

sudo service nginx reload

bench --site misitio2.gt install-app erpnext

bench --site misitio2.gt install-app payments

bench --site misitio2.gt install-app hrms

bench --site misitio2.gt install-app ecommerce_integrations
# Remember maybe need restart to init supervisorctl

# Add developer mode
bench --site misitio.gt set-config --global developer_mode 1

# Delete site
bench drop-site misitio.gt --no-backup

# Delete APP
bench --site misite.local uninstall-app hrms --no-backup

bench --site misite.local remove-from-installed-apps frappedesk

# Add support to FEL Guatemala  (In testing on V14)

bench get-app --branch production-v14 https://github.com/sihaysistema/factura_electronica_gt.git --resolve-deps

bench --site misitio.gt  install-app factura_electronica

bench setup requirements

bench update --patch

bench build --app factura_electronica

bench --site misitio.local migrate

bench restart && bench clear-cache

bench build --app factura_electronica --production

# Remember maybe need restart to init supervisorctl


# * * * Other operations V14 (notes) * * * 

# Active developer mode (bench) Desk -> User dropdown list -> Set Desktop Icons -> check "Developer"
bench set-config developer_mode 1

bench clear-cache

bench setup requirements --dev

#
bench set-maintenance-mode off




# Create a new app (bench)
bench new-app myapp_name

# Delete app from site
bench --site {site} uninstall-app {app} --force --no-backup

or 

bench --site misitio.local uninstall-app ecommerce_integrations --force

# Delete all app from site 
bench remove-app myapp_name

bench --site all uninstall-app tareas --force

# Connect app (local) to a remote repository (github) | Command in local app to connect like "/apps/myapp_name$"
git remote add origin https://github.com/usergit/myapp_name.git

# Connect and push to a "master" branch
git branch -M master

git push -u origin master

# Change to a "develop" or any other branch
git checkout -b develop

git push --set-upstream origin develop

# Show status git
git status

# Add files to committed git
git add .

# Commit and send changes to git branch develop
git commit -m "some files change descriptions"

git push origin develop

# Return git change from github branch to local 



# Fix error restart

chmod -R o+rx /home/frappe


sudo nano /etc/supervisor/supervisord.conf

(Add these lines under [unix_http_server])
--------
chmod=0760

chown=frappe:frappe

-----
# Restore database (copy uncompressed file .sql in frappe location.
bench --site misite.com --force restore 20231102_060002-misite-database.sql
---
sudo -A systemctl restart supervisor


 # Reset Site to default values
 bench --site site_name reinstall


 -----
 # Install Node Last version (prep to V15)
 
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

source ~/.bashrc

nvm list-remote

nvm install v20.9.0

nvm alias default v20.9.0

node -v

# Change branch to specific (V15)

bench get-app hrms --branch version-15

bench --site [sitename] install-app hrms

cd apps/hrms
git fetch upstream version-15:version-15
git checkout version-15
cd ../..
bench --site [sitename] migrate
or
bench update

----




