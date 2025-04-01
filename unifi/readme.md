# Unifi Network Controller
# [linuxserver/unifi-network-application](https://github.com/linuxserver/docker-unifi-network-application)
## Application Setup

After setup, the web UI is available at https://ip:8443. The application can be configured, or a backup restored, using the first run wizard.

**This container requires an external mongodb database instance.**

### Setting Up Your External Database

Starting with version 8.1 of Unifi Network Application, mongodb 3.6 through 7.0 are supported. Starting with version 9.0 of Unifi Network Application, mongodb 8.0 is also supported.

**Make sure you pin your database image version and do not use `latest`, as mongodb does not support automatic upgrades between major versions.**

**MongoDB >4.4 on X86_64 Hardware needs a CPU with AVX support. Some lower end Intel CPU models like Celeron and Pentium (before Tiger-Lake) more Details: [Advanced Vector Extensions - Wikipedia](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX) don't support AVX, but you can still use MongoDB 4.4.**

If you are using the [official mongodb container](https://hub.docker.com/_/mongo/), you can create your user using an `init-mongo.sh` file with the following contents (do not modify; copy/paste as is):

```sh
#!/bin/bash

if which mongosh > /dev/null 2>&1; then
  mongo_init_bin='mongosh'
else
  mongo_init_bin='mongo'
fi
"${mongo_init_bin}" <<EOF
use ${MONGO_AUTHSOURCE}
db.auth("${MONGO_INITDB_ROOT_USERNAME}", "${MONGO_INITDB_ROOT_PASSWORD}")
db.createUser({
  user: "${MONGO_USER}",
  pwd: "${MONGO_PASS}",
  roles: [
    { db: "${MONGO_DBNAME}", role: "dbOwner" },
    { db: "${MONGO_DBNAME}_stat", role: "dbOwner" }
  ]
})
EOF
```

Mount the sh file into your *mongodb* container, and make sure to set the env vars below with the same values you supplied to the Unifi container.

For example:
  ```yaml
    unifi-db:
      image: docker.io/mongo:<version tag>
      container_name: unifi-db
      environment:
        - MONGO_INITDB_ROOT_USERNAME=root
        - MONGO_INITDB_ROOT_PASSWORD=
        - MONGO_USER=unifi
        - MONGO_PASS=
        - MONGO_DBNAME=unifi
        - MONGO_AUTHSOURCE=admin
      volumes:
        - /path/to/data:/data/db
        - /path/to/init-mongo.sh:/docker-entrypoint-initdb.d/init-mongo.sh:ro
      restart: unless-stopped
  ```


*Note that the init script method will only work on first run. If you start the Mongodb container without an init script it will generate test data automatically and you will have to manually create your databases, or restart with a clean `/data/db` volume and an init script mounted.*

You can also run the commands directly against the database using either `mongo` (< 6.0) or `mongosh` (>= 6.0).

### Device Adoption

For Unifi to adopt other devices, e.g. an Access Point, it is required to change the inform IP address. Because Unifi runs inside Docker by default it uses an IP address not accessible by other devices. To change this go to Settings > System > Advanced and set the Inform Host to a hostname or IP address accessible by your devices. Additionally the checkbox "Override" has to be checked, so that devices can connect to the controller during adoption (devices use the inform-endpoint during adoption).

Note that you must use 8080:8080. If you MUST change the port, it must be changed on both sides and manually changed in your system.properties file. Otherwise, devices will initially communicate and then break after.

**Please note, Unifi change the location of this option every few releases so if it's not where it says, search for "Inform" or "Inform Host" in the settings.**

In order to manually adopt a device take these steps:

```
ssh ubnt@$AP-IP
set-inform http://$address:8080/inform
```

The default device password is `ubnt`. `$address` is the IP address of the host you are running this container on and `$AP-IP` is the Access Point IP address.

When using a Security Gateway (router) it could be that network connected devices are unable to obtain an ip address. This can be fixed by setting "DHCP Gateway IP", under Settings > Networks > network_name, to a correct (and accessible) ip address.

### Migration From [Unifi-Controller](https://github.com/linuxserver/docker-unifi-controller)

If you were using the `mongoless` tag for the Unifi Controller container, you can switch directly to the Unifi Network Application container without needing to perform any migration steps.

**You cannot perform an in-place upgrade from an existing Unifi-Controller container, you must run a backup and then a restore.**

The simplest migration approach is to take a full backup of your existing install, including history, from the Unifi-Controller web UI, then shut down the old container.

You can then start up the new container with a clean `/config` mount (and a database container configured), and perform a restore using the setup wizard.

### Strict reverse proxies

This image uses a self-signed certificate by default. This naturally means the scheme is `https`.
If you are using a reverse proxy which validates certificates, you need to [disable this check for the container](https://docs.linuxserver.io/faq#strict-proxy).

## Usage

To help you get started creating a container from this image you can either use docker-compose or the docker cli.

>[!NOTE]
>Unless a parameter is flaged as 'optional', it is *mandatory* and a value must be provided.

### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
services:
  unifi-network-application:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi-network-application
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - MONGO_USER=unifi
      - MONGO_PASS=
      - MONGO_HOST=unifi-db
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi
      - MONGO_AUTHSOURCE=admin
      - MEM_LIMIT=1024 #optional
      - MEM_STARTUP=1024 #optional
      - MONGO_TLS= #optional
    volumes:
      - /path/to/unifi-network-application/data:/config
    ports:
      - 8443:8443
      - 3478:3478/udp
      - 10001:10001/udp
      - 8080:8080
      - 1900:1900/udp #optional
      - 8843:8843 #optional
      - 8880:8880 #optional
      - 6789:6789 #optional
      - 5514:5514/udp #optional
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=unifi-network-application \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e MONGO_USER=unifi \
  -e MONGO_PASS= \
  -e MONGO_HOST=unifi-db \
  -e MONGO_PORT=27017 \
  -e MONGO_DBNAME=unifi \
  -e MONGO_AUTHSOURCE=admin \
  -e MEM_LIMIT=1024 `#optional` \
  -e MEM_STARTUP=1024 `#optional` \
  -e MONGO_TLS= `#optional` \
  -p 8443:8443 \
  -p 3478:3478/udp \
  -p 10001:10001/udp \
  -p 8080:8080 \
  -p 1900:1900/udp `#optional` \
  -p 8843:8843 `#optional` \
  -p 8880:8880 `#optional` \
  -p 6789:6789 `#optional` \
  -p 5514:5514/udp `#optional` \
  -v /path/to/unifi-network-application/data:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/unifi-network-application:latest
```

## Parameters

Containers are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 8443:8443` | Unifi web admin port |
| `-p 3478:3478/udp` | Unifi STUN port |
| `-p 10001:10001/udp` | Required for AP discovery |
| `-p 8080:8080` | Required for device communication |
| `-p 1900/udp` | Required for `Make controller discoverable on L2 network` option |
| `-p 8843` | Unifi guest portal HTTPS redirect port |
| `-p 8880` | Unifi guest portal HTTP redirect port |
| `-p 6789` | For mobile throughput test |
| `-p 5514/udp` | Remote syslog port |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Etc/UTC` | specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `-e MONGO_USER=unifi` | Mongodb Username. Only evaluated on first run. **Special characters must be [url encoded](https://en.wikipedia.org/wiki/Percent-encoding)**. |
| `-e MONGO_PASS=` | Mongodb Password. Only evaluated on first run. **Special characters must be [url encoded](https://en.wikipedia.org/wiki/Percent-encoding)**. |
| `-e MONGO_HOST=unifi-db` | Mongodb Hostname. Only evaluated on first run. |
| `-e MONGO_PORT=27017` | Mongodb Port. Only evaluated on first run. |
| `-e MONGO_DBNAME=unifi` | Mongodb Database Name (stats DB is automatically suffixed with `_stat`). Only evaluated on first run. |
| `-e MONGO_AUTHSOURCE=admin` | Mongodb [authSource](https://www.mongodb.com/docs/manual/reference/connection-string/#mongodb-urioption-urioption.authSource). For Atlas set to `admin`. Only evaluated on first run. |
| `-e MEM_LIMIT=1024` | Optionally change the Java memory limit (in Megabytes). Set to `default` to reset to default |
| `-e MEM_STARTUP=1024` | Optionally change the Java initial/minimum memory (in Megabytes). Set to `default` to reset to default |
| `-e MONGO_TLS=` | Mongodb enable [TLS](https://www.mongodb.com/docs/manual/reference/connection-string/#mongodb-urioption-urioption.tls). Only evaluated on first run. |
| `-v /config` | Persistent config files |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__MYVAR=/run/secrets/mysecretvariable
```

Will set the environment variable `MYVAR` based on the contents of the `/run/secrets/mysecretvariable` file.

## Umask for running applications

For all of our images we provide the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags), permissions issues can arise between the host OS and the container, we avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id your_user` as below:

```bash
id your_user
```

Example output:

```text
uid=1000(your_user) gid=1000(your_user) groups=1000(your_user)
```

## Docker Mods

[![Docker Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=unifi-network-application&query=%24.mods%5B%27unifi-network-application%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=unifi-network-application "view available mods for this container.") [![Docker Universal Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=universal&query=%24.mods%5B%27universal%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=universal "view available universal mods.")

We publish various [Docker Mods](https://github.com/linuxserver/docker-mods) to enable additional functionality within the containers. The list of Mods available for this image (if any) as well as universal mods that can be applied to any one of our images can be accessed via the dynamic badges above.

## Support Info

* Shell access whilst the container is running:

    ```bash
    docker exec -it unifi-network-application /bin/bash
    ```

* To monitor the logs of the container in realtime:

    ```bash
    docker logs -f unifi-network-application
    ```

* Container version number:

    ```bash
    docker inspect -f '{{ index .Config.Labels "build_version" }}' unifi-network-application
    ```

* Image version number:

    ```bash
    docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/unifi-network-application:latest
    ```

## Updating Info

Most of our images are static, versioned, and require an image update and container recreation to update the app inside. With some exceptions (noted in the relevant readme.md), we do not recommend or support updating apps inside the container. Please consult the [Application Setup](#application-setup) section above to see if it is recommended for the image.

Below are the instructions for updating containers:

### Via Docker Compose

* Update images:
    * All images:

        ```bash
        docker-compose pull
        ```

    * Single image:

        ```bash
        docker-compose pull unifi-network-application
        ```

* Update containers:
    * All containers:

        ```bash
        docker-compose up -d
        ```

    * Single container:

        ```bash
        docker-compose up -d unifi-network-application
        ```

* You can also remove the old dangling images:

    ```bash
    docker image prune
    ```

### Via Docker Run

* Update the image:

    ```bash
    docker pull lscr.io/linuxserver/unifi-network-application:latest
    ```

* Stop the running container:

    ```bash
    docker stop unifi-network-application
    ```

* Delete the container:

    ```bash
    docker rm unifi-network-application
    ```

* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images:

    ```bash
    docker image prune
    ```

### Image Update Notifications - Diun (Docker Image Update Notifier)
