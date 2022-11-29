[![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/ggogel/seafile-server?label=docker%20build%3A%20seafile-server%20)](https://hub.docker.com/r/ggogel/seafile-server)
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/ggogel/seafile-server?label=docker%20build%3A%20seafile-server%20)](https://hub.docker.com/r/ggogel/seafile-server)
[![Docker Image Version (tag latest semver)](https://img.shields.io/docker/v/ggogel/seafile-server/9.0.9)](https://hub.docker.com/r/ggogel/seafile-server)
[![Docker Pulls](https://img.shields.io/docker/pulls/ggogel/seafile-server)](https://hub.docker.com/r/ggogel/seafile-server)

[![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/ggogel/seahub?label=docker%20build%3A%20seahub)](https://hub.docker.com/r/ggogel/seahub)
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/ggogel/seahub?label=docker%20build%3A%20seahub)](https://hub.docker.com/r/ggogel/seahub)
[![Docker Image Version (tag latest semver)](https://img.shields.io/docker/v/ggogel/seahub/9.0.9)](https://hub.docker.com/r/ggogel/seahub)
[![Docker Pulls](https://img.shields.io/docker/pulls/ggogel/seahub)](https://hub.docker.com/r/ggogel/seahub)

[![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/ggogel/seahub-media?label=docker%20build%3A%20seahub-media)](https://hub.docker.com/r/ggogel/seahub-media)
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/ggogel/seahub-media?label=docker%20build%3A%20seahub-media)](https://hub.docker.com/r/ggogel/seahub-media)
[![Docker Image Version (tag latest semver)](https://img.shields.io/docker/v/ggogel/seahub/9.0.9)](https://hub.docker.com/r/ggogel/seahub-media)
[![Docker Pulls](https://img.shields.io/docker/pulls/ggogel/seahub-media)](https://hub.docker.com/r/ggogel/seahub-media)

[![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/ggogel/seafile-caddy?label=docker%20build%3A%20seafile-caddy)](https://hub.docker.com/r/ggogel/seafile-caddy)
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/ggogel/seafile-caddy?label=docker%20build%3A%20seafile-caddy)](https://hub.docker.com/r/ggogel/seafile-caddy)
[![Docker Image Version (tag latest semver)](https://img.shields.io/docker/v/ggogel/seafile-caddy/1.0.6)](https://hub.docker.com/r/ggogel/seafile-caddy)
[![Docker Pulls](https://img.shields.io/docker/pulls/ggogel/seafile-caddy)](https://hub.docker.com/r/ggogel/seafile-caddy)


# Containerized Seafile Deployment
A fully containerized deployment of Seafile for Docker and Docker Swarm.

## Table of Contents
1. [Features](#features)
1. [Structure](#structure)
1. [Getting Started](#getting-started)
1. [Additional Information](#additional-information)
    1. [Enable the New GoLang Fileserver coming with 9.0](#enable-the-new-golang-fileserver-coming-with-90)
    1. [Upgrading Seafile Server](#upgrading-seafile-server)
    1. [User Avatars and Custom Logos](#user-avatars-and-custom-logos)
    1. [LDAP](#ldap)
    1. [OAuth](#oauth)
    1. [Garbage Collection](#garbage-collection)
    1. [Access Log](#access-log)
    1. [Docker Swarm](#Docker-Swarm-1)
        1. [Storage](#storage)
        1. [Network](#network)
        1. [Reverse Proxy load balancing](#reverse-proxy-load-balancing)
        1. [Example](#example)

## Features
- Complete redesign of the [official Docker deployment](https://manual.seafile.com/docker/deploy%20seafile%20with%20docker/) with containerization best-practices in mind.
- Runs seahub (frontend) and seafile server (backend) in separate containers, which commuicate with each other over TCP.
- Cluster without pro edition.
- Completely removed Nginx and self-implemented Let's Encrypt and replaced it with two caddy services.
- Increased Security:
    - The caddy reverse proxy serves as a single entry point to the stack. Everything else runs in an isolated network.
    - Using [Alpine Linux](https://alpinelinux.org/about/) based images for the frontend, which is designed with security in mind and comes with proactive security features.
    - Official Seafile Docker deployment uses entirely outdated base images and dependencies. Here base images and dependencies are updated on a regular basis.
- Reworked Dockerfiles featuring multi-stage builds, allowing for smaller images and faster builds.
- Schedule offline garbage collection with cron job.
- Runs upgrade scripts automatically when a new image version is deployed.
- All features of Seafile Community Edition are included.

## Structure

Services:
- *seafile-server*
    - contains the backend component called [seafile-server](https://github.com/haiwen/seafile-server)
    - handles storage, some direct client access and seafdav
- *seahub*
    - dynamic frontend component called [seahub](https://github.com/haiwen/seahub)
    - serves the web-ui
    - communicates with seafile-server
- *seahub-media*
    - serves static website content as well as avatars and custom logos
- *db*
    - the database used by *seafile-server* and *seahub*
- *memcached*
    - database cache for *seahub*
- *seafile-caddy*
    - reverse proxy that forwards paths to the correct endpoints: *seafile-server*, *seahub* or *seahub-media*
    - is the single external entrypoint to the deployment

Volumes:

- *seafile-data*
    - shared data volume of *seafile-server* and *seahub*
    - also contains configuration and log files
- *seafile-mariadb*
    - volume of the *db* service
    - stores the database
- *seahub-custom*
    - contains custom logos
    - stored by *seahub* and served by *seahub-media*
- *seahub-avatars*
    - contains user avatars
    - stored by *seahub* and served by *seahub-media*

*Note: In the official docker deployment custom and avatars are served by nginx. Seahub alone is not able to serve them for some reason, hence the separate volumes.*

Networks:
- *seafile-net*
    - isolated local network used by the services to communicate with each other

## Getting Started

1. ***Prerequisites***

    Requires Docker and docker-compose to be installed.

2. ***Get the compose file***
    
    #### Docker Compose

    Use this compose file as a starting point.
    ```
    wget https://raw.githubusercontent.com/ggogel/seafile-containerized/master/compose/docker-compose.yml
    ```
    
    _Note:_ We expect certain services names. Do not rename services except `db`.
    
    #### Docker Swarm 

    If you run a single node swarm and don't want to run multiple replicas, you can use the same compose file. Otherwise refer to [Additional Information / Docker Swarm](#Docker-Swarm-1).
   

3. ***Set environment variables***

    **Important:** The environment variables are only relevant for the first deployment. Existing configuration in the volumes is **not** overwritten.

    On a first deployment you need to carefully set those values. Changing them later can be tricky. Refer to the Seafile documentation on how to change configuration values.
   
    ### *seafile-server*
    The name of the mariadb service, which is automatically the docker-internal hostname.
     ```
    - DB_HOST=db 
     ```
    Password of the mariadb root user. This must equal MYSQL_ROOT_PASSWORD.
     ```
    - DB_ROOT_PASSWD=db_dev
     ```
    Time zone used by Seafile.
     ```
    - TIME_ZONE=Europe/Berlin
     ```
   
    This will be used for the SERVICE_URL and FILE_SERVER_ROOT. 
    Important: Changing those values in the config files later won't have any effect because they are written to the database. Those values have priority over the config files. To change them enter the "System Admin" section on the web-ui. If you encounter issues with file upload, it's likely that those are configured incorrectly.
     ```
    - SEAFILE_SERVER_HOSTNAME=seafile.mydomain.com 
     ```
    If you plan to use a reverse proxy with https, set this to true. This will replace http with https in the SERVICE_URL and FILE_SERVER_ROOT.
     ```
    - HTTPS=false
     ```

    ### *seahub*
    Username / E-Mail of the first admin user.
     ```
    - SEAFILE_ADMIN_EMAIL=me@example.com
     ```
    Password of the first admin user.
     ```
    - SEAFILE_ADMIN_PASSWORD=asecret
     ```

    ### *db*
    Password of the mariadb root user. Must match DB_ROOT_PASSWD.
     ```
    - MYSQL_ROOT_PASSWORD=db_dev
     ```
    Enable logging console.
     ```
    - MYSQL_LOG_CONSOLE=true
    ```

4. ***(Optional) Migrating volumes from official Docker deployment or native install***

    **If you set up Seafile from scratch you can skip this part.**

    The [official Docker deployment](https://manual.seafile.com/docker/deploy%20seafile%20with%20docker/) uses [bind mounts](https://docs.docker.com/storage/bind-mounts/) to the host path instead of actual docker volumes. This was probably chosen to create compatibility between a native install and the docker deployment. This deployment uses [named volumes](https://docs.docker.com/storage/volumes/), which come with several advantages over bind mounts and are the recommended mechanism for persisted storage on Docker. The default path for named volumes on Docker is `/var/lib/docker/volumes/PROJECT-NAME_VOLUME-NAME/_data`.

   
    To migrate storage from the official Docker deployment run:
    ```
    mkdir -p /var/lib/docker/volumes/seafile_seafile-data/_data
    mkdir -p /var/lib/docker/volumes/seafile_seafile-mariadb/_data
    mkdir -p /var/lib/docker/volumes/seafile_seahub-custom/_data
    mkdir -p /var/lib/docker/volumes/seafile_seahub-avatars/_data

    cp -r /opt/seafile-data /var/lib/docker/volumes/seafile_seafile-data/_data
    cp -r /opt/seafile-mysql/db /var/lib/docker/volumes/seafile_seafile-mariadb/_data
    mv /var/lib/docker/volumes/seafile_seafile-data/_data/seafile/seahub-data/custom /var/lib/docker/volumes/seafile_seahub-custom/_data
    mv /var/lib/docker/volumes/seafile_seafile-data/_data/seafile/seahub-data/avatars /var/lib/docker/volumes/seafile_seahub-avatars/_data

    ```
    Of course you could also just use the old paths but I would strongly advise against that.

     *Tip:* If you want to use a different path, like a separate drive, to store your Docker volumes, simply create a symbolic link like this:
    ```
    service docker stop
    mv /var/lib/docker/volumes /var/lib/docker/volumes-bak
    mkdir -p /mnt/external/volumes
    ln -sf /mnt/external/volumes /var/lib/docker
    service docker start
    ```

5. ***(Optional) Reverse Proxy***
    
    Short version:
    The caddy reverse proxy integrated in the deployment exposes **port 80**. Point your reverse proxy to that port.
    
    Long version:
    This deployment does by design **not** include a reverse proxy that is capable of https and Let's Encrypt, because usually Docker users already have some docker-based reverse proxy solution deployed, which does exactly that. If you're using Docker for a while already, you probably know what to do and you can skip this section.

    If you are new to Docker or you are interested in another revers proxy solution, have a look at those options:
    - [jwilder/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) (recommended for beginners)
        - popular solution for beginners
        - doesn't support Docker Swarm
        - automatic Let's Encrypt with plugin
        - lacks a lot of advanced features
    - [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy ) (recommended for Docker Swarm and advanced users)
        - designed for Docker Swarm but can be used on regular Docker too
        - automatic Let's Encrypt integrated
        - very feature rich
        - first setup can be difficult
    - [traefik](https://doc.traefik.io/traefik/providers/docker/) 
        - very popular
        - supports Docker and Docker Swarm
        - automatic Let's Encrypt integrated
        - lacks a lot of advanced features
    

    Refer to the respective documentation. Often this is just one line you have to add to the `docker-compose.yml`. 

    Example for [jwilder/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy)
    ```
    seafile-caddy:
        image: ggogel/seafile-caddy:1.0.6
        expose:
        - "80"
        networks:
        - seafile-net
        - default
        environment:
        - VIRTUAL_PORT=80
        - VIRTUAL_HOST=seafile.mydomain.com
    ```

    Example for [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) on Docker Swarm
    ```
    seafile-caddy:
        image: ggogel/seafile-caddy:0.1
        networks:
        - seafile-net
        - caddy
        deploy:
        labels:
            caddy: seafile.mydomain.com #this will automatically enable https and Let's Encrypt
            caddy.reverse_proxy: "{{upstreams 80}}"
    ```
    Note that you don't need the ports definition any longer with this reverse proxy, because it connects to the service through its own network.

6. ***Deployment***
    
    #### Docker Compose
    After you followed the above steps and you have configured everything correctly run:
    ```
    docker-compose -p seafile up -d
    ```
    #### Docker Swarm
    After you followed the above steps and you have configured everything correctly run:
    ```
    docker stack deploy -c docker-compose.yml seafile
    ```
## Additional Information

### Enable the New GoLang Fileserver coming with 9.0

The Seafile team has reimplemented the previous Python fileserver component in GoLang. With 9.0 both variants are included in Seafile. By default the old Python fileserver is actived. The new one is supposed to provide better performance in high concurrency workloads.

To activate it add the following line to `seafile-data/seafile/conf/seafile.conf`:

```
[fileserver]
use_go_fileserver = true
```

Then restart or recreate the seafile-server container.

Note: After enabling it, you might need to clear your brwoser cache, in order for avatars to load.

### Upgrading Seafile Server
The *seafile-server* images contains scripts that will detect if a newer version of *seafile-server* is deployed and will automatically run the migration scripts included in the Seafile package. Upgrade from 7.1 is succesfully tested.

### User Avatars and Custom Logos

If you are unable to upload user avatars and custom logos, check your `SERVICE_URL` in the admin settings. It should not contain any port number. It should begin with `https` if you use a reverse proxy with https.
    
### LDAP

In order for LDAP to work, the *seafile-server* needs to be able to establish a connection to the LDAP server. Because the `seafile-net` is defined as `internal: true`, the service won't reach the LDAP server, as long as you don't deploy it in to the same stack and also connect it to `seafile-net`. As a workaround define another network in the `networks` top-level element. Like this:
```
networks:
  seafile-net:
    internal: true
  ext:
```
If not defined otherwise, the network will automatically have external access.
Then hook up *seafile-server* to this network:
```
seafile-server:
    ...
    networks:
    - seafile-net
    - ext
```

### OAuth

For OAuth the same network problem as with LDAP will occur, but here you will need to hook up the *seahub* service to the external network.

*Tip:* If you always want to use OAuth without clicking on the *Single Sign-On* button, you can rewrite the following paths in your reverse proxy:
```
caddy.rewrite: /accounts/login* /oauth/login/?
```

### Garbage Collection
Seafile has a block-based storage backend. This means that every file is associated to one or many blocks. If a file is permanently deleted from the trash bin, those blocks need to be cleared in order to free disk space. This process is called garbage collection. 

You can manually run the garbage collection with `docker exec`, where `seafile-server` is the name of the container running *seafile-server*:
```
docker exec -it seafile-server /scripts/gc.sh
```

You can schedule a cron job for garbage collection, by adding the following environment variable to *seafile-server*:
```
- GC_CRON=0 6 * * SUN
```
This would run the garbage collection every sunday at 6AM.

### Access Log
In order to make the access log of *seahub* visible through `docker logs` add the following line to the `gunicorn.conf.py`:
```
accesslog = '/proc/1/fd/1'
```


### Docker Swarm

If you want to deploy this stack on a Docker Swarm with multiple nodes or if you want to run replicas of the frontend (clustering), there are several things you have to consider first.

**Important:** You can only deploy multiple replicas of the frontend services *seahub* and *seahub-media*. Deploying replicas of the backend or the database would cause data inconsistency or even data corruption.

#### Storage
In order to make the same volumes available to services running on different nodes, you need an advanced storage solution. This could either be distributed storage like GlusterFS, Ceph or NFS. The volumes are then usually mounted through storage plugins. The repository [marcelo-ochoa/docker-volume-plugins](https://github.com/marcelo-ochoa/docker-volume-plugins) contains some good storage plugins for Docker Swarm.


#### Network
If you have services running on different nodes, which have to communicate to each other, you have to define their network as an overlay network. This will span the network across the whole Swarm.
```
seafile-net:
    driver: overlay
    internal: true
```

#### Reverse Proxy load balancing
If you want to run frontend replicas (clustering), you'll need to enable dnsrr endpoint mode, which is needed for proper load balancing.

To enable load balancing you have to configure the following options:

Set the endpoint mode for the frontend services *seahub* and *seahub-media* to dnsrr. This will enable *seafile-caddy* to see the IPs of all replicas, instead the default virtual IP (VIP) created by the Swarm routing mesh.
```
deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

```
Then you have to set the following environment variable for *seafile-caddy*, which will enable a periodic DNS resolution for the frontend services.
```
environment:
      - SWARM_DNS=true
```

The load balancer, in this case *seafile-caddy*, will then create so called sticky sessions, which means that a client connecting with a certain IP will be forwarded to the same service for the time being. Hashing is based on the header `X-Forwarded-For`. This is better than client ip based hashing, when you have another reverse proxy in front of *seafile-caddy*, which is highly recommended. With client ip based hashing *seafile-caddy* would just forward everything to the same container, as it only sees the IP of the reverse proxy. Instead the X-Forwarder-For header contains the actual client IP.

It is also recommended to use dnsrr mode on the *seafile-server*, when you run multiple replicas of *seahub*. This will enable *seafile-server* to see the actual IPs of the *seahub* replicas when they connect to it, instead of a single virtual IP for all of them. This will circumvent probable IP:PORT overlaps in the TCP connection between *seahub* and *seafile-server* if you run many *seahub* replicas.
```
deploy:
      endpoint_mode: dnsrr

```

#### Example
You can check out this example and use it as a starting point for your Docker Swarm deployment. It is using [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) as the external reverse proxy and the GlusterFS plugin from [marcelo-ochoa/docker-volume-plugins](https://github.com/marcelo-ochoa/docker-volume-plugins). This resembles my personal production setup.
```
    wget https://raw.githubusercontent.com/ggogel/seafile-containerized/master/compose/docker-compose-swarm.yml
```
