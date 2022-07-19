

## Basic Docker CML Operation

### Check docker information

```bash
"""check for the client and server version details"""
docker version

"""check for config"""
docker info

"""Login"""
docker login

"""Logout"""
docker logout
```



### Docker Container

Key concepts: immutable, ephemeral
Can't store persistent data

```bash
"""Run container"""
# parameters:
-p / --publish: open port1 on the host IP, route traffic to container IP with port2
-d / --detach: run container on background
--name: specify a name to container
--help: visit more cml options
-e / --env: pass env variables
-it: interactively (pseudo-tty + keep session open)
--rm: delete when quit
docker container run -p 8080:8080 --name container_name -d image_name
docker container run -it --name container_name image_name bash[COMMEND RUN IN CONTAINER]

"""Start container """
docker container start -ai container_name


"""Execute command in existing container"""
# parameters:
-it: interactively
docker container exec -it container_name/ID bash[COMMEND]


"""Stop container (not remove)"""
# stop foreground
ctrl-c 
# stop background (id with first 3 digit enough)
docker container stop container_ids/names


"""Remove container"""
# parameters:
-f / --force: remove running containers
docker container rm container_ids/names


"""List container"""
# list running container
docker container ls
# list all container
docker container ls -a


"""List Process"""
# list process of container
docker container top container_id/name
# list process of whole system
ps aux
ps aux | grep key_word
# list config detail of container
docker container inspect container_id/name
# list the performance of all containers
docker container stats


"""Logs info"""
docker container logs container_name


"""List container ports"""
--format: formatting the output
docker container port container_name/ID
docker container inspect --format '{{ .NetworkSettings.IPAddress}}' container_id/name

"""Clean docker containers"""
docker container prune

"""Build multiplatform image"""
docker buildx 
```



### Docker Networks

#### Network Driver

Build-in or 3rd party extensions that give you virtual network features.

* --driver bridge: Default docker virtual network which is NAT'ed behind the Host IP, not DNS server.
* --driver host: It gain performance by skipping virtual networks but cacrifices security of container model
* --driver none: removes eth0 and only leaves you with localhost interface in container.
* --driver overlay: for container to container traffic inside a single Swarm

#### Docker DNS

Docker container IP address changes all the time and we need build-in DNS server to help us for inter-communication.

```bash
"""Create network"""
# parameters
--network network_name: create a type of network
--drive virtual_net_type: default bridge as VN driver
docker network create network_name
docker container run --name container_name --network network_name -d image_name


"""List all networks"""
docker network ls


"""List all container using a network"""
docker network inspect network_name


"""Attach a network"""
# create a NIC in a container on an existing virtual network
docker network connect network_ID container_ID/name


"""Detach a network"""
# remove a NIC in a container on an existing virtual network
docker network disconnect network_ID container_ID/name


"""List host IP"""
ifconfig en0


```



### Docker Images

Images are made up of file system changes and metadata
Image_name = <usr>/<repo>:<tag>

#### Docker image has / don't have

* App binaries and dependencies
* Metadata about the image data and how to run the image
* Not a complete OS, No kernel and kernel modules

#### Image Layer

* Each layer is uniquely identified and only stored once on a host (reuse for different images for same layer)
* This saves storage space on the host and transfer time on push/pull
* A container is just a single read/write layer on the top of image

#### Image Tag

Common tag for image: latest(new)/stable/alpine(samll)

```bash
"""Download images"""
docker pull image_name

"""List all images"""
docker image ls

"""List history for layers in image"""
docker image history image_name

"""Details of image """
docker image inspect image_name

"""Assign a tag to image"""
docker image tag source_image_name target_image_name

"""Push the change of image to DockerHub"""
docker image push
docker image push image_name

"""Build image from dockerfile"""
-f/--file: name of dockerfile
-t/--tag: name of new image_name(<repo><image>:<tag>)
docker image build -t image_name dir_to_create(.)

"""Clean docker image"""
docker image prune
docker system prune


```



### Dockerfile (create docker image)

#### Docker file example

Create a SHA hash value for each layer and next time it will check if it need to modify (rebuild). If docker can find it in cache, next time will build in much faster speed.

```dockerfile
# 1. start from a Linux distribution system, like debain, ubuntu...
# these system includes package manager, like apt and yum
FROM distri_sys_name

# 2. set environment variables
ENV NGINX_VERSION 1.11.10-1~jessie

# 3. the cml code will run at shell inside container at build time, like add package repo and depandencies
# use && to make sure each cml as each layer in image
RUN apt-key adv ...... \
		&& echo '....' \ 
		&& apt-get update \ 
		&& apt-get install .... \
		.....
		
# 4. change working directory, like cd
WORKDIR /usr/share/nginx/html

# 5. copy file to container
COPY index.html index.html
		
# 6. expose these ports on the docker virtual network
EXPOSE 80 443

# 7. the command will run when container is launched
CMD ['nginx', '-g', 'daemon off;']

```



### Docker Volumes

Two ways to store persistent data outside the container:

1. Docker Volumes: 
   make special location outside of container UFS and need to be delete separately.

```dockerfile
# define volumn in dockerfile
VOLUME /var/lib/mysql
```

```bash
"""Define volume in container run"""
# parameter
-v: define docker volumn inside container

docker container run --name container_name -e ENV_VAR=True -v volume_name:volumn_path -d/--rm/--it image_name

docker container run --name mysql_vol -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mariadb

"""Create a volume"""
docker volumn create

"""List all docker volumes"""
docker volume ls

"""Find detail of volume"""
docker volume inspect volume_name


```

2. Bind Mounts: 
   Maps a host file or directory to a container file or directory
   Skip UFS and host files overwrite any in container
   Only use in container run, not dockerfile

```bash
"""Bind Mounts (using $(pwd) to refer working dir in host)"""
# map host working dir to container path
docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx


```



### Docker Update

Manage the source like cpu, gpu and RAM can be use to container.

```bash
# update container
-c: resource of cpu
-m: resource of memory
docker update -c ... -m ... container_name/ID
```



### Docker Registry

The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. You should use the Registry if you want to:

- tightly control where your images are being stored
- fully own your images distribution pipeline
- integrate image storage and distribution tightly into your in-house development workflow

```bash
# registry workflow
# 1. start registry image
docker container run -d -p 5000:5000 --name registry registry
# 2. retag an existing image and push it to new registry
docker tag exist_image_name localhost:5000/new_image_name
docker push localhost:5000/new_image_name
# 3. remove image from local cache and pull it
docker image remove exist_image_name
docker image remove localhost:5000/new_image_name
docker pull localhost:5000/new_image_name
# 4. stop registry container
docker container stop registry && docker container rm -v registry
```



### Docker Security

[Instruction for docker security](https://github.com/BretFisher/ama/issues/17)

1. Use Docker Control Group and Namespaces
2. Host Config Scanner / Image Scanner (Docker Bench)
3. Using USER in Dockerfile instead of running as ROOT
4. Docker rootless mode



### Docker Context

We can create a new docker context to change some detail of the docker, like tcp/ip, ssh and host environment variable.

```bash
"""Create a docker context"""
docker context create context_name
docker context create context_name --docker 'host=ssh:...'

"""List all contexts"""
docker context ls

"""Use context"""
docker context use context_name
```



## Docker Compose

### YAML-formatted file

* containers
* networks
* volumes

```yaml
services:  # containers. same as docker run

  container_name1: # a name for container and it is also DNS name of network
  	image: # Optional, if you use build, it become new_image_name
  	build: # Optional if want to build image from local dockerfile
      context: dockerfile_dir # common(.)
      dockerfile: dockerfile_name
    ports: # Optional for expose ports
    	- port1:port2
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    	- ENV_VAR1=var1
      - ENV_VAR2=var2
    volumes: # Optional, same as -v in docker run
    	- vol_name1:path1
    	- vol_name2:path2
    depends_on:
    	- depended_container1
    	- depended_container2
    	
  container_name2:

volumes: # Optional, same as docker volume create
	vol_name1:
	vol_name2:

networks: # Optional, same as docker network create
```



### Docker Compose CLI

```bash
"""Setup volumes/networks and start all containers"""
docker-compose build # if need customied image
docker-compose up
docker-compose up -d

"""Stop all containers and remove containers/volumes/networks"""
-v: remove the volumes
-rmi local: reove image build
docker-compose down

"""List all containers of compose"""
docker-compose ls


```



## Swarm

Create a manager node to manager a task using different worker nodes with scale up and scale out ability.

#### Initialize a Swarm node:

1. Root signing certificate create
2. Certificate is issued
3. Join tokens are created

#### Routing Mesh to make Swarm load balances (OSL layer 3 -> TCP):

1. Container to container in a Overlay network (uses VIP)
2. External traffic incoming to published ports (all nodes listen)

<img src="https://s2.loli.net/2022/06/21/1875iMwsSIRVluD.png" alt="image-20220620231739667" style="zoom:50%;" />

### Swarm Service (multi-services / multi-nodes / multi-container) 

1. Swarm -> multi-services
2. service -> multi-nodes (OS environments)
3. node -> multi-containers

```bash
"""1. Create a single manager node"""
docker swarm init
docker swarm init --advertise-addr IP_address

"""2. Check join token and join as worker"""
docker swarm join-token manager
docker swarm join --token '....'

"""3. List all the nodes"""
docker node ls

"""4. Change the role of node"""
docker node update --role manager node_name

"""5. Create network for swarm"""
docker network create --driver/-d overlay network_name

"""6. Run swarm service"""
docker service create image_name
# db
docker service create --name db --network network_name -e POSTGRES_HOST_AUTH_METHOD=trust --mount type=volume,source=db-data,target=/var/lib/postgresql/data image_name(postgres:9.4)
# backend
docker service create --name backend --network network_name -p 80:80 image_name(drupal)
# worker, can add multiply overlay network
docker service create --name worker --network frontend --network backend image_name

"""7. List swarm services"""
docker service ls

"""8. List all containers inside service"""
docker service ps service_name

"""9. Scale up the service (add more containers)"""
docker service update service_id --replicas num_containers

"""Check for log file"""
docker service logs service_name

"""10. Delete the service"""
docker service rm service_name/ID

"""11. Leave the swarm"""
docker swarm leave -f

```

#### 

### Swarm Stacks (multi-services in one step):

Docker add a new layer of abstraction to Swarm called Stacks, which **accept Docker compose files** as their declarative definition for services, networks and volumes.
Stacks will manages all objects for us, including overlay network per stack.

Need **deploy** in Compose file to use **Swarm Stack** and docker-compose ignore deploy.
Need **build** in Compose file to use **docker-compose** and Swarm ignore build.
So we don't need to change the file for different proposes.

<img src="https://s2.loli.net/2022/06/28/5CjU3YsGkum96yx.png" alt="image-20220627224623583" style="zoom:40%;" />

Example for docker-compose file

```yaml
services:

  redis:
    image: redis:alpine
    deploy:
    	# the num of containers for this service
      replicas: 1 
      update_config:
        parallelism: 2
        delay: 10s
        max_attempts: 3
        window: 120s
      # some paras for restart service
      restart_policy:
        condition: on-failure
      placement:
      	# set the role of node
        constraints: [node.role == manager]
```

Summary for CML

```bash
"""Deploy a swarm stack to create service using yml file """
-c: refers to compose
docker stack deploy -c docker_compose.yml service_name

"""List all swarm stack"""
docker stack ls

"""List all the services under a stack"""
docker stack services stack_name (show the replicas of services)
docker stack ps stack_name (show the placement of services)

"""Remove the swarm stack"""
docker stack remove stack_name
```



### Swarm Secrets Storage:

Storing secret information in Swarm, including password, SSH keys and TLS certificates key.

* Swarm Secrets only stored on disk of manager nodes, then assign to other services
* Only containers in assigned Service can see them
* /run/secrets/<secret_name> or /run/secrets/<secret_alias>

#### Use Secret in Swarm services:

```bash
"""Create secret from file"""
esc buttom
:w :refers to edit the file
:q :refers to quit the file

vim secret_name.txt
docker secret create secret_name secret_name.txt

"""Create secret from CML"""
-: refers to read the stdin to replace -
echo 'mypassword_or_key' | docker secret create secret_name - 

"""List all secret"""
docker secret ls

"""Detail info for secret"""
docker secret inspect secret_name

"""Pass secret to service"""
docker service create --name service_name --secret secret_name1 --secret secret_name2 -e DB_PASSWORD_FILE=/run/secrets/secret_name1 -e DB_USER_FILE=/run/secrets/secret_name2 image_name

"""Remove secret from service"""
docker service update --secret-rm
```



#### Use Secret in Swarm Stacks:

```bash
"""1.1 Create secret file in manager node (internal)"""
vim secret_name.txt

"""1.2 Create secret file using CML (external)"""
echo 'mypassword_or_key' | docker secret create secret_name -

"""3. Run Swarm Stack"""
docker stack deploy -c docker-compose.yml stack_name
```

```yaml
"""2. Edit docker-compose yaml file"""
version: "3.9"

services:
  psql:
    image: postgres
    # secret field for service
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

""" 2.1 set the path to the secret file (internal)"""
secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt
    
""" 2.2 pass the secret external"""
secrets:
  psql_password:
    external: true
```



#### Use Secret in Docker Compose

```bash
"""Start dcoker compose"""
docker-compose up -d

"""Pass secret inside container"""
docker-compose exec container_name cat /run/secret/secret_name
```



```yaml
"""Use same secret field in docker compose file (internal only)"""
services:
  psql:
    image: postgres
    # secret field for service
    secrets:
      - secret_name1
      - secret_name2
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/secret_name1
      POSTGRES_USER_FILE: /run/secrets/secret_name2

""" 2.1 set the path to the secret file (internal)"""
secrets:
  psql_user:
    file: ./secret_name1.txt
  psql_password:
    file: ./secret_name2.txt
```



### Swarm Service Update:

```bash
"""Update the replica / Scale the service"""
docker service scale service_name1=num_replicas1 service_name2=num_replicas2

"""Update the publish port"""
docker service update --publish-rm old_port --publish-add new_port:port2

"""Update the env parameters"""
docker service update --env-rm ENV_VAR=value1 --env-add ENV_VAR=value2

"""Rebalance the service"""
docker service update --force service_name

"""Rollback service to previous change"""
docker service rollback service_name
```



### Docker Compose: Dev/Test/Deploy

* **docker-compose.yml:** an interface layer to define all basic services info (not detail)
* **docker-compose.override.yml:** Dev propose, it contains the detail of containers under local development env and will be automated override based on docker-compose.yml file.
* **docker-compose.test.yml:** Test propose (CI env), it contains the detail of containers under CI env and will need to be specify override based on docker-compose.yml file.
* **docker-compose.prod.yml:** Production propose (Cloud/Server env), it contains the detail of containers under Cloud/Server env and will need to be specify override based on docker-compose.yml file.

```yaml
"""docker-compose.yml"""
version: '3.9'

services:
  drupal:
    image: custom-drupal:latest

  postgres:
    image: postgres:14.3

"""docker-compose.override.yml"""
services:

  drupal:
    build: .
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules # docker volumes
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - ./themes:/var/www/html/themes # blind mount
 
  postgres:
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw
    secrets:
      - psql-pw
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

secrets:
  psql-pw:
    file: psql-fake-password.txt
```

```bash
"""Dev Env (autoload override file)"""
docker-compose up id

"""Test Env"""
docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d

"""Production Env"""
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > docker-stack.yml

"""Deploy Env"""
docker stack deploy -c docker-stack.yml service_name
```



### Docker Healthchecks:

HEALTHCHECK supports in Dockerfile, Compose YAML, docker run and Swarm Services.
Docker engine will run healthcheck command in the container.
Exit0: OK / true, Exit1: error / false

Three states:

* Starting
* Healthy
* Unhealthy

#### Healthcheck in docker run

```bash
"""Show Healthcheck states"""
docker container ls
docker container inspect

"""Healthcheck in docker run"""
docker run \
	--health-cmd='curl -f http://localhost/ || exit 1' \ # Basic CML run inside
	--health-interval=5s \ # interval time to run health check
	--health-retried=3 \ # num of retired if unhealthy
	--health-timeout=2s \  # if failure last time
	--health-start-period=15s \  # buffer time before interval period
	image_name
	
```

#### Healthcheck in Dockerfile

```dockerfile
"""Healthcheck in Dockerfile"""
FROM nginx:1.13

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3\ 
	CMD curl -f http://localhost/ || exit 1

"""In DB"""
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3\ 
	CMD pg_isready -U postgres || exit 1
```

#### Healthcheck in docker compose file

```bash
services:
	web:
		image: image_name
		healthcheck:
			test:["CMD", "curl", "-f", "http://localhost"]
			interval: 1m30s
			timeout: 10s
			retries: 3
			start_period: 1m
```



## Kubernetes orchestration

Kubernetes (k8s):  is a container orchestration, which makes many servers act like one. It provides API/CLI to manage containers across servers.

Kubectl: CLI to configure K8s and manage apps.

Kubernetes can add 3rd party resources and controllers, which extends k8s API and CLI. (CRD)

* **Helm** (the most popular), keep k8s yaml easier to write.
* Compose on Kubernetes, use docker-compose.yml to translate into k8s yml, and it can use docker stack to deploy the app.

#### Kubernetes Role:

1. Control Plane (Master)
   necessary containers including:
   * Etcd: RAFT protocol
   * API Server: talk to other cluster
   * Scheduler: control the place of nodes
   * Controller Manager: take the order
   * Core DNS
2. Node: single server in the K8s cluster
   necessary containers including:
   * Kubelet: K8s agent running on nodes
   * Kube-proxy

#### Kubernetes Container Concepts:

* Pod: (basic unit of deployment)
  One or more containers running together on one Node
* Controller:
  For creating/updating pods and other objects
  Types: Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob
* Service:
  Network endpoint to connect to a pod
* Namespace:
  Filtered group of objects in cluster
* Secrets
* ConfigMaps

<img src="https://s2.loli.net/2022/06/29/jZkz7s1g2hnPFGN.png" alt="image-20220628184203881" style="zoom:50%;" />



### Three Management Approaches:

**Imperative**: Focus on how a program operates

**Declarative**: Focus on what a program should accomplish

1. Imperative commands: (run, expose, scale, create...)
   Best for dev/learning/personal projects, but hardest to manage over time.

2. Imperative objects: (create/replace/delete -f file.yml)

   Good for production of small environments, single file per command, can store your changes in git-based yaml files, but hard to automate.

3. Declarative objects: (apply -f file.yml)
   Best for production and easy to automate. Hard to understand and predict changes.

**Don't mix the three approaches!!!**



### Imperative Commands:

#### Kubernetes CML:

```bash
"""Check version"""
kubectl version

"""Create Pod"""
kubectl run pod_name --image image_name

"""Create a tmp Pod for test"""
--rm: refers to remove pod once leave
-it: refers to interactive mode
-- bash: refers to run shell inside pod
kubectl run pod_name --rm -it --image image_name -- bash

"""Create Deployment (include Pod + Controller)"""
"""automated restart pods once it stop with any reasons"""
--dry-run -o yaml: refers to output config file at same time
kubectl create deployment deployment_name --image image_name
kubectl create deployment deployment_name --image image_name --dry-run -o yaml

"""Create Job"""
"""automated restart pods only if it failures)"""
kubectl create job job_name --image image_name
kubectl create job job_name --image image_name --dry-run -o yaml

"""Scale up the deployment (add pods)"""
kubectl scale development deployment_name --replicas num_of_pod

"""List of Pods info"""
-w: refers to watch, keep output the change once happen
-l: refers to list pods with key:value
kubectl get pods
kubectl get pods -w
kubectl get pods -l label_key=label_value

"""List of all object info"""
kubectl get all

"""Check logs file"""
-l: refers to label
--tail: refers to tail line
kubectl logs deployment/deployment_name --follow --tail 1
kubectl logs pod/pod_name
kubectl logs -l run=deployment_name

"""Check object detail"""
kubectl describe pod pod_name
kubectl describe deployment deployment_name

"""Delete Deployment (include Pod + Controller)"""
kubectl delete deployment deployment_name

"""Delete Pod"""
kubectl delete pod pod_name

"""List namespaces (virtual cluster)"""
kubectl get namespaces

```



#### Kubernetes scale up process:

1. Development updated to n replicas
2. ReplicaSet Controller sets pod count to 2
3. Control Plane assigns node to pod
4. Kubelet sees pod is needed, start container



#### Kubernetes Exposing Containers:

**Service**: a stable address for pod(s), which can connect multiply pods together.
CoreDNS allows us to resolve services by name

Services Type:

1. ClusterIP (default, internal): 
   Single, internal virtual IP allocated, which only reachable from within cluster(nodes and pods)
   Pods can reach service on apps port number
2. NodePort (external): 
   High port allocated on each node, which is open on every node's IP
   Anyone can connect (Publish), if know this port number
3. LoadBalancer: 
   Controls a LoadBalancer endpoint external to the cluster
   Only available when infra provider gives you a LB (AWS, etc)
   Creates NodePort+ClusterIP services, tell LB to send to NodePort
4. ExternalName: 
   Adds CNAME DNS record to CoreDNS only
   Not use for pods but for giving pods a DNS name to use something outside K8s
5. Ingress (OSI Layer 7, HTTP)
   Ingress Controllers route outside connections based on hostname or URL by 3rd party proxies.

Outside -> LoadBalancer (if exist) -> NodePort -> inside Node -> ClusterIP -> access Cluster -> Communicate between pods

```bash
"""Create a ClusterIP service"""
--port: refers to port number
--name: refers to specify name
--type: refers to specify the service_type
--dry-run -o yaml: refers to output config file at same time
kubectl expose deployment clu_service_name --port clu_port_number

"""Create a NodePort service"""
kubectl expose deployment clu_service_name --port clu_port_num --name node_service_name --type NodePort

"""Create a LoadBalancer service"""
kubectl expose deployment clu_service_name --port clu_port_num --name lb_service_name --type LoadBalancer

"""List all services"""
kubectl get service

"""Delete services"""
kubectl delete service service_names

```



### Declarative objects:

Using declarative way, which only use **yaml file** and **apply** command to create our infrastructure.

```bash
"""Create/Update from a yaml file"""
-l: refers to only apply resources with label
kubectl apply -f file_name.yml
kubectl apply -f file_name.yml -l label_name=label_value

"""Create/Update from a dir of yaml file"""
kubectl apply -f dir_name/

"""Create/Update from a URL"""
kubectl apply -f url/file_name.yml

"""Test yaml file before Update(Dry Run)"""
--dry-run: return a output to tell you the change depend on file change (client side)
--server-dry-run: return a output to tell you the change depend on file change (server side)
kubectl apply -f file_name.yml --dry-run
kubectl apply -f file_name.yml --server-dry-run

"""Compare file difference""
kubectl diff -f file_name.yml

```



#### Kubernetes Configuration YAML

Each file contains one or more **manifests** and each manifests describes an **API object** (pod, deployment, job, secret, service...)

Manifest needs four parts of key:values in the file:

1. apiVersion: we can get the API versions the cluster supports
2. kind: we can get a list of resources the cluster supports

3. metadata: only name is required
   * **labels:** 
     Simple list of key:value for identifying your resource later by selection, grouping and filtering for it (glue together).
     EX: tier:frontend,  app:api,  env:prod,  customer:acme
4. spec: Where all the action is at
   * **selector / selector.matchLabels:** 
     Select the resources with same key:value pair to do action, which also can control the placement of pods
5. ---: use this to split different manifests in one file

```bash
"""Get all the resources list (version + kind)"""
kubectl api-resources

"""Get all the api list (version)"""
kubectl api-versions

"""Get all the keys of spec"""
kubectl explain services.spec.kind_name
```



```yaml
# Manifest1
apiVersion: v1
kind: Service
metadata:
  name: app-nginx-service
spec:
  type: NodePort
  ports:
    - port: 80
  # apply on resource with same label
  selector:
    app: app-nginx
---
# Manifest2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-nginx-deployment
spec:
  replicas: 3
  # select pods with same label
  selector:
    matchLabels:
      app: app-nginx
  template:
    metadata:
    	# create a label for this resource
      labels:
        app: app-nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.3
          ports:
            - containerPort: 80
```



### Volumes in Kubernetes:

1. Volumes
   * Tied to lifecycle of a Pod
   * All containers in a single Pod can share them
2. PersistentVolumes
   * Create at cluster level and outlive a Pod
   * Separate storage config from Pod
   * Multiple Pods can share them

3. CSI plugins



















