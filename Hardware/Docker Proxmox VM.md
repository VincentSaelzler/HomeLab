# Docker | Proxmox VM
## Specifications
- 2 CPU Cores
- 2 GiB RAM
- 1x 1 Gbit Network Card
- 1x 20 GB Virtual Hard Drive
## Configuration
- OS: Arch Linux (ISO from 2019-05-02)
- Host Name: `docker0.vnet`
- Admin User: `root`
- Unprivileged* user: `vince`

*Due to the nature of Docker, `vince` ends up with root equivalent privileges anyways. Keeping to best practice (of not doing everything as root) nonetheless. 

## Docker
### Utility Files
`docker-wipe.sh` wipes everything on the host. It's especially helpful when troubleshooting setups where containers are inter-dependent on each other.

```shell
$ cat docker-wipe.sh
#!/bin/bash

echo "--STOPPING CONTAINERS--"
docker stop $(docker ps -aq)

echo "--DELETING CONTAINERS, VOLUMES, AND NETWORKS--"
docker rm $(docker ps -aq)
docker volume prune
docker network prune

echo "--DISPLAYING CONTAINERS, VOLUMES, AND NETWORKS (SHOULD BE BLANK)--"
echo
docker ps -a
echo
docker volume ls
echo
docker network ls
```
`docker-create-web-net.sh` adds a shared network. This is used in the configuration of other containers. It's probably overkill having a shell script for this, but it helps me remember what the network should be called!

```shell
$ cat docker-create-web-net.sh
#!/bin/bash

docker network create websites
```
### Reverse Proxy SSL Containers
**This setup has dependencies in terms of file structure on the host!**

The docker-compose files assume that a template nginx configuration file is located at `/home/vince/rvrs-prxy-ssl/conf/nginx.tmpl`

Create directories and download the template.
```
$ mkdir ~/rvrs-prxy-ssl
$ mkdir ~/rvrs-prxy-ssl/conf
$ curl https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl > ~/rvrs-prxy-ssl/conf/nginx.tmpl
```

The `docker-compose.yml` file.
```yml
$ cat ~/rvrs-prxy-ssl/docker-compose.yml
version: '2'

services:
  nginx-proxy:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - certs:/etc/nginx/certs:ro

  docker-gen:
    image: jwilder/docker-gen
    container_name: nginx-proxy-gen
    command: -notify-sighup nginx-proxy -watch /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
    volumes_from:
      - nginx-proxy
    volumes:
      - /home/vince/rvrs-prxy-ssl/conf/nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.docker_gen"

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-proxy-le
    volumes_from:
      - nginx-proxy
    volumes:
      - certs:/etc/nginx/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  conf:
  vhost:
  html:
  certs:

networks:
  default:
    external:
      name: websites
```
### Wedding Website Containers
It's just a "plain vanilla" WordPress container with host names specified for my wedding website.
```yml
$ cat bv/docker-compose.yml
version: '2'

services:

  wp-bv:
    image: wordpress
    environment:
      WORDPRESS_DB_PASSWORD: MY-DB-PW
      VIRTUAL_HOST: bethanyandvincent.com,www.bethanyandvincent.com
      LETSENCRYPT_HOST: bethanyandvincent.com,www.bethanyandvincent.com
      LETSENCRYPT_EMAIL: my.email@gmail.com
    networks:
      - websites
      - backend

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: MY-DB-PW
    networks:
      - backend

networks:
  backend:
  websites:
    external:
      name: websites
```

### Collatz Conjecture ASP.NET Core Site

This one has no saved data at all, so it makes an ideal Docker container!
```yml
version: '2'

services:

  asp-net-core:
    image: vincentsaelzler/collatzcorerazorpage
    environment:
      VIRTUAL_HOST: collatz.saelzler.org
      LETSENCRYPT_HOST: collatz.saelzler.org
      LETSENCRYPT_EMAIL: my.email@gmail.com
    networks:
      - websites

networks:
  websites:
    external:
      name: websites
```

