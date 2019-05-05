# Docker training HP - Workshop 07 -  Docker Swarm

## Swarm mode CLI commands

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
docker-machine ssh manager1 "docker service create -p 80:80 --name web nginx:latest"
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

* hyper-v [workaround](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-hyperv-setup.ps1) and [here](https://github.com/docker/labs/blob/master/swarm-mode/beginner-tutorial/swarm-node-hyperv-teardown.ps1)  .ps1