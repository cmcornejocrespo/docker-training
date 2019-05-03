# Docker training HP - Workshop 03 -  Networking

## Network drivers

* bridge: The default network driver. If you don’t specify a driver, this is the type of network you are creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate. See [bridge](https://docs.docker.com/network/bridge/) networks [demo]

* host: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly. See use the [host](https://docs.docker.com/network/host/) network.

* overlay: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers. See [overlay](https://docs.docker.com/network/overlay/) networks.

## Lifecycle

* [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/)
* [`docker network rm`](https://docs.docker.com/engine/reference/commandline/network_rm/)
* [`docker network prune`](https://docs.docker.com/engine/reference/commandline/network_prune/)

## Info

* [`docker network ls`](https://docs.docker.com/engine/reference/commandline/network_ls/)
* [`docker network inspect`](https://docs.docker.com/engine/reference/commandline/network_inspect/)

## Connection

* [`docker network connect`](https://docs.docker.com/engine/reference/commandline/network_connect/)
* [`docker network disconnect`](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

## Demo - network bridge

```sh
# Manage a user-defined bridge
docker network create my-net

docker network ls

# Connect a container to a user-defined bridge
docker run --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest

docker inspect my-nginx

docker network inspect my-net

#Disconnect a container from a user-defined bridge
docker network disconnect my-net my-nginx

docker network inspect my-net

docker stop my-ngnix

docker rm my-ngnix

docker run --name my-nginx -d --publish 8080:80 nginx:latest

docker network rm my-net
```

## Demo - network bridge jenkins sonarqube

```sh

# create network
docker network create stack-ci

# https://hub.docker.com/r/jenkinsci/blueocean/

docker run -d --name jenkins --network stack-ci -p 8080:8080 jenkins/jenkins


#https://docs.docker.com/samples/library/sonarqube/

docker run -d --name sonarqube --network stack-ci -p 9000:9000 sonarqube

# tip

sh "'${mvnHome}/bin/mvn' clean install sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -Psonar-coverage"

```

## Demo - network host

```sh
docker run --rm -d --network host --name my_nginx nginx

sudo netstat -tulpn | grep :80
```