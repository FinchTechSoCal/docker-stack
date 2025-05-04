
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