# Docker training - Workshop 05 -  Volumes

## Volumes

Docker volumes are [free-floating filesystems](https://docs.docker.com/engine/tutorials/dockervolumes/). They don't have to be connected to a particular container.

### Lifecycle

* [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/)
* [`docker volume rm`](https://docs.docker.com/engine/reference/commandline/volume_rm/)

### Info

* [`docker volume ls`](https://docs.docker.com/engine/reference/commandline/volume_ls/)
* [`docker volume inspect`](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

You can [map MacOS host directories as docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume):

## Demo

[demo](https://docs.docker.com/storage/volumes/)

```sh
docker volume create jenkins

docker run --name jenkins -d -p 8080:8080 -v jenkins:/var/jenkins_home jenkins/jenkins

docker stop jenkins

docker rm jenkins

docker run --name jenkins -d -p 8080:8080 -v jenkins:/var/jenkins_home jenkins/jenkins
```