## Use

Trying without the external MySQL database

**Clone git to system**
```bash
rm -fr ~/appdata/stacks/npm
mkdir -p ~/appdata/npm/
git clone https://github.com/FinchTechSoCal/docker-stack.git ~/appdata/stacks/npm
sed -i 's;/path/to/appdata/;'$HOME'/appdata/;g' ~/appdata/stacks/npm/.env
```




# IMPORTANT
This is wrong, do not use it. Needs to be updated.


# docker-npm
To do: Add DB

## Install Instructions
```bash
cd ~/docker-files/
git clone git@github.com:FinchTechSoCal/ft-docker-npm.git ~/docker-files/npm
mkdir ~/appdata/npm/
mkdir ~/appdata/npm-certs
ln -s /home/finchtech/docker-files/ft-docker-env/ft-cloud-1.prod.env .env
cd ft-docker-npm

docker-compose up -d
```