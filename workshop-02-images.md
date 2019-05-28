# Docker training - Workshop 02 - Registry & Repository

A repository is a *hosted* collection of tagged images that together create the file system for a container.

A registry is a *host* -- a server that stores repositories and provides an HTTP API for managing the uploading and downloading of repositories.

Docker.com hosts its own [index](https://hub.docker.com/) to a central registry which contains a large number of repositories.  Having said that, the central docker registry [does not do a good job of verifying images](https://titanous.com/posts/docker-insecurity) and should be avoided if you're worried about security.

* [`docker login`](https://docs.docker.com/engine/reference/commandline/login) to login to a registry.
* [`docker logout`](https://docs.docker.com/engine/reference/commandline/logout) to logout from a registry.
* [`docker search`](https://docs.docker.com/engine/reference/commandline/search) searches registry for image.
* [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull) pulls an image from registry to local machine.
* [`docker push`](https://docs.docker.com/engine/reference/commandline/push) pushes an image to the registry from local machine.

# demo 

```sh
docker login

docker pull <your-image>

docker tag <your-image> <your-docker-name>/<your-image>

docker push <your-docker-name>/<your-image>

# check docker hub

```
