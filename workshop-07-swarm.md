# Docker training HP - Workshop 07 -  Docker Swarm

## Swarm mode CLI [commands](https://docs.docker.com/engine/swarm/)

### docker swarm [init](https://docs.docker.com/engine/reference/commandline/swarm_init/)

* Initialize a swarm

```sh
docker swarm init [OPTIONS]
```

### docker swarm [join](https://docs.docker.com/engine/reference/commandline/swarm_join/)

* Join a swarm as a node and/or manager

```sh
docker swarm join [OPTIONS] HOST:PORT
```

### docker service [create](https://docs.docker.com/engine/reference/commandline/service_create/)

* Create a new service

```sh
docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### docker service [inspect](https://docs.docker.com/engine/reference/commandline/service_inspect/)

* Display detailed information on one or more services

```sh
docker service inspect [OPTIONS] SERVICE [SERVICE...]
```

### docker service [ls](https://docs.docker.com/engine/reference/commandline/service_ls/)

* List services

```sh
docker service ls [OPTIONS]
```

### docker service [rm](https://docs.docker.com/engine/reference/commandline/service_rm/)

* Remove one or more services

```sh
docker service rm SERVICE [SERVICE...]
```

### docker service [scale](https://docs.docker.com/engine/reference/commandline/service_scale/)

* Scale one or multiple replicated services

```sh
docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]
```

### docker service [ps](https://docs.docker.com/engine/reference/commandline/service_ps/)

* List the tasks of one or more services

```sh
docker service ps [OPTIONS] SERVICE [SERVICE...]
```

### docker service [update](https://docs.docker.com/engine/reference/commandline/service_update/)

* Update a service

```sh
docker service update [OPTIONS] SERVICE
```

## Workshop 01 - Operate a docker swarm cluster

* Create a folder named swarm-operation. Add this file create-swar.sh

```sh
#!/bin/bash

# Swarm mode using Docker Machine

#This configures the number of workers and managers in the swarm
managers=3
workers=3

# This creates the manager machines
echo "======> Creating $managers manager machines ...";
for node in $(seq 1 $managers);
do
	echo "======> Creating manager$node machine ...";
	docker-machine create -d virtualbox manager$node;
done

# This create worker machines
echo "======> Creating $workers worker machines ...";
for node in $(seq 1 $workers);
do
	echo "======> Creating worker$node machine ...";
	docker-machine create -d virtualbox worker$node;
done
```

* Check config

´´´sh
# This lists all machines created
docker-machine ls
```

* Next you create a swarm by initializing it on the first manager. You do this by using docker-machine ssh to run docker swarm init

```sh
# initialize swarm mode and create a manager
echo "======> Initializing first swarm manager ..."
docker-machine ssh manager1 "docker swarm init --listen-addr $(docker-machine ip manager1) --advertise-addr $(docker-machine ip manager1)"
```

* Next you get join tokens for managers and workers. Create a file with this content:

```sh
#!/bin/bash

# get manager  tokens
export manager_token=`docker-machine ssh manager1 "docker swarm join-token manager -q"`

#This configures the number of managers in the swarm
managers=3

for node in $(seq 2 $managers);
do
	echo "======> manager$node joining swarm as manager ..."
	docker-machine ssh manager$node \
		"docker swarm join \
		--token $manager_token \
		--listen-addr $(docker-machine ip manager$node) \
		--advertise-addr $(docker-machine ip manager$node) \
		$(docker-machine ip manager1)"
done

# show members of swarm
docker-machine ssh manager1 "docker node ls"
```

* Add the worker machines and join them to the swarm. Create a file with this content:

```sh
#!/bin/bash

# get worker tokens
export worker_token=`docker-machine ssh manager1 "docker swarm join-token worker -q"`

#This configures the number of workers in the swarm
workers=3

# workers join swarm
for node in $(seq 1 $workers);
do
	echo "======> worker$node joining swarm as worker ..."
	docker-machine ssh worker$node \
	"docker swarm join \
	--token $worker_token \
	--listen-addr $(docker-machine ip worker$node) \
	--advertise-addr $(docker-machine ip worker$node) \
	$(docker-machine ip manager1):2377"
done

# show members of swarm
docker-machine ssh manager1 "docker node ls"
```

* The next step is to create a service and list out the services. This creates a single service called web that runs the latest nginx:

```sh
docker-machine ssh manager1 "docker service create \
-p 80:80 \
--name web \
nginx:latest"

docker-machine ssh manager1 "docker service create \
  --name portainer \
  --publish 9000:9000 \
  --constraint node.role==manager \
  --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  portainer/portainer"

docker-machine ssh manager1 "docker service ls"
```

* Check nginx service is running. Get the cluster ip by typying:

```sh
docker-machine ls
```

* Next let's inspect the service

```sh
docker-machine ssh manager1 "docker service inspect web"
```

* let's scale the service:

```sh
docker-machine ssh manager1 "docker service scale web=15"
```

* Docker has spread the 15 services evenly over all of the nodes

```sh
docker-machine ssh manager1 "docker service ps web"
```

* You can also drain a particular node, that is remove all services from that node. The services will automatically be rescheduled on other nodes.

```sh
docker-machine ssh manager1 "docker node update --availability drain worker1"
docker-machine ssh manager1 "docker node inspect worker1 --pretty"
docker-machine ssh manager1 "docker service ps web"
```

* You can check out the nodes and see that worker1 is still active but drained.

```sh
docker-machine ssh manager1 "docker node ls"
```

* You can also scale down the service

```sh
docker-machine ssh manager1 "docker service scale web=10"
docker-machine ssh manager1 "docker service ps web"
```

* Now bring worker1 back online and show it's new availability

```sh
docker-machine ssh manager1 "docker node update --availability active worker1"
docker-machine ssh manager1 "docker node inspect worker1 --pretty"
```

* Now let's take the manager1 node, the leader, out of the Swarm

```sh
# check current leader
docker-machine ssh manager1 "docker node ls"
docker-machine ssh manager1 "docker swarm leave --force"
```

* Check new leader

```sh
docker-machine ssh manager2 "docker node ls"
```

* Try to remove the service

```sh
docker-machine ssh manager2 "docker service rm web"
docker-machine ssh manager2 "docker service ps web"
```

* Clean up

```sh
#!/bin/bash
### Warning: This will remove all docker machines running ###
# Stop machines
docker-machine stop $(docker-machine ls -q)

# remove machines
docker-machine rm $(docker-machine ls -q)
```

* hyper-v [workaround](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-hyperv-setup.ps1) and [here](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-hyperv-teardown.ps1)  .ps1 [here-hyperv](https://docs.docker.com/get-started/part4/#localwin)


## Workshop 02 - Services and Stacks in Swarm

* Create docker-compose.yml (make sure your image is publish in dockerhub)

```yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: cmcornejocrespo/hp-course
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

* Run in in service mode

```sh
docker-compose up -d
```

* Check is working and stop it

* Run your new load-balanced app. Init the cluster

```sh
docker swarm init
```

* Deploy the stack

```sh
docker stack deploy -c docker-compose.yml hp-demo
```

* Check 5 replicas are running

```sh
docker service ls
```

or

```sh
docker stack services hp-demo
```

* A single container running in a service is called a task. Tasks are given unique IDs that numerically increment, up to the number of replicas you defined in docker-compose.yml. List the tasks for your service:

```sh
docker service ps hp-demo_web
```

* You can run curl -4 http://localhost:4000 several times in a row, or go to that URL in your browser and hit refresh a few times.

```sh
curl -4 http://localhost:4000
```

* To view all tasks of a stack, you can run docker stack ps followed by your app name, as shown in the following example:

```sh
docker stack ps hp-demo
```

* Scale the app. You can scale the app by changing the replicas value in docker-compose.yml, saving the change, and re-running the docker stack deploy command

```sh
docker stack deploy -c docker-compose.yml hp-demo
```

* Check new status

```sh
docker stack ps hp-demo
```

* Take down the app and the swarm

```sh
docker stack rm hp-demo
docker swarm leave --force
```

## Workshop 03 - Docker playground

* Open [play-with-docker](https://labs.play-with-docker.com)

* Add three masters

* Initialize the swarm and add nodes

```sh
docker swarm init --advertise-addr <IP>

docker swarm join --token TOKEN IP:PORT
```

* Check the swarm in the leader

```sh
docker node ls
```

* Add a new service and redeploy

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  portainer:
    image: portainer/portainer
    ports:
      - "9000:9000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

* Deploy stack

```sh
docker stack deploy -c docker-compose.yml hp-demo
```

* Check status

```sh
docker stack services hp-demo
```

* Persist the data. Update docker-compose.yml

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  portainer:
    image: portainer/portainer
    ports:
      - "9000:9000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:
```

* Scale the app

```sh
docker service scale hp-demo_web=15
```

* Delete the stack

```sh
docker stack rm hp-demo
```

## Workshop 03 - Services and Stacks in Swarm II

* Check this [link](https://github.com/dockersamples/example-voting-app). Update the stack and add portainer