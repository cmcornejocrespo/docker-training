# Docker training HP - Workshop 04 -  Dockerfile

## Dockerfile

[The configuration file](https://docs.docker.com/engine/reference/builder/). Sets up a Docker container when you run `docker build` on it.

Here are some common text editors and their syntax highlighting modules you could use to create Dockerfiles:
* [Sublime Text 2](https://packagecontrol.io/packages/Dockerfile%20Syntax%20Highlighting)
* [Atom](https://atom.io/packages/language-docker)
* [Vim](https://github.com/ekalinin/Dockerfile.vim)
* [Emacs](https://github.com/spotify/dockerfile-mode)
* [TextMate](https://github.com/docker/docker/tree/master/contrib/syntax/textmate)
* [VS Code](https://github.com/Microsoft/vscode-docker)
* Also see [Docker meets the IDE](https://domeide.github.io/)

## Instructions

* [.dockerignore](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
* [FROM](https://docs.docker.com/engine/reference/builder/#from) Sets the Base Image for subsequent instructions.
* [LABEL](https://docs.docker.com/config/labels-custom-metadata/) apply key/value metadata to your images, containers, or daemons.
* [RUN](https://docs.docker.com/engine/reference/builder/#run) execute any commands in a new layer on top of the current image and commit the results.
* [CMD](https://docs.docker.com/engine/reference/builder/#cmd) provide defaults for an executing container.
* [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) informs Docker that the container listens on the specified network ports at runtime.  NOTE: does not actually make ports accessible.
* [ENV](https://docs.docker.com/engine/reference/builder/#env) sets environment variable.
* [COPY](https://docs.docker.com/engine/reference/builder/#copy) copies new files or directories to container.  By default this copies as root regardless of the USER/WORKDIR settings.  Use `--chown=<user>:<group>` to give ownership to another user/group.
* [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint) configures a container that will run as an executable.
* [VOLUME](https://docs.docker.com/engine/reference/builder/#volume) creates a mount point for externally mounted volumes or other containers.
* [USER](https://docs.docker.com/engine/reference/builder/#user) sets the user name for following RUN / CMD / ENTRYPOINT commands.
* [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir) sets the working directory.
* [ARG](https://docs.docker.com/engine/reference/builder/#arg) defines a build-time variable.

## demo 01

[example](https://docs.docker.com/engine/examples/)

## demo 02

* Create Dockerfile

```yml
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
````

* The app

It has two files:

- requirements.txt

```sh
Flask
Redis
```

- app.py

```yml
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

* Make sure all files are present

```sh
$ ls
Dockerfile		app.py			requirements.txt
```

* Let´s tag your image

```sh
docker build --tag=<change-it> .
````

* Check is properly tagged

```sh

$ docker image ls
````

* Run the app

```sh
docker run -p 4000:80 <change-it>
```

* Check the app is running (localhost:4000)

* Stop it (Ctrl+z)

* Let´s run it in dettached mode

```sh
docker run -d -p 4000:80 <change-it>
````

* Check the id

```sh
docker container ls
```

* Stop it with id

```sh
docker stop <id>
```

* Share your image

* Login

```sh
docker login
```

* Tag the image

```sh
docker tag <change-it> <your-repo>/<change-it>:0.1
```

* Check image is created

```sh
$ docker image ls
```

* Publish the image

```sh
docker push <your-repo>/<change-it>:0.1
```

* Pull and run the image from the remote repository (force to download it)

```sh
docker rmi <change-it>:0.1
docker rmi <your-repo>/<change-it>:0.1
docker run -p 4000:80 <your-repo>/<change-it>:0.1

```