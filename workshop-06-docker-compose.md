# Docker training HP - Workshop 06 -  Dcoker Compose

## Install Docker Compose

* On desktop systems like Docker Desktop for Mac and Windows, Docker Compose is included as part of those desktop installs.

```sh
docker-compose version
```

## Service configuration [reference](https://docs.docker.com/compose/compose-file/)

### build

* Configuration options that are applied at build time. build can be specified either as a string containing a path to the build context:

```yaml
version: "3.7"
services:
  webapp:
    build: ./dir
```

* Or, as an object with the path specified under context and optionally Dockerfile and args:

ARGS

```yaml
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

#### ARGS

Add build arguments, which are environment variables accessible only during the build process.

First, specify the arguments in your Dockerfile:

```sh
ARG buildno
ARG gitcommithash

RUN echo "Build number: $buildno"
RUN echo "Based on commit: $gitcommithash"
```

Then specify the arguments under the build key. You can pass a mapping or a list:

```yaml
build:
  context: .
  args:
    buildno: 1
    gitcommithash: cdc3b19
```

### depends_on

Express dependency between services, Service dependencies cause the following behaviors:

* docker-compose up starts services in dependency order. In the following example, db and redis are started before web.

* docker-compose up SERVICE automatically includes SERVICE’s dependencies. In the following example, docker-compose up web also creates and starts db and redis.

```yaml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

### environment

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

```yaml
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

### healthcheck

* Configure a check that’s run to determine whether or not containers for this service are “healthy”

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### logging

* Logging configuration for the service.

```yaml
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

### networks

* Networks to join, referencing entries under the top-level networks key.

```yaml
services:
  some-service:
    networks:
     - some-network
     - other-network
```

#### ALIASES

* Aliases (alternative hostnames) for this service on the network. Other containers on the same network can use either the service name or this alias to connect to one of the service’s containers.

```yaml
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

### ports

* Expose ports.

```yaml
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
````

* long syntax

```yaml
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

### restart

* no is the default restart policy, and it does not restart a container under any circumstance. When always is specified, the container always restarts. The on-failure policy restarts a container if the exit code indicates an on-failure error.

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

### volumes

* Mount host paths or named volumes, specified as sub-options to a service. You can mount a host path as part of a definition for a single service, and there is no need to define it in the top level volumes key.

* This example shows a named volume (mydata) being used by the web service, and a bind mount defined for a single service (first path under db service volumes). The db service also uses a named volume called dbdata (second path under db service volumes), but defines it using the old string format for mounting a named volume. Named volumes must be listed under the top-level volumes key, as shown.

```yaml
version: "3.7"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

### Network configuration reference

* The top-level networks key lets you specify networks to be created. It can be bridge (default), host or overlay defined by driver

```yaml
networks:
  stack-ci:
    driver: bridge
```

## Worshop 01 

### Step 1: Setup

* Create a directory for the project:

```sh
$ mkdir composetest
$ cd composetest
```

* Create a file called app.py in your project directory and paste this in:

```sh
import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

* Create another file called requirements.txt in your project directory and paste this in

```sh
flask
redis
```

### Step 2: Create a Dockerfile

```yaml
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

This tells Docker to:

* Build an image starting with the Python 3.4 image.
* Add the current directory . into the path /code in the image.
* Set the working directory to /code.
* Install the Python dependencies.
* Set the default command for the container to python app.py.

** Step 3: Define services in a Compose file

* Create a file called docker-compose.yml in your project directory and paste the following:

```yaml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

This Compose file defines two services, web and redis. The web service:

* Uses an image that’s built from the Dockerfile in the current directory.
* Forwards the exposed port 5000 on the container to port 5000 on the host machine. We use the default port for the Flask web server, 5000.

* Step 4: Build and run your app with Compose

```sh
$ docker-compose up
```

* Check http://localhost:5000 and try to refresh to increase the counter

* Stop the application, either by running docker-compose down from within your project directory in the second terminal, or by hitting CTRL+C in the original terminal where you started the app.

### Step 5: Edit the Compose file to add a bind mount

* Edit docker-compose.yml in your project directory to add a bind mount for the web service:

```yml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

* The new volumes key mounts the project directory (current directory) on the host to /code inside the container, allowing you to modify the code on the fly, without having to rebuild the image.

### Step 6: Re-build and run the app with Compose

```sh
$ docker-compose up
```

### Step 7: Update the application

* Because the application code is now mounted into the container using a volume, you can make changes to its code and see the changes instantly, without having to rebuild the image. Change the greeting in app.py and save it.

* Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.


## Worshop 02 Jenkins and sonarqube 

* Create a file called docker-compose.yml

```yml
version "3"

services:
```

* Add jenkins service

```yml
  jenkins:
    image: jenkins/jenkins
    ports:
        - "8080:8080"
    networks:
        - stack-ci
    volumes:
        - jenkins_home:/var/jenkins_home
```

* Replace image to use build context

```yml
    build:
      context: ./jenkins
```

* Create folder named jenkins.

* Create a Dockerfile with this content:

```sh
FROM jenkins/jenkins:lts

LABEL mantainer="Your name - youremail"
LABEL version="1.0"
LABEL description="HP Dockerfile example"

ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

COPY basic-security.groovy /usr/share/jenkins/ref/init.groovy.d/basic-security.groovy
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
```

* Create a file called basic-security.groovy and add this content:
```sh
#!groovy
import hudson.security.*
import jenkins.model.*

def instance = Jenkins.getInstance()
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
def users = hudsonRealm.getAllUsers()
users_s = users.collect { it.toString() }

// Create the admin user account if it doesn't already exist.
if ("admin" in users_s) {
    println "Admin user already exists - updating password"

    def user = hudson.model.User.get('admin');
    def password = hudson.security.HudsonPrivateSecurityRealm.Details.fromPlainPassword('admin')
    user.addProperty(password)
    user.save()
}
else {
    println "--> creating local admin user"

    hudsonRealm.createAccount('admin', 'admin')
    instance.setSecurityRealm(hudsonRealm)

    def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
    instance.setAuthorizationStrategy(strategy)
    instance.save()
}
```

* Create a file called plugins.txt and ad this content:

```sh
workflow-aggregator:latest
git:latest
```

* Add the sonarqube section to docker-compose.yml

```yaml
  sonarqube:
    image: sonarqube
    ports:
      - "9000:9000"
    networks:
      - stack-ci
    environment:
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
```

* Add the postgres section to docker-compose.yml

```yaml
  db:
    image: postgres
    ports:
      - "5432:5432"
    networks:
      - stack-ci
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
```

* Add network section

```yaml
networks:
  stack-ci:
    driver: bridge
```

* Add volume section

```yaml
volumes:
  jenkins_home:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  postgresql:
  postgresql_data:
```

* Build the lot

```sh
docker-compose up -d --build
```

* Get the pipeline from [here](https://gist.github.com/cmcornejocrespo/59a06bb2688bce7bf9a37d025c043531)

* Stop and destroy stack

```sh
docker-compose down --volumes 
```

## Worshop 03 Compose and Django

### Define the project components

*  set up and run a simple Django/PostgreSQL app.

* For this project, you need to create a Dockerfile, a Python dependencies file, and a docker-compose.yml file. (You can use either a .yml or .yaml extension for this file.)

* Create an empty project directory.

* Create a new file called Dockerfile in your project director

* Add the following content to the Dockerfile.

```yaml
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

* Create a requirements.txt in your project directory.

```sh
Django>=2.0,<3.0
psycopg2>=2.7,<3.0
```

* Create a file called docker-compose.yml in your project directory.

```yaml
version: '3'

services:
  db:
    image: postgres
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
```

### Create a Django project

* Change to the root of your project directory.

```sh
docker-compose run web django-admin startproject composeexample .
```

This instructs Compose to run django-admin startproject composeexample in a container, using the web service’s image and configuration. Because the web image doesn’t exist yet, Compose builds it from the current directory, as specified by the build: . line in docker-compose.yml.

Once the web service image is built, Compose runs it and executes the django-admin startproject command in the container. This command instructs Django to create a set of files and directories representing a Django project.

* After the docker-compose command completes, list the contents of your project.

```sh
$ ls -l
drwxr-xr-x 2 root   root   composeexample
-rw-rw-r-- 1 user   user   docker-compose.yml
-rw-rw-r-- 1 user   user   Dockerfile
-rwxr-xr-x 1 root   root   manage.py
-rw-rw-r-- 1 user   user   requirements.txt
```

### Connect the database

* In your project directory, edit the composeexample/settings.py

* Replace the DATABASES = ... with the following:

```sh
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

* Run the docker-compose up 

* Open http://localhost:8000

* List running containers.

```sh
docker ps
```

* Shut down services

```sh
docker-compose down
```

## Worshop 04 - Elastic stack

* Check this [link](https://github.com/cmcornejocrespo/elastic-full-stack-demo)