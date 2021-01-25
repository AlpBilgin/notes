# Docker

## Restart policy debugging:

https://stackoverflow.com/questions/29680274/how-to-check-if-the-restart-policy-works-of-docker/35086537#35086537

After running the container you can inspect its policy, restart coun and last started time:
docker inspect -f "{{ .HostConfig.RestartPolicy }}" <container_id>
docker inspect -f "{{ .RestartCount }}" <container_id>
docker inspect -f "{{ .State.StartedAt }}" <container_id>
Then you can look into container processes:
docker exec -it <container_id> ps -aux
The PID 1 process - is the main process, after its death the whole container would die.
Kill him using:
docker exec -it <container_id> kill -9  (maybe omit -9 option? it somehow obstructed the test on UI test iMAC (28))
And after this ensure that the container autorestarted:
docker inspect -f "{{ .RestartCount }}" <container_id>


## backup and restore folders in volumes

Volume Backup Tip:

in the same folder as dockerfile
`docker run --rm --volumes-from <running docker container id> -v $(pwd):/backup ubuntu tar cvf /backup/<backup tar file name> /<folder in container to backup>`

this command will start a new container that has access to the volumes of the indicated container

`-v $(pwd):/backup ubuntu` => this bit from above command will create a /backup folder in the new container that is mapped to $(pwd) in real filesystem

`tar cvf /backup/<backup tar file name> /<folder in container to backup>` => this bit from above will run in the container and will indireclty cause the tar file to appear in $(pwd)


`docker run --rm --volumes-from <running docker container id> -v $(pwd):/backup ubuntu bash -c "cd /<folder in container to restore> && tar xvf /backup/<backup tar file name> --strip 1"`

this command will start a new container that has access to the volumes of the indicated container

`-v $(pwd):/backup ubuntu` => this will create a /backup folder in the new container that is mapped to $(pwd) in real filesystem

`bash -c "cd /<folder in container to restore> && tar xvf /backup/<backup tar file name> --strip 1"` => this will extract the tarfile into some folder. Hopefully the same folder as above





## to be parsed

Docker
Docker is a command line program, a background daemon and a set of remote services that take a logistical approach to solving common
software problems and simplifying your experience installing, running, publishing and removing software.It accomplishes this using a unix
technology called containers.
The containers that Docker builds are isolated with respect to eight aspects.
1.PID namespace-Process identifiers and capabilities
2.UTS namespace-Host and domain name
3.MNT namespace-File system access and structure
4.IPC namespace-Process communication over shared memory
5.NET namespace-Network access and structure
6.USR namespace-User names and identifiers
7.chroot()-Controls the location of the file system root
8.cgroups -Resource protection
Docker Images
A Docker image is a file containing the code and components needed to run software in a container.
Containers and images use a layered file system.Each layer contains only the differences from the previous layer.
The image consists of one or more read-only layers,while the container adds one addition writable layer.
The layered file system allows multiple images and containers to share the same layers. This results in:
Smaller overall storage footprint.
Faster image transfer.
Faster image build.
Docker Images
Download an image from a remote registry to the local machine.
docker image pull IMAGE[:TAG]
List the layers used to build an image.
docker image history IMAGE
Run a container. The image will be automatically downloaded if it does not exist on the system:
docker run nginx:1.15.8
Download an image:
docker image pull nginx
View file system layers in an image:
docker image history nginx
The components of a Dockerfile
If you want to create your own images, you can do so with a Dockerfile.
A Dockerfile is a set of instructions which are used to construct a Docker image. These instructions are called directives.
FROM: Starts a new build stage and sets the base image. Usually must be the first directive in the Dockerfile (except ARG can be placed before
from)
ENV: Set environment variables. These can be referenced in the Dockerfile itself and are visible to the container at runtime.
RUN: Creates a new layer on top of the previous layer by running a command inside that new layer and committing the changes.
CMD: Specify a default command used to run a container at execution time.
EXPOSE: Documents which ports are intended to published when running a container.
WORKDIR: Sets the current working directory for subsequent directives such as ADD,COPY,CMD,ENTRYPOINT, etc. Can be used multiple
times to change directories throughout the Dockerfile. You can also use a relative path, which sets the new working directory relative to the
previous working directory.
COPY: Copy files from the local machine to the image.
ADD: Similar to COPY, but can also pull files using a URL and extract an archive into loose files in the image.
STOPSIGNAL: Specify the signal that will be used to stop the container
HEALTHCHECK: Specify a command to run in order to perform a custom health check to verify that the container is working properly.
Dockerfile
FROM ubuntu:bionic
ENV NGINX_VERSION 1.14.0-0ubuntu1.3
RUN apt-get update && apt-get install -y curl
RUN apt-get update && apt-get install -y nginx=$NGINX_VERSION
WORKDIR /var/www/html
ADD index.html ./
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
STOPSIGNAL SIGTERM
HEALTHCHECK CMD curl localhost:80
Building an Image
once you have a Dockerfile, you are ready to build your image!
docker build -t TAG .
eg: docker build -t custom-nginx .
Then you can run your image:
docker run IMAGE
docker run --name custom-nginx -d -p 8080:80 custom-nginx
curl localhost:8080
Efficient Docker Images
When working with Docker in the real world, it is important to create Docker images that are as efficient as possible.
This means that they are as small as possible and result in ephemeral containers that can be started,stopped, and destroyed easily.
General tips:
Put things that are less likely to change on lower-level layers.
Don't create unnecessary layers.
Avoid including any unnecessary files,packages, etc. in the image.
Multi-Stage Builds
Docker supports the ability to perform multi-stage builds. Multi-stage builds have more than one FROM directive in the Dockerfile,with each
FROM directive starting a new stage.
Each stage begins a completely new set of file system layers, allowing you to selectively copy only the files you need from previous layers.
Use the --from flag with COPY to copy files from a previous stage.
COPY --from=0 ...
You can also name your stages with FROM... AS,then reference the name with :
FROM <image> AS stage1
...
FROM <Image> AS stage2
COPY --from=stage1 ...
Create some project directories:
cd ~/
mkdir efficient
mkdir inefficient
cd inefficient
Create the source code file:
vi helloworld.go
package main
import "fmt"
func main() {
fmt.Println("hello world")
}
Create the Dockerfile:
vi Dockerfile
FROM golang:1.12.4
WORKDIR /helloworld
COPY helloworld.go .
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .
CMD ["./helloworld"]
Build and test the inefficient image:
docker build -t inefficient .
docker run inefficient
docker image ls
Switch to the efficient project directory and copy the files from the inefficient project:
cd ~/efficient
cp ../inefficient/helloworld.go ./
cp ../inefficient/Dockerfile ./
Change the Dockerfile to use a multi-stage build:
vi Dockerfile
FROM golang:1.12.4 AS compiler
WORKDIR /helloworld
COPY helloworld.go .
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .
FROM alpine:3.9.3
WORKDIR /root
COPY --from=compiler /helloworld/helloworld .
CMD ["./helloworld"]
Build and test the efficient image:
docker build -t efficient .
docker run efficient
docker image ls
Managing Images
Download an image from a remote registry.
docker image pull IMAGE[:TAG]
Basically docker image pull and docker pull is more less same.
List images.
docker image ls
Add the -a flag to include intermediate images
docker image ls -a
Get detailed information about an image
docker image inspect IMAGE
Use the --format flag to get only a subset of the information(uses Go templates).
docker image inspect IMAGE --format TEMPLATE
These commands can both be used to delete an image. Note that if an image has other tags, they must be deleted first.
docker image rm IMAGE
docker rmi IMAGE
Use the -f flag to automatically remove all tags and delete the image.
docker image rm -f IMAGE_ID
docker rmi -f IMAGE_ID
Delete unused images from the system.
docker image prune
Download an image:
docker image pull nginx:1.14.0
List images on the system:
docker image ls
docker image ls -a
Inspect image metadata:
docker image inspect nginx:1.14.0
docker image inspect nginx:1.14.0 --format "{{.Architecture}}"
docker image inspect nginx:1.14.0 --format "{{.Architecture}} {{.Os}}"
Delete an image:
docker image rm nginx:1.14.0
Force deletion of an image that is in use by a container:
docker run -d --name nginx nginx:1.14.0
docker image rm -f nginx:1.14.0
Locate a dangling image and clean it up:
docker image ls -a
docker container ls
docker container rm -f nginx
docker image ls -a
docker image prune
Flattening a Docker Image to a Single Layer
Sometimes, images with fewer layers can perform better. In a few cases, you may want to take an image with many layers and flatten them into a
single layer.
Docker does not provide an official method for doing this, but you can accomplish it by doing the following:
Run a container from the image.
Export the container to an archive using docker export.
Import the archive as a new image using docker import.
The resulting image will have only one layer!
Set up a new project directory to create a basic image:
cd ~/
mkdir alpine-hello
cd alpine-hello
vi Dockerfile
Create a Dockerfile that will result in a multi-layered image:
FROM alpine:3.9.3
RUN echo "Hello, World!" > message.txt
CMD cat message.txt
Build the image and check how many layers it has:
docker build -t nonflat .
docker image history nonflat
Run a container from the image and export its file system to an archive:
docker run -d --name flat_container nonflat
docker export flat_container > flat.tar
Import the archive to a new image and check how many layers the new image has:
cat flat.tar | docker import - flat:latest
docker image history flat
Docker Registries
A Docker Registry is responsible for storing and distributing Docker images.
we have already pulled images from the default public registry, Docker Hub
You can also create your own registries using Docker’s open source registry software, or Docker Trusted Registry, the non-free enterprise
solution.
To create a basic registry, simply run a container using the registry image and publish port 5000.
Registry Server:
Distribution - Ubuntu 18.04 Bionic Baver LTS
Size - Small
Configuring a Registry
You can override individual values in the default registry configuration by supplying environment variables with docker run -e.
Name the variable REGISTRY_ followed by each configuration key, all uppercase and separated by underscores.
For example, to change the config:
log:
level: info
set the environment variable:
REGISTRY_LOG_LEVEL=debug
Securing a Registry
By default, the registry is completely unsecured. It does not use TLS and does not require authentication.
You can take some basic steps to secure your registry:
Use TLS with a certificate.
Require user authentication.
Run a simple registry:
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker container stop registry && docker container rm -v registry
Override the log level using an environment variable:
docker logs registry
docker container stop registry && docker container rm -v registry
docker run -d -p 5000:5000 --restart=always --name registry -e
REGISTRY_LOG_LEVEL=debug registry:2
docker logs registry
docker container stop registry && docker container rm -v registry
Secure the registry by generating an htpasswd file to be used for authentication:
mkdir ~/registry
cd ~/registry
mkdir auth
docker run --entrypoint htpasswd registry:2.7.0 -Bbn testuser password
> auth/htpasswd
Generate a self-signed certificate. When generating the cert, leave the prompts blank except for Common Name*. For *Common Name, put the
public hostname of the registry server. The public hostname is in the playground interface:
mkdir certs
openssl req \
-newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
-x509 -days 365 -out certs/domain.crt
Run the registry with authentication and TLS enabled:
docker run -d -p 443:443 --restart=always --name registry \
-v /home/cloud_user/registry/certs:/certs \
-v /home/cloud_user/registry/auth:/auth \
-e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
-e REGISTRY_AUTH=htpasswd \
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
registry:2
Using Docker Registries
Download an image from a registry to the local system.
docker pull IMAGE[:TAG]
Authenticate with a remote registry.
docker login REGISTRY
When working with Docker registries, if the registry is not specified, the default registry will be used(Docker Hub).
There are multiple ways to use a registry with a self-signed certificate.
Turn off certificate verification(Very insecure)
Provide the public certificate to the Docker engine.
To push and pull images from your private registry,tag the images with the registry hostname( and optionally, port).
<registry public hostname>/<image name>:<tag>
upload the image to a remote registry.
docker push IMAGE
Pull and search images on Docker hub:
docker pull ubuntu
docker search ubuntu
Attempt to authenticate against the private registry:
docker login <registry public hostname>
Log in with the credentials we created earlier (testuser and password). A certificate signed by unknown authority message
should pop up, because we are using a self-signed certificate.
Configure docker to ignore certificate verification when accessing the private registry:
sudo vi /etc/docker/daemon.json
{
"insecure-registries" : ["<registry public hostname>"]
}
Restart docker:
sudo systemctl restart docker
Try docker login again:
docker login <registry public hostname>
This time it should work!
However, this method of accessing the registry is very insecure. It turns off certificate verification entirely, exposing us to man-in-the-middle
attacks. So, let's do this the right way.
First, log out of the private registry:
docker logout <registry public hostname>
Next, remove the insecure-registries key and value from /etc/docker/daemon.json.
Restart Docker:
sudo systemctl restart docker
Download the cert public key from the registry and configure the local docker engine to use it:
sudo mkdir -p /etc/docker/certs.d/<registry public hostname>
sudo scp cloud_user@<registry public hostname>:/home/cloud_user/registry
/certs/domain.crt /etc/docker/certs.d/<registry public hostname>
Try docker login:
docker login <registry public hostname>
Push to and pull from your private registry:
docker pull ubuntu
docker tag ubuntu <registry public hostname>/ubuntu
docker push <registry public hostname>/ubuntu
docker image rm <registry public hostname>/ubuntu
docker image rm ubuntu
docker pull <registry public hostname>/ubuntu
Locking and Unlocking a Swarm Cluster
1.Enable autolock.
docker swarm update --autolock=true
2.Make note of the unlock key!
3.Run a command to interact with the swarm, then restart docker and try
the command again to verify that the swarm is locked.
docker node ls
sudo systemctl restart docker
docker node ls
4.Unlock the swarm using the unlock key and verify that it is unlocked.
docker swarm unlock
docker node ls
5.Obtain the existing unlock key.
docker swarm unlock-key
6.Rotate the unlock key.
docker swarm unlock-key --rotate
7.Disable autolock
docker swarm update --autolock=false
sudo systemctl restart docker
docker node ls
High Availability in a Swarm Cluster
Multiple Managers
In order to build a highly-available and fault-tolerant swarm, it is a
good idea to have multiple swarm managers.
Docker uses the Raft consensus algorithm to maintain a consistent
cluster state across multiple managers.
More manager nodes means better fault tolerance.However, there can be a
decrease in performance as the number of managers grows,since more
managers means more network traffic as managers agree to updates in the
cluster state
Quorum
A Quorum is the majority(more than half) of the managers in a swarm.For
example,for a swarm with 5 managers,the quorum is 3.
A quorum must be maintained in order to make changes to the cluster
state. If a quorum is not available, nodes cannot be added or removed,
new tasks cannot be added, and existing tasks cannot be changed or
moved.
Note that since a quorum requires more than half of the manager nodes,
it is recommended to have an odd number of managers.
Availability Zones
Docker recommends that you distribute your manager nodes across at
least 3 availability zones.
Distribute your managers across these zones so that you can maintain a
quorum if one of them goes down
Manger Nodes Availability Zone Distribution
3 1-1-1
5 2-2-1
7 3-2-2
9 3-3-3
Docker services
A service is used to run an application on a Docker Swarm. A service specifies a set ofone or more replica tasks. These tasks will be distributed
automatically accross the nodes in the cluster and executed as containers
docker service create nginx
docker service create --name nginx --replicas 3 -p 8080:80 nginx
curl localhost:8080
Templates can be used to give somewhat dynamic values to some flags with docker service create
The following flags accept templates
--hostname
--mount
--env
This command sets an environment variable for each container that contains the hostname of the node that container is running on
docker serivce create --name node-hostname --env NODE_HOSTNAME=”{{.Node.Hostname}}” --replicas 3 nginx
docker ps
docker exec dff1f29d622a printenv
Managing Services
List current services:
docker service ls
List a service’s tasks:
docker service ps SERVICE
Get more information about a service:
docker service inspect SERVICE
Make changes to a service:
docker service update [options] SERVICE
docker service update --replicas 2 nginx
Delete an existing service
docker service rm SERVICE
Replicated vs Global Services
Replicated Services run the requested number of replica tasks across the swarm cluster
docker service create --replicas 3 nginx
Global Services run one task on each node in the cluster
docker service create --mode global nginx
Scaling Services
Scaling services means changing the number of replica tasks
There are two ways to scale a service Update the service with a new number of replicas.
docker service update --replicas REPLICAS SERVICE
Use docker service scale
docker service scale nginx=4
Create a simple service running the nginx image.
docker service create nginx
Create an nginx service with a specified name, multiple replicas, and a published port.
docker service create --name nginx --replicas 3 -p 8080:80 nginx
Use a template to pass the node hostname to each container as an environment variable.
docker service create --name node-hostname --replicas 3 --env NODE_HOSTNAME="{{.Node.Hostname}}" nginx
Get the container running on the current machine, and print its environment variables to verify that the NODE_HOSTNAME variable is set properly.
docker ps
docker exec <CONTAINER_ID> printenv
List the services in the cluster.
docker service ls
List the tasks for a service
docker service ps nginx
Inspect a service.
docker service inspect nginx
docker service inspect --pretty nginx
Change a service.
docker service update --replicas 2 nginx
Delete a service.
docker service rm nginx
Create a global service.
docker service create --name nginx --mode global nginx
Two different ways to scale a service:
docker service update --replicas 3 nginx
docker service scale nginx=4
Docker Inspect
Docker inspect is a command that allows yout to get information about Docker objects,such as containers,images,services,etc.
docker inspect OBJECT_ID
if you know what kind of object you are inspecting you can also use an alternate form of the command:
docker container inspect CONTAINER
docker service inspect SERVICE
This form allows you to specify an object name instead of an ID.
For some object types,you can also supply the --pretty flag to get a more readable output.
Docker inspect --format
Use the --format flag to retrieve a specific subsection of the data using a Go template
docker service inspect --format='{{.ID}}' SERVICE
Run a container and inspect it.
docker run -d --name nginx nginx
docker inspect <CONTAINER_ID>
List the containers and images to get their IDs, then inspect an image.
docker container ls
docker image ls
docker inspect <IMAGE_ID>
Create and inspect a service.
docker service create --name nginx-svc nginx
docker service ls
docker inspect <SERVICE_ID>
docker inspect nginx-svc
Use the type-specific commands to inspect objects.
docker container inspect nginx
docker service inspect nginx-svc
Use the --format flag to retrieve a subset of the data in a specific format
docker service inspect --format='{{.ID}}' nginx-svc
Docker Compose
Docker Compose is a tool that allows you to run multi-container applications defined using a declarative format.
Docker Compose uses YAML files to declaratively define a set of containers and other resources that will be created as part of the larger
application.
Setting up a new Docker compose project:
Make a directory to contain your Docker Compose project.
change to the project directory.
Add a docker.compose.yml file to the directory
Define your application in docker-compose.yml
Install Docker Compose.
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/dockercompose
sudo chmod +x /usr/local/bin/docker-compose docker-compose version
Set up a Docker Compose project.
mkdir nginx-compose
cd nginx-compose
vi docker-compose.yml
Create your docker-compose.yml.
version: '3'
services:
web:
image: nginx
ports:
- "8080:80"
redis:
image: redis:alpine
Start your Compose app.
docker-compose up -d
List the Docker Compose containers, then stop the app.
docker-compose ps
docker-compose down
Docker Stacks
Services are capable of running a single,replicated application across nodes in the cluster,but what if you need to deploy a more complex
application consisting of multiple services?
A Stack is a collection of interrelated services that can be deployed and scaled as a unit.
Docker stacks are similat to the multicontainer applications created using Docker Compose. However,they can be scaled and executed across
the swarm just like normal swarm services.
Deploy a new Stack to the cluster using a compose file
docker stack deploy -c COMPOSE_FILE STACK
List current stacks
docker stack ls
List the tasks associated with a stack
docker stack ps STACK
List the services assosciated with a stack
docker stack services STACK
Delete a stack
docker stack rm STACK
Create a compose file for the stack.
vi simple-stack.yml
version: '3'
services:
web:
image: nginx
busybox:
image: radial/busyboxplus:curl
command: /bin/sh -c "while true; do echo Hello!; sleep 10; done"
Deploy the stack and examine it using various commands.
docker stack deploy -c simple-stack.yml simple
docker stack ls
docker stack ps simple
docker stack services simple
docker service logs simple_busybox
Modify the stack to use an environment variable
vi simple-stack.yml
version: '3'
services:
web:
image: nginx
busybox:
image: radial/busyboxplus:curl
command: /bin/sh -c "while true; do echo $$MESSAGE; sleep 10; done"
environment:
- MESSAGE=Hello!
docker stack deploy -c simple-stack.yml simple
docker service logs simple_busybox
Modify the stack to expose a port.
vi simple-stack.yml
version: '3'
services:
web:
image: nginx
ports:
- "8080:80"
busybox:
image: radial/busyboxplus:curl
command: /bin/sh -c "while true; do echo $$MESSAGE; sleep 10; done"
environment:
- MESSAGE=Hello!
docker stack deploy -c simple-stack.yml simple
curl localhost:8080
Modify the stack to use the BusyBox service to communicate with the web service.
vi simple-stack.yml
version: '3'
services:
web:
image: nginx
ports:
- "8080:80"
busybox:
image: radial/busyboxplus:curl
command: /bin/sh -c "while true; do echo $$MESSAGE; curl web:80;
sleep 10; done"
environment:
- MESSAGE=Hello!
docker stack deploy -c simple-stack.yml simple
Delete the stack.
docker stack rm simple
Node labels
You can add pieces of metadata to your swarm nodes using node labels.
You can then use these labels to determine which nodes tasks will run on.
Exampe use case:
with nodes in multiple datacenters or availability zones, you can use labels to specify which zone each node is in. Then,execute tasks in specific
zones or distribute them evenly across zones.
Add a label to a node
docker node update --label-add LABEL=VALUE NODE
You can view existing node labels with
docker node inspect --pretty NODE
Node Constraints
To run a service’s tasks only on nodes with a specified label value,use the --constraint flag with docker service create
docker service create --constraint node.labels.LABEL== VALUE IMAGE
You can also use a constraint to run only on nodes without a particular value.
docker service create --constraint node.labels.LABEL!=VALUE IMAGE
You can use the --constraint flag multiple times to list multiple constraints. All constraints must be satisfied for tasks to run on a node.
Placement-pref
Use --placement-pref with the spread strategy to spread tasks evenly across all values of a particular label
docker service create --placement-pref spread=node.labels.LABEL IMAGE
for example,if you have a label called availability_zone with three values(east,west,and south),the tasks will be divided evenly among the node
groups with each of these three values,no matter how many nodes are in each group.
List your current nodes.
docker node ls
Add a label to a node.
docker node update --label-add availability_zone=east <NODE_NAME>
docker node update --label-add availability_zone=west <NODE_NAME>
View existing labels with:
docker node inspect --pretty <NODE_NAME>
You can use --constraint when creating a service to restrict which nodes will be used to execute a service's tasks.
docker service create --name nginx-east --constraint node.labels.
availability_zone==east --replicas 3 nginx
docker service ps nginx-east
docker service create --name nginx-west --constraint node.labels.
availability_zone!=east --replicas 3 nginx
docker service ps nginx-west
Use --placement-pref to spread tasks evenly based on the value of a specific label.
docker service create --name nginx-spread --placement-pref spread=node.
labels.availability_zone --replicas 3 nginx
docker service ps nginx-spread
Storage Drivers
Storage drivers are sometimes known as Graph drivers. The proper storage driver to use often depends on your operating system and other local
configuration factors.
overlay2: Current Ubuntu and CentOS/RHEL versions.
aufs: Ubuntu 14.04 and older
devicemapper: CentOS 7 and earlier.
Storage models
Persistent data can be managed using several storage models.
Filesystem storage:
Data is stored in the form of a file system.
Used by overlay2 and aufs.
Efficient use of memory.
Inefficient with write-heavy workloads.
Block storage
Stores data in blocks.
used by devicemapper
Efficient with write-heavy workloads.
Object storage
Stores data in an external object-based store.
Application must be designed to use object-based storage.
Flexible and Scalable.
Run a basic container:
docker run --name storage_nginx nginx
Use docker inspect to find the location of the container's data on the host:
docker container inspect storage_nginx
ls /var/lib/docker/overlay2/<STORAGE_HASH>/
Use docker inspect to find the location of an image's data:
docker image inspect nginx
Configuring DeviceMapper
Device Mapper is one of the Docker storage drivers available for some linux distributions. It is the default storage driver for CentOS7 and earlier
You can customize your Device Mapper configuration using the daemon config file.
DeviceMapper supports two modes:
loop-lvm mode:
Loopback mechanism simulates an additional physical disk using files on the local disk.
Minimal setup,does not require an additional storage device.
Bad performance,only use for testing.
direct-lvm mode:
Stores data on a seperate device.
Requires an additional storage device.
Good performance,use for production.
For this scenario, use a CentOS 7 server with a size of Small. Before starting the scenario, you'll first need to install Docker.
Add a new storage device to your server. In Playground, select Actions, then select Add /dev/nvme1n1 and wait for it to finish adding the
device.
Stop and disable Docker.
sudo systemctl disable docker
sudo systemctl stop docker
Delete any existing Docker data.
sudo rm -rf /var/lib/docker
Configure DeviceMapper in daemon.json.
sudo vi /etc/docker/daemon.json
{
"storage-driver": "devicemapper",
"storage-opts": [
"dm.directlvm_device=/dev/nvme1n1",
"dm.thinp_percent=95",
"dm.thinp_metapercent=1",
"dm.thinp_autoextend_threshold=80",
"dm.thinp_autoextend_percent=20",
"dm.directlvm_device_force=true"
]
}
Start and enable Docker.
sudo systemctl enable docker
sudo systemctl start docker
Check the storage driver information provided by docker info.
docker info
Run a container to verify that everything is working.
docker run hello-world
Bind Mounts vs. Volumes
When mounting external storage to a container, you can use either a bind mount or a volume.
Bind Mounts
Mount a specific path on the host machine to the container.
Not portable,dependent on the host machine’s file system and directory structure.
Volumes
Stores data on the host file system,but the storage location is managed by Docker.
More portable.
Can mount the same volume to multiple containers.
Work in more scenarios.
Working with Volumes
--mount syntax:
docker run --mount[key=value][,key=value..]
type: bind(bind mount),volume, or tmpfs(temporary in-memory storage)
source,src: Volume name or bind mount path.
destination,dst,target: Path tp mount inside the container.
readonly: Make the volume or bind mount read-only.
-v syntax
docker run -v SOURCE:DESTINATION[:OPTIONS]
SOURCE: If this is a volume name, it will create a volume.If this is a path,it will create a bind mount
DESTINATION: Location to mount the data inside the container.
OPTIONS: Comma-seperated list of options. For example,ro for read-only
You can mount the same volume to multiple containers,allowing them to interact with one another by sharing data!
Just mount the volume to both using the same volume name.
docker run --name container1 --mount source=shared-vol ..
docker run --name container2 --mount source=shared-vol ..
You can also use --mount with services!
Docker containers are designed so that their internal storage can be easily destroyed. However, sometimes you might need more permanent
data. Docker volumes and bind mounts allow you to attach external storage to containers. In this lesson, we will discuss bind mounts and
volumes. We will also demonstrate how to create them using both the --mount and -v flags. We will talk about sharing volumes between
multiple containers and go over some commands you can use to manage volumes.
Create a directory on the host with some test data.
cd ~/
mkdir message
echo Hello, world! > message/message.txt
Mount the directory to a container with a bind mount.
docker run --mount type=bind,source=/home/cloud_user/message,destination=/root,readonly busybox cat /root
/message.txt
Run a container with a mounted volume.
docker run --mount type=volume,source=my-volume,destination=/root busybox sh -c 'echo hello > /root
/message.txt && cat /root/message.txt'
Use the -v syntax to create a bind mount and a volume.
docker run -v /home/cloud_user/message:/root:ro busybox cat /root
/message.txt
docker run -v my-volume:/root busybox sh -c 'cat /root/message.txt'
Use a volume to share data between containers.
docker run --mount type=volume,source=shared-volume,destination=/root
busybox sh -c 'echo I wrote this! > /root/message.txt'
docker run --mount type=volume,source=shared-volume,destination=
/anotherplace busybox cat /anotherplace/message.txt
Create and manage volumes using docker volume commands.
docker volume create test-volume
docker volume ls
docker volume inspect test-volume
docker volume rm test-volume
Image Cleanup
Get information about disk usage on a system
docker system df
Get even more detailed disk usage information
docker system df -v
Remove dangling images(images not referenced by any tag or container)
docker image prune
Remove all unused images(not used by a container)
docker image prune -a
Display the storage space being used by Docker.
docker system df
Display disk usage by individual objects.
docker system df -v
Delete dangling images (images with no tags or containers).
docker image prune
Pull an image not being used by any containers, and use docker image prune -a to clean up all images with no containers.
docker image pull nginx:1.14.0
docker image prune -a
Storage in a Cluster
When working with multiple Docker machines,such as a swarm cluster,you may need to share Docker volume staorage between those machines.
Some options:
Use application logic to store data in external object storage.
Use a volume driver to create a volume that is external to any specific machine in your cluster.
Install the vieux/sshfs plugin on all nodes in the swarm.
docker plugin install --grant-all-permissions vieux/sshfs
Set up an additional server to use for storage. You can use the Ubuntu 18.04 Bionic Beaver LTS image with a size of Small. On this new
storage server, create a directory with a file that can be used for testing.
mkdir /home/cloud_user/external
echo External storage! > /home/cloud_user/external/message.txt
On the swarm manager, manually create a Docker volume that uses the external storage server for storage. Be sure to replace the text <STORAG
E_SERVER_PRIVATE_IP> and <PASSWORD> with actual values.
docker volume create --driver vieux/sshfs \
-o sshcmd=cloud_user@<STORAGE_SERVER_PRIVATE_IP>:/home/cloud_user
/external \
-o password=<PASSWORD> \
sshvolume
docker volume ls
Create a service that automatically manages the shared volume, creating the volume on swarm nodes as needed. Be sure to replace the text <ST
ORAGE_SERVER_PRIVATE_IP> and <PASSWORD> with actual values.
docker service create \
--replicas=3 \
--name storage-service \
--mount volume-driver=vieux/sshfs,source=cluster-volume,destination=
/app,volume-opt=sshcmd=cloud_user@<STORAGE_SERVER_PRIVATE_IP>:/home
/cloud_user/external,volume-opt=password=<PASSWORD> busybox cat /app
/message.txt
Check the service logs to verify that the service is reading the test data from the external storage server.
docker service logs storage-service
Docker Networking
Docker uses an architecture called the Container Networking Model(CNM) to manage networking for Docker containers.
The CNM utilizes the following concepts:
Sandbox: An isolated unit containing all networking components associated with a single container. Usually a Linux Network namespace.
Endpoint: Connects a sandbox to a network. Each sandbox/container can have any number of endpoints,but has exactly one endpoint for each
network it is connected to.
Network: A Collection of endpoints connected to one another.
Network Driver: Handles the actual implementation of the CNM concepts.
IPAM Driver: IPAM means IP Address Management. Automatically allocates subnets and IP Addresses for networks and endpoints.
Network Drivers
Docker includes several built-in network drivers,know as Native Network Drivers.
These network drivers implement the concepts described in the Container Networking Model(CNM).
The Native Network Drivers are:
Host
Bridge
Overlay
MACVLAN
None
The Host Network Driver
The Host Network Driver allows containers to use the host’s network stack directly.
Containers use the host’s networking resources directly.
No sandboxes,all containers on the host using the host driver share the same network namespace.
No two containers can use the same port(s).
Use Cases: Simple and easy setup, one or only a few containers on a single host.
The Bridge Network Driver
The Bridge Network Driver uses Linux bridge networks to provide connectivity between containers on the same host.
This is the default driver for containers running on a single host(i.e, not in a swarm)
Creates a Linux Bridge for each Docker network.
Creates a default Linux bridge network called bridge0. Containers automatically connect to this if no other network is specified.
Use Cases: Isolated networking among containers on a single host.
The Overlay Network Driver
The Overlay Network driver provides connectivity between containers across multiple Docker Hosts, i.e with Docker swarm.
Uses a VXLAN data plane,which allows the underlying network infrastructure(underlay) to route data between hosts in a way that is
transparent to the containers themselves.
Automatically configures network interfaces,bridges,etc. on each host as needed.
Use Cases: Networking between containers in a Swarm
The MACVLAN Network Driver
The MACVLAN Network Driver offers a more lightweight implementation by connectiong container inferaces directly to host interfaces.
Uses direct association with Linux interfaces instead of a bridge interface.
Harder to configure and greater dependency between MACVLAN and the external network.
More lightweight and less latency.
Use cases: When there is a need for extremely low latency,or a need for containers with IP addresses in the external subnet.
The None Network Driver
The None Network Driver does not provide any networking implementation.
Container is completely isolated from other containers and the host.
If you want networking with the None driver,you must set everything up manually.
None does create a seperate networking namespace for each container, but no interfaces or endpoints.
Use cases: When there is no need for container networking or you want to set all of the networking up yourself.
Host
Create two containers and demonstrate communication between them using the host network driver. The ip add commands also demonstrate
that the container is using the host's eth0 network interface directly
docker run -d --net host --name host_busybox radial/busyboxplus:curl
sleep 3600
docker run -d --net host --name host_nginx nginx
ip add | grep eth0
docker exec host_busybox ip add | grep eth0
docker exec host_busybox curl localhost:80
curl localhost:80
Bridge
Create two containers and demonstrate that they can communicate using a custom bridge network. The ip link command can be used to
explore the Linux bridge interfaces created by the bridge network driver.
ip link
docker network create --driver bridge my-bridge-net
ip link
docker run -d --name bridge_nginx --network my-bridge-net nginx
docker run --rm --name bridge_busybox --network my-bridge-net radial
/busyboxplus:curl curl bridge_nginx:80
Overlay
Create two services in the Docker Swarm cluster and demonstrate that they are able to communicate using a custom overlay network.
docker network create --driver overlay my-overlay-net
docker service create --name overlay_nginx --network my-overlay-net
nginx
docker service create --name overlay_busybox --network my-overlay-net
radial/busyboxplus:curl sh -c 'curl overlay_nginx:80 && sleep 3600'
docker service logs overlay_busybox
MACVLAN
Create a MACVLAN network. Then run two containers that are able to communicate using the MACVLAN network.
docker network create -d macvlan --subnet 192.168.0.0/24 --gateway
192.168.0.1 -o parent=eth0 my-macvlan-net
docker run -d --name macvlan_nginx --net my-macvlan-net nginx
docker run --rm --name macvlan_busybox --net my-macvlan-net radial
/busyboxplus:curl curl 192.168.0.2:80
None
Create a container that uses the none network and demonstrate that a normal container cannot reach it.
docker run --net none -d --name none_nginx nginx
docker run --rm radial/busyboxplus:curl curl none_nginx:80
Creating a Docker Bridge Network
You can create and manage your own networks with the docker network commands. If you do not specify a network driver,bridge will be used.
create a network
docker network create NETWORK
Run a new container, connecting it to the specified network.
docker run --network NETWORK
Connect a running container to an existing network
docker network connect NETWORK CONTAINER
Embedded DNS
Docker networks implement an embedded DNS server, allowing containers and services to locate and communicate with one another.
Containers can communicate with other containers and services using the service or container name or network alias.
Provide a network alias to a container
docker run –-network-alias ALIAS
docker network connect --alias ALIAS
Create a bridge network and demonstrate that two containers can communicate using the network.
docker network create my-net
docker run -d --name my-net-busybox --network my-net radial/busyboxplus:
curl sleep 3600
docker run -d --name my-net-nginx nginx
docker network connect my-net my-net-nginx
docker exec my-net-busybox curl my-net-nginx:80
Create a container with a network alias and communicate with it from another container using both the name and the alias.
docker run -d --name my-net-nginx2 --network my-net --network-alias mynginx-
alias nginx
docker exec my-net-busybox curl my-net-nginx2:80
docker exec my-net-busybox curl my-nginx-alias:80
Create a container and provide a network alias with the docker network connect command.
docker run -d --name my-net-nginx3 nginx
docker network connect --alias another-alias my-net my-net-nginx3
docker exec my-net-busybox curl another-alias:80
Manage existing networks on a Docker host.
docker network ls
docker network inspect my-net
docker network disconnect my-net my-net-nginx
docker network rm my-net
Deploying a Service on a Docker Overlay Network
You can create a network with the overlay driver to provide connectivity between services and containers in Docker Swarm.
By default,services are attached to a default overlay network called ingress. You can create your own networks to provide isolated
communication between services and containers.
Create an overlay network
docker network create --driver overlay NETWORK
Run a service attached to an existing overlay network
docker service create --network NETWORK
List the networks on the host. You should be able to see the default ingress overlay network.
docker network ls
Create an attachable overlay network.
docker network create --driver overlay --attachable my-overlay
Create a service that uses that network, then test the network by attaching a container to it and using the container to communicate with the
service.
docker service create --name overlay-service --network my-overlay --
replicas 3 nginx
docker run --rm --network my-overlay radial/busyboxplus:curl curl
overlay-service:80
Exposing Containers Externally
You can expose containers by publishing ports. This maps a port on the host(or all swarm hosts in the case of Docker swarm) to a port with in the
container.
Publish a port on the host,mapping it to a port on the container.
docker run -p HOST_PORT:CONTAINER_PORT
Publish a port on all swarm hosts,mapping it to a port on the service’s containers.
docker service create -p HOST_PRT:SERVICE_PORT
Host Vs. Ingress
Docker swarm supports two modes for publishing ports for services.
Ingress
The default used if no mode is specified.
Uses a routing mesh. The published port listens on every node in the cluster, and transparently directs incoming traffic to any task that is part of
the service, on any node.
Host
Publishes the port directly on the host where a task is running.
Cannot have multiple replicas on the same node if you use a static port.
Traffic to the published port on the node goes directly to the task running on that specific node.
Publish a service port using host mode
docker service create --publish mode=host, target=80, published=8080
Run a container with a published port, then test and examine the published ports for the container.
docker run -d -p 8080:80 --name nginx_pub nginx
curl localhost:8080
docker port nginx_pub
docker ps
Create a service with a published port using the ingress publishing mode. Test the service and check which Swarm node the service's task is
running on.
docker service create -p 8081:80 --name nginx_ingress_pub nginx
curl localhost:8081
docker service ps nginx_ingress_pub
Create a service published using host mode. Check which node the task is running on and attempt to access it from the node where the task is
running, as well as a node where it is not running.
docker service create -p mode=host,published=8082,target=80 --name
nginx_host_pub nginx
docker service ps nginx_host_pub
Check which node the task is running on, and access the service from that node.
curl localhost:8082
Try accessing the service from another node. It should fail.
curl localhost:8082
Network Troubleshooting
There are several ways you can gather information to troubleshoot networking issues.
Get container logs
docker logs CONTAINER
Get collated logs from the tasks of a service
docker service logs SERVICE
Get Docker Daemon logs.
journalctl -u docker
A great way to troubleshoot network issues is to run a container with in the context of a Docker network. You can use it to test connectivity and
gather information.
Netshoot is an image that comes with a variety of network troubleshooting tools.
You can run netshoot by using the container image nicolaka/netshoot
You can even run netshoot with in the networking namespace of an existing container!
Run a container and access the container logs.
docker run --name log-container busybox echo Here is my container log!
docker logs log-container
Run a service and access the service logs.
docker service create --name log-svc --replicas 3 -p 8080:80 nginx
curl localhost:8080
docker service logs log-svc
Check the Docker daemon logs.
sudo journalctl -u docker
Create a custom network with a container, and attach a netshoot container to the network for troubleshooting.
docker network create custom-net
docker run -d --name custom-net-nginx --network custom-net nginx
Inject a netshoot container into another container's network sandbox.
docker run --rm --network custom-net nicolaka/netshoot curl custom-netnginx:
80
docker run --rm --network container:custom-net-nginx nicolaka/netshoot
curl localhost:80
Inject a netshoot container into the sandbox of a container that uses the none network.
docker run -d --name none-net-nginx --network none nginx
docker run --rm --network container:none-net-nginx nicolaka/netshoot
curl localhost:80
Configuring Docker to Use External DNS
External DNS
You may need to customize the external DNS server(s) used by your containers.
You can change the default for the host with the dns setting in daemon.json
{
“dns”: [“8.8.8.8”]
}
Run a container with a custom external DNS
docker run --dns DNS_ADDRESS
Edit the Docker daemon config file to set a custom DNS for the host.
sudo vi /etc/docker/daemon.json
{
"dns": ["8.8.8.8"]
}
Restart Docker.
sudo systemctl restart docker
Test your DNS by looking up an external domain.
docker run nicolaka/netshoot nslookup google.com
Run a container with a custom DNS and test it by doing an nslookup.
docker run --dns 8.8.4.4 nicolaka/netshoot nslookup google.com
Signing Images and Enabling Docker Content Trust
Docker Content Trust(DCT) provides a secure way to verify the integrity of images before you pull or run them on your systems
With DCT, the image creator signs each image with a certificate, which clients can use to verify the image before running it.
docker trust key generate <signer nam>
Generate a delegation key pair. This gives users access to sign images for a repository.
docker trust signer add --key <key file> <signer name> <repo>
Enabling DCT
Docker Content Trust can be enabled by setting the DOCKER_CONTENT_TRUST environment variable to 1.
In Docker Enterprise Edition,you can also enable it in daemon.json
When DCT is enabled,Docker will only pull and run signed images. Attempting to pull and/or run an unsigned image will result in an error
message.
Note that when
DOCKER_CONTENT_TRUST=1, docker push will automatically sign the image before pushing it.
a Docker Hub account is required. An account can be created for free at https://hub.docker.com.
First, log in to Docker Hub. Enter your Docker Hub credentials when prompted.
docker login
Generate a delegation key pair. We can enter a passphrase of our choosing, but make note of it as we will need it later on in the lesson.
cd ~/
docker trust key generate <your docker hub username>
Then we'll add ourselves as a signer to an image repository. Once again, be sure to make note of the passphrases used.
docker trust signer add --key <your docker hub username>.pub <your docker hub username> <your docker hub
username>/dct-test
Create and build a simple Docker image with an unsigned tag, and then push it to Docker Hub:
mkdir ~/dct-test
cd dct-test
vi Dockerfile
FROM busybox:latest
CMD echo It worked!
docker build -t <your docker hub username>/dct-test:unsigned .
docker push <your docker hub username>/dct-test:unsigned
Run the image to verify whether it can run successfully:
docker run <your docker hub username>/dct-test:unsigned
Next, enable Docker content trust and attempt to run the unsigned image again:
Note: We should see it fail.
export DOCKER_CONTENT_TRUST=1
docker run <your docker hub username>/dct-test:unsigned
Build and push a signed tag to the repo. Enter the passphrase — this will be the one that was chosen earlier when running the docker trust
key generate command:
docker build -t <your docker hub username>/dct-test:signed .
docker trust sign <your docker hub username>/dct-test:signed
Run it to verify that the signed image can run properly with Docker Content Trust enabled:
docker image rm <your docker hub username>/dct-test:signed .
docker run <your docker hub username>/dct-test:signed
Turn off Docker Content Trust and attempt to run the unsigned image again:
Note: It should work this time.
export DOCKER_CONTENT_TRUST=0
docker run <your docker hub username>/dct-test:unsigned
Default Docker Engine Security
Namespaces and Cgroups
Namespaces and Control Groups(cgroups) provide isolation to containers.
Isolation means that container processes cannot see or affect other containers or processes running directly on the host system.
This limits the impact of certain exploits or privilege escalation attacks. If one container is compromised, it is less likely that it can be used to gain
any further access outside the container.
Docker Daemon Attack Surface
It is important to note that the Docker daemon itself requires root privileges. Therefore, you should be aware of the potential attack surface
presented by the Docker daemon.
Only allow trusted users to access the daemon. Control of the Docker daemon could allow the entire host to be compromised.
Be aware of this if you are building any automation that accesses the Docker daemon, or granting any user direct access to it.
Linux Kernel Capabilities
Docker uses capabilities to fine-tune what container processes can access.
This means that a process can run as root inside a container, but does not have access to do everything root could normally do on the host.
For example, Docker uses the net_build_service capability to allow container processes to bind to a port below 1024 without running as root.
Docker MTLS
Encrypting Overlay Networks
In a distributed model, such as the one used by Docker Swarm, it is important to encrypt communication between nodes to prevent potential
attackers from obtaining sensitive data from network communications. We'll discuss the two ways that Docker Swarm utilizes for securing cluster
communication. We'll also cover how to encrypt overlay network communication to secure the communication between containers within the
cluster, and we'll discuss how Docker Swarm uses Mutually Authenticated Transport Layer Security (MTLS) to encrypt and authenticate clusterlevel
communication.
You can encrypt communication between containers on overlay networks in order to provide greater security within your swarm cluster.
Use the --opt encrypted flag when creating an overlay network to encrypt it.
docker network create --opt encrypted --driver overlay NETWORK
MTLS in Docker Swarm
Docker Swarm provides additional security by encrypting communication between various components in the cluster.
Mutually Authenticated Transport Layer Security(MTLS)
Both participants in communication exchange certificates and all communication is authenticated and encrypted.
When a swarm is initialized, a root certificate authority(CA) is created, which is used to generate certificates for all nodes as they join the cluster.
Worker and Manager tokens are generated using the CA and are used to join new nodes to the cluster.
Used for all cluster-level communication between swarm nodes.
Enabled by default, you don’t need to do anything to set it up.
Create an encrypted overlay network:
docker network create --opt encrypted --driver overlay my-encrypted-net
Create two services on the encrypted overlay network and demonstrate that one service can communicate with the other:
docker service create --name encrypted-overlay-nginx --network myencrypted-
net --replicas 3 nginx
docker service create --name encrypted-overlay-busybox --network myencrypted-
net radial/busyboxplus:curl sh -c 'curl encrypted-overlaynginx:
80 && sleep 3600'
Check the logs for the busybox service, and then verify that it shows the Nginx welcome page:
docker service logs encrypted-overlay-busybox
Securing the Docker Daemon HTTP Socket
Docker uses a socket that is not exposed to the network by default.
However,you can configure Docker to listen on an HTTP port,which you can connect to in order to remotely manage the daemon.
In order to do this securely, we need to:
Create a certificate authority.
Create server and client certificates.
Configure the daemon on the server to use tlsverify mode.
Configure the client to connect securely using the client certificate.
Follow along with this lesson using two playground servers:
Image - Ubuntu 18.04 Bionic Beaver LTS
Size - Micro
Generate a certificate authority and server certificates for your Docker server. Make sure you replace <server private IP> with the actual
private IP of your server.
Note: If you get a message that says Can't load /home/cloud_user/.rnd into RNG, it is safe to ignore that message. The command
will still succeed.
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem -
subj "/C=US/ST=Texas/L=Keller/O=Linux Academy/OU=Content/CN=$HOSTNAME"
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=$HOSTNAME" -sha256 -new -key server-key.pem -out
server.csr
echo subjectAltName = DNS:$HOSTNAME,IP:<server private IP>,IP:127.0.0.1
>> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey cakey.
pem \
-CAcreateserial -out server-cert.pem -extfile extfile.cnf
Then generate the client certificates:
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile-client.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey cakey.
pem \
-CAcreateserial -out cert.pem -extfile extfile-client.cnf
Set appropriate permissions on the certificate files:
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
Configure your Docker host to use tlsverify mode with the certificates that were created earlier:
sudo vi /etc/docker/daemon.json
{
"tlsverify": true,
"tlscacert": "/home/cloud_user/ca.pem",
"tlscert": "/home/cloud_user/server-cert.pem",
"tlskey": "/home/cloud_user/server-key.pem"
}
sudo vi /lib/systemd/system/docker.service
Look for the line that begins with ExecStart and change the -H so that it looks like this:
ExecStart=/usr/bin/dockerd -H=0.0.0.0:2376 --containerd=/run/containerd/containerd.sock
sudo systemctl daemon-reload
sudo systemctl restart docker
Copy the CA cert and client certificate files to the client machine:
scp ca.pem cert.pem key.pem cloud_user@<client private IP>:/home/cloud_user
On the client machine, configure the client to securely connect to the remote Docker daemon:
mkdir -pv ~/.docker
cp -v {ca,cert,key}.pem ~/.docker
export DOCKER_HOST=tcp://<docker server private IP>:2376
DOCKER_TLS_VERIFY=1
Test the connection:
docker version
Installing Docker EE
UCP Manager
Distribution: Ubuntu 18.04 Bionic Neaver LTS
Size: Large
UCP Worker
Distribution: Ubuntu 18.04 Bionic Neaver LTS
Size: Small
UCP Worker with DTR
Distribution: Ubuntu 18.04 Bionic Neaver LTS
Size: Medium
Log in to each node and enable passwordless sudo for cloud_user.
sudo visudo
At the bottom of the file, add:
cloud_user ALL=(ALL) NOPASSWD: ALL
Log in to the manager node and generate a key pair, then copy it to all three nodes to allow cloud_user to log in using the key.
ssh-keygen -t rsa
ssh-copy-id cloud_user@<manager private IP>
ssh-copy-id cloud_user@<worker private IP>
ssh-copy-id cloud_user@<dtr private IP>
Download and set up Mirantis launchpad.
wget https://github.com/Mirantis/launchpad/releases/download/0.14.0
/launchpad-linux-x64
mv launchpad-linux-x64 launchpad
chmod +x launchpad
Verify that you can execute launchpad.
./launchpad version
Register your Docker EE cluster.
./launchpad register
Create a cluster configuration file.
vi cluster.yaml
apiVersion: launchpad.mirantis.com/v1beta3
kind: DockerEnterprise
metadata:
name: launchpad-ucp
spec:
ucp:
version: 3.3.2
installFlags:
- --admin-username=admin
- --admin-password=secur1ty!
- --default-node-orchestrator=kubernetes
hosts:
- address: <manager private IP>
privateInterface: ens5
role: manager
ssh:
user: cloud_user
keyPath: ~/.ssh/id_rsa
- address: <worker private IP>
privateInterface: ens5
role: worker
ssh:
user: cloud_user
keyPath: ~/.ssh/id_rsa
./launchpad apply
You can access the UCP interface in a web browser at https://<UCP server public IP>. Use the credentials specified in cluster.yaml
(in the ucp section) to log in.
When you are asked for a license, select skip for now.
Universal Control Plane
Docker Universal Control Plane(UCP) provides enterprise-level cluster management.
At first glance, UCP may look like “Docker Swarm with a web interface,” but it provides additional features as well, such as:
Organization and team management
Role-based access control
Orchestration with both Docker Swarm and Kubernetes
Once the installation is complete, access UCP in a web browser using the UCP manager's Public IP address: https://<your UCP manager
public IP>.
Log in to the UCP manager using the credentials you supplied earlier in cluster.yaml.
You may be prompted to provide a license. Select Skip for now.
You should see the UCP Dashboard.
Let's run a Docker swarm service using UCP.
Click on Swarm > Services on the left.
Click Create, then enter nginx as a name for your service. For the image, enter nginx.
Click Create. Note that it may take a few moments for your new service to spin up.
Go to Shared Resources > Nodes. Select the gear icon to configure the node. Note the Orchestrator Type setting. This determine which
orchestrators are used on which node.
Security in UCP
UCP provides a flexible model for controlling access to cluster resources and functionality.
UCP offers a variety of features to help you secure your Docker cluster. One of these is a role-based access control system designed to help you
manage permissions in your cluster. We’ll explore role-based access control in UCP.
User A person who is authenticated
Team A group of users that share certain permissions.
Organization A group of teams that share certain permissions
Service Account A kubernetes object that allows a container to access cluster resources.
Subject A user, team organization, or service account that has the ability to do something
Resource set A docker swarm collection of cluster objects, like containers, services, and nodes
Role A permission that can be used to operate on objects in a collection
Grant Provides a specific permission(role) to a subject, with regard to a resource set
UCP also support LDAP integration, allowing you to manage users and teams using an LDAP-enabled user directory.
On the left, select Access Control.
Select Users to create an manage users.
Create a new user with the username bob.
Manage Organizations and Teams from the Orgs & Teams panel.
Manage Docker Swarm resource sets by going to Shared Resources > Collections.
Select View Children next to the Swarm collection. Create a new collection by clicking Create Collection. Give it the name mycollection
.
Select Swarm > Services, then click on your nginx service.
Click the gear icon to configure the service. Select Collection, pick View Children for the Swarm collection, then use the Select
Collection button that appears when hovering over mycollection. Click Save. This will add the service to your collection.
Click Access Control > Roles to explore the roles that are available. Use the Kubernetes and Swarm to view roles for Kubernetes and
Docker Swarm, respectively.
Go to Access Control > Grants, then select Swarm at the top. Click Create Grant.
Select the bob user for the Subject. For the Resource Set, select View Children next to Swarm, then choose Select Collection by
hovering over mycollection. For the Role, pick View Only, then click Create.
Congratulations! You just granted the Bob user the ability to view information about your nginx service.
Feel free to further explore the UCP access control features.
Also, check out the LDAP integration settings.
On the left, select admin > Admin Settings > Authentication & Authorization, then click the LDAP toggle.
Docker Trusted Registry(DTR)
Docker Trusted Registry (DTR) is an enhanced, enterprise-ready private Docker registry.
Additional features:
A web UI
High availability through multiple registry nodes
Cache images geographically close to users
Role-based access control
Security vulnerability scanning for images
DTR High Availability
Docker Trusted Registry supports high availability by allowing you to run multiple DTR replicas in a cluster.
A DTR cluster can tolerate failures as long as there is a quorum.
Quorum - More than half of the replicas are available.
DTR Cache
DTR Caches allow you to cache images geographically close to your users, allowing for faster download speeds.
You should know:
Each cache pulls the image from the main DTR the first time a user requests the image from that cache.
Users still authenticate using the main DTR URL.
Users request images from the main DTR URL, and DTR responds with a redirect to the cache.
A user profile setting determines which cache a user will pull images from.
This means authentication and image pulling are completely transparent to users; they do not need to think about caches.
Setting up docker Trusted Registry(DTR)
Log in to your UCP manager server.
Generate certificates for your DTR instance. Begin by creating a certificate authority.
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -subj "
/OU=dtr/CN=DTR CA" -out ca.crt
Create the certificate key and certificate signing request.
openssl genrsa -out dtr.key 2048
openssl req -new -sha256 -key dtr.key -subj "/OU=dtr/CN=system:dtr" -
out dtr.csr
Create a file containing your certificate extension data. Be sure to supply the hostname and private IP address of your DTR server.
vi extfile.cnf
keyUsage = critical, digitalSignature, keyEncipherment
basicConstraints = critical, CA:FALSE
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = DNS:<DTR server hostname>,IP:<DTR server private IP>,
IP:127.0.0.1
Create the public certificate.
openssl x509 -req -in dtr.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dtr.crt -days 365 -sha256 -
extfile extfile.cnf
Edit your cluster.yaml (you should have created one earlier when initially setting up the cluster).
vi cluster.yaml
Add the dtr section to the file, as well as your third server with the role of dtr. You should be able to leave the other sections the same.
Note: For the certificate data, copy in the contents of each respective certificate file. Ensure that the indentation is correct for the yaml format.
Each new line of the certificate data should be indented equal to the |- at the top of each certificate section.
apiVersion: launchpad.mirantis.com/v1beta3
kind: DockerEnterprise
metadata:
name: launchpad-ucp
spec:
...
dtr:
version: 2.8.2
installFlags:
- --ucp-insecure-tls
- |-
--dtr-cert "<contents of dtr.crt>"
- |-
--dtr-key "<contents of dtr.key>"
- |-
--dtr-ca "<contents of ca.crt>"
hosts:
...
1.
2.
3.
4.
- address: <DTR server private IP>
privateInterface: ens5
role: dtr
ssh:
user: cloud_user
keyPath: ~/.ssh/id_rsa
Re-apply the cluster configuration to set up the third server and install DTR.
./launchpad apply
You should be able to access DTR in a browser at https://<DTR public IP>. You can use your UCP admin credentials to authenticate.
When you are asked for a license, select skip for now.
Sizing Requirements for Docker UCP and DTR
Docker
Sizing requirements for Docker depend on the container workloads you will be running.
UCP
Minimum 8 GB memory and 2 CPUs for manager nodes
Recommended 16 GB memory and 4 CPUs for manger nodes
Minimum 4 GB memory for worker nodes
DTR
Minimum 16 GB memory, 2 CPUs,10 GB disk
Recommended 16 GB memory,4 CPUs,25-100 GB disk
Configuring Backups for UCP and DTR
In a production environment, it is important to regularly back up your UCP infrastructure so that you can quickly recover in the event of data loss.
The basic steps for backing up your UCP and DTR infrastructure are:
Back up the Docker swarm. (Note: This is done the same way for a UCP swarm as it is for a regular, non-UCP swarm.)
Back up UCP.
Back up DTR images.
Back up DTR Metadata.
Back up UCP
Log in to the UCP manager server.
Back up the UCP data.
sudo docker container run \
--rm \
--log-driver none \
--name ucp \
--volume /var/run/docker.sock:/var/run/docker.sock \
--volume /tmp:/backup \
mirantis/ucp:3.3.2 backup \
--file ucp_backup.tar \
--passphrase "mysecretphrase" \
--include-logs=false
List the contents of the backup file to verify that it worked. You will need to enter your passphrase.
gpg --decrypt /tmp/ucp_backup.tar | tar --list
Back up DTR
Log in to the DTR server.
Get the DTR replica ID.
REPLICA_ID=$(sudo docker ps --format '{{.Names}}' -f name=dtr-rethink | cut -f 3 -d '-') && echo
$REPLICA_ID
Back up the registry images.
sudo tar -cvf dtr-backup-images.tar /var/lib/docker/volumes/dtr-registry-$REPLICA_ID
Verify the image backup file's contents.
tar -tf dtr-backup-images.tar
Back up DTR metadata.
UCP_PRIVATE_IP=<UCP Manager Private IP>
read -sp 'ucp password: ' UCP_PASSWORD; \
sudo docker run --log-driver none -i --rm \
--env UCP_PASSWORD=$UCP_PASSWORD \
mirantis/dtr:2.8.2 backup \
--ucp-url https://$UCP_PRIVATE_IP \
--ucp-insecure-tls \
--ucp-username admin \
--existing-replica-id $REPLICA_ID > dtr-backup-metadata.tar
Verify the backup file's contents.
tar -tf dtr-backup-metadata.tar
DTR Security Features
Security Scanning with DTR
Docker Trusted Registry has the ability to scan images for security vulnerabilities.
You can enable security scanning via the DTR web UI.
Immutable Tags
You can set Immutability on a repository at any time. This prevents users from overwriting existing tags.
In DTR, navigate to System > Security.
Here, you can enable vulnerability scanning, but only if you have the appropriate license.
You can also set vulnerability scanning settings for each repository, but you need a license for scanning to take place.
While creating a new repository, click Show advanced settings. Here, you can enable or disable immutability. You can also do this after
creating a repository.
Managing Certificates with UCP and DTR
External Certificates
Your UCP infrastructure generates its own self-signed certificates by default, but you can also supply your own certificates.
You can apply your own certificates both UCP and DTR using their web interfaces!
Client Bundles
Client bundles provide a pre-packaged certificate configuration for clients.
They include client certificates which allow you to authenticate with UCP from the Docker command line.
To upload your own certificates in UCP, go to admin > Admin Settings > Certificates.
In DTR, click System >General, then under Domain & Proxies select Show TLS settings.
To generate client bundles in UCP, go to admin > My Profile > New Client Bundle.
Kubernetes Orchestration in Docker
Docker Kubernetes Service
Docker Kubernetes Service allows us to orchestrate container workloads in UCP using kubernetes. This combines the UCP interface with the
flexibility of a kubernetes cluster.
Orchestrator Type
Each UCP node has an orchestrator type, whcih determines whether the node will run workloads managed by Docker Swarm or Kubernetes.
Docker Swarm - The node will run Docker Swarm-managed workloads.
Kubernetes - The node will run kubernetes-managed workloads.
Mixed - The node will run workloads managed by both Docker Swarm and kubernetes.
Note that this mode is not recommended for use in production.
Namespaces
Every kubernetes object exists in a namespace.
Namespaces allow you to organize your obejcts and keep them seperate between different teams or applications.
When no namespace is specified in kubernetes, a namespace called default will be assumed.
creating objects
You can create kubernetes objects using YAML in the UCP interface.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
Navigate to Shared Resources > Nodes, then select one of our UCP worker nodes.
Click the gear icon to edit the node.
Take note of the Orchestrator Type option, which allows us to change whether the node will run workloads for Docker Swarm, Kubernetes, or
both.
Navigate to Kubernetes > Namespaces.
Click the Create button.
Create a new namespace:
apiVersion: v1
kind: Namespace
metadata:
name: my-namespace
Click Create to create the Namespace. The new Namespace will now appear in the Namespaces list.
Select Set Context for my-namespace.
Navigate to Kubernetes > + Create.
Select the my-namespace Namespace from the dropdown.
Create a pod with the following YAML:
apiVersion: v1
kind: Pod
metadata:
name: my-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80
Click Create to create the pod. It should soon enter the Ready status.
Application Configuration in Kubernetes
application Configuration is all about managing configuration data and passing that data to applications.
Kubernetes uses ConfigMaps and Secrets to store configuration data and pass it to containers.
Configmaps allow you to store configuration data in a key value format. This data can include simple values or larger blocks of data such as
configuration files.
Secrets are similar to ConfigMaps, but their data is encrypted at rest.Use secrets to store sensitive data, such as passwords or API tokens.
Using Config Data
configuration data stored in Configmaps or Secrets can be passed to our containers in two ways:
Environment variables - Configuration values can be passed in as environment variables that will be visible to the container process at runtime.
Volume mounts - configuration data can be mounted to the container file system, where it will appear in the form of files.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Create a ConfigMap:
apiVersion: v1
kind: ConfigMap
metadata:
name: my-configmap
data:
key1: important data
key2.txt: |
Another value
multiple lines
more lines
Create a secret:
apiVersion: v1
kind: Secret
metadata:
name: my-secret
type: Opaque
data:
username: dXNlcgo=
password: ZG9ja2VyCg==
Create a pod that uses the configuration data from the ConfigMap and secret:
apiVersion: v1
kind: Pod
metadata:
name: my-configured-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ["sh", "-c", "echo \"ConfigMap environment variable:
$CONFIGMAPVAR\" && echo \"Secret environment variable: $SECRETVAR\" &&
echo \"The ConfigMap mounted file is: $(cat /etc/configmap/key2.txt)\"
&& echo \"The Password is:\" $(cat /etc/secret/password)"]
env:
- name: CONFIGMAPVAR
valueFrom:
configMapKeyRef:
name: my-configmap
key: key1
- name: SECRETVAR
valueFrom:
secretKeyRef:
name: my-secret
key: username
volumeMounts:
- name: configmap-vol
mountPath: /etc/configmap
- name: secret-vol
mountPath: /etc/secret
volumes:
- name: configmap-vol
configMap:
name: my-configmap
- name: secret-vol
secret:
secretName: my-secret
Click on the my-configured-pod pod.
Click the Log icon near the upper right.
We should see our configuration data printed to the log, both from the ConfigMap and the secret, like so:
ConfigMap environment variable: important data
Secret environment variable: user
The ConfigMap mounted file is:Another value
multiple lines
more lines
The Password is: docker
Kubernetes Network Model
The kubernetes network model is a set of standards that define how networking between pods behaves.
The Kubernetes network model defines how pods communicate with each other, regardless of which node they are running on.
Each pod has its own unique IP address with in the cluster
Any pod can reach any other pod using that pod’s IP address. This creates a virtual network that allows pods to easily communicate with each
other, regardless of which node they are on.
Services and DNS
Services
Services allow us to expose an application that is running as a set of pods.
A service exposes a set of pods, load-balancing client traffic across them. These pods can then be dynamically created and destroyed without
interruption to clients using the service.
Services Types
Each service has a type. The type determines the behavior of the service and what it can be used for.
ClusterIp(the default) - Exposes to pods within the cluster using a unique IP on the cluster’s virtual network.
NodePort - Exposes the service outside the cluster on each Kubernetes node, using a statis port.
Loadbalancer - Expose the service externally using a cloud provider’s load balancing functionality.
ExternalName - Expose an external resource inside the cluster network using DNS.
Cluster DNS
Kubernetes includes a cluster DNS that helps containers locate Services using domain names.
Pods within the same namespace as a service can simply use the service name as a domain name.
pods in a different namespace must use the full domain name, which takes the form:
my-svc.my-namespace.svc.cluster-domain.example
Services are a great way to expose applications to one another (or to the outside world) within a Kubernetes cluster.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Creating Services
Make sure to view the default Namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a pod which will be exposed using a service.
apiVersion: v1
kind: Pod
metadata:
name: my-svc-pod
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80
Create a ClusterIP service.
apiVersion: v1
kind: Service
metadata:
name: nginx-internal-service
spec:
type: ClusterIP
selector:
app: nginx
ports:
- protocol: TCP
port: 80
targetPort: 80
Create a busybox pod in the same namespace as our services:
apiVersion: v1
kind: Pod
metadata:
name: busybox-dns
namespace: default
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ["sh", "-c", "while true; do sleep 3600; done"]
Navigate to Kubernetes > Services > nginx-internal-service, copy the Cluster IP.
Navigate to Kubernetes > Pods > busybox-dns.
Click the Exec icon near the upper-right and enter sh to access a shell prompt within the container.
Make a request to the service:
curl <nginx-internal-service Cluster IP>
We should see some HTML representing the Nginx welcome page.
Exploring Kubernetes Service DNS
Look up DNS records for the ClusterIP Service.
nslookup nginx-internal-service
Make a request to the service using the service name.
curl nginx-internal-service
Make a request using the full domain name.
curl nginx-internal-service.default.svc.cluster.local
Switch to the my-namespace Namespace (Kubernetes > Namespaces, then click Set Context next to my-namespace).
Create a busybox Pod in the my-namespace Namespace:
apiVersion: v1
kind: Pod
metadata:
name: busybox-dns-other-namespace
namespace: my-namespace
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ["sh", "-c", "while true; do sleep 3600; done"]
Navigate to Kubernetes > Pods > busybox-dns-other-namespace.
Click the Exec icon near the upper-right and enter sh to access a shell prompt within the container.
Make a request to the service using the service name. This will fail since the short name can only be used when using a service within the same
namespace as the pod.
curl nginx-internal-service
Make a request using the full domain name.
curl nginx-internal-service.default.svc.cluster.local
Deployments
Deployments allow you to manage changes to a set of replica pods. They specify a desired state for a set of pods, and work to achieve and
maintain that state, even if it is changed.
You can use deployments to do things like rolling updates and scaling.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default Namespace (Kubernetes > Namespaces, then click Set Context next to default).
Creating Deployments
Create a deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
spec:
replicas: 3
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80
Wait for the deployment status to turn green. Note that the deployment automatically creates an accompanying ReplicaSet.
Navigate to Kubernetes > Pods. Note that the Deployment automatically created three pods.
Scaling
Navigate to Kubernetes > Controllers > nginx-deployment. Click the gear icon to edit the deployment.
Change the replicas value to 5 and click Save. This will scale up the deployment to five replicas. After a few moments, two additional pods will be
automatically created.
Rolling Updates
Navigate to Kubernetes > Controllers > nginx-deployment. Click the gear icon to edit the Deployment.
Change the image for the container to nginx:1.19.2 and click Save.
Quickly navigate to Kubernetes > Controllers.
We should be able to watch the rolling update process take place here. We will notice a new ReplicaSet. Watch the statuses of the two
ReplicaSets. We will see the new ReplicaSet gradually replace the old one, with the old ReplicaSet removing its pods as the new ReplicaSet's
pods become available.
DaemonSets
DaemonSets are a tool that allows you to run a replica pod dynamically on each node in the cluster.
They will run a replica on each node and create new replicas on new nodes as they are added.
DaemonSets in Kubernetes offer the ability to run container replicas on each cluster node.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a DaemonSet.
apiVersion: apps/v1
kind: DaemonSet
metadata:
name: my-daemonset
spec:
selector:
matchLabels:
app: my-ds
template:
metadata:
labels:
app: my-ds
spec:
containers:
- name: busybox
image: busybox
command: ["sh", "-c", "while true; do sleep 10; done"]
Watch as the DaemonSet spins up its pods. The number of pods on the DaemonSet should match the number of Kubernetes nodes in the cluster.
Navigate to Kubernetes > Pods.
We should see our DaemonSet pods listed. Note which node each pod is running on. There should be exactly one replica for the DaemonSet on
each node.
Scheduling
In kubernetes, scheduling refers to the process of selecting a node on which to run a workload.
Scheduling is handled by the kubernetes Scheduler.
Node Taints
Node taints control which pods will be allowed to run on which nodes. Pods can have tolerations, which override taints for the specific pod.
Each taint has an effect.
The NoExecute effect does two things to pods that do not have the appropriate toleration:
Prevents new pods from being scheduled on the node.
Evicts existing pods.
Resource Requests
In the container spec, you can specify resource requests for resources such as memory and CPU.
The scheduler will avoid scheduling pods on nodes that do not have enough available resources to satisfy the requests.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a pod with a resource request that is too large to be satisfied
apiVersion: v1
kind: Pod
metadata:
name: busybox-rr-unreasonable
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ["sh", "-c", "while true; do sleep 3600; done"]
resources:
requests:
memory: 64Mi
cpu: 10000m
The pod will remain in the Pending state since it cannot be scheduled due to the large resource request.
Create a pod with a more reasonable resource request.
apiVersion: v1
kind: Pod
metadata:
name: busybox-rr-reasonable
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ["sh", "-c", "while true; do sleep 3600; done"]
resources:
requests:
memory: 64Mi
cpu: 250m
This pod will be scheduled successfully.
Probes
Probes allow you to customize how the kubernetes cluster determines the state of each container.
Liveness Probes check whether the container is healthy.
Readiness Probes check whether the container is ready to service user requests.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a pod with a liveness probe and a readiness probe.
apiVersion: v1
kind: Pod
metadata:
name: probe-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80
livenessProbe:
httpGet:
path: /
port: 80
initialDelaySeconds: 3
periodSeconds: 3
readinessProbe:
httpGet:
path: /
port: 80
initialDelaySeconds: 3
periodSeconds: 3
Verify that the pod starts up successfully.
Storage with Volumes
Kubernetes uses volumes to provide simple storage to containers. Volume storage is more persistent than the container file system and can
remain even if a container is restarted or re-created.
Volumes are part of the pod spec, and the same volume can be mounted to multiple containers.
Verify that the pod starts up successfully.
Volume Types
A volume’s type determines how the underlying storage is handled. There are many volume types available.
Some volume types to know:
nfs - Shared network storage is provided using nfs.
hostPath - Files are stored in a directory on the worker node. The same files will not appear if the pod runs on another node.
emptyDir - Temporary storage that is deleted if the pod is deleted of the pod is deleted. Useful for sharing simple storage between containers in
a pod.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a pod that uses a hostPath volume.
apiVersion: v1
kind: Pod
metadata:
name: volume-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ["sh", "-c", "echo \"Successful output!\" > /output/output.
txt"]
volumeMounts:
- name: host-vol
mountPath: /output
volumes:
- name: host-vol
hostPath:
path: /tmp/output
Look at the pod list and check which node the volume-pod pod is running on. Log in to that node via SSH.
Check the output location on the node.
cat /tmp/output/output.txt
We should see the text Successful output!.
Storage with Persistent Volumes
Persistent Volumes allow you to manage storage resources in an abstract fashion. They let you define a set of available storage resources as a
kubernetes object, then later claim those storage resources for use in your pods.
A PersistentVolumeClaim defines a request for storage resources that can be mounted to a pod’s containers.
Reclaim Policy
A PersistentVolume has a reclaim policy which determines what happens when the associated claims are deleted.
Retain - Keeps the volume and its data and allows manual reclaiming. Admin is responsible for cleaning up existing data.
Delete - Deletes both the PersistentVolume and its underlying storage infrastructure (such as a cloud storage object).
Recycle - Deletes the data contained in the volume and allows it to be used again.
Expanding a PVC
You can expand a PersistentVolumeClaim by simply editing the object to have a larger size.
For this to work, the storage class must support volume expansion. If this is the case, the storage class’s allowVolumeExpansion field will be
set to true.
Access UCP in a browser at https://<UCP_SERVER_PUBLIC_IP>.
We can create Kubernetes objects in UCP by navigating to Kubernetes > + Create. Make sure to use the default namespace throughout this
exercise.
Make sure to view the default namespace (Kubernetes > Namespaces, then click Set Context next to default).
Create a StorageClass and PersistentVolume
Create a StorageClass that supports volume expansion.
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
Create a PersistentVolume.
kind: PersistentVolume
apiVersion: v1
metadata:
name: my-pv
spec:
storageClassName: localdisk
persistentVolumeReclaimPolicy: Recycle
capacity:
storage: 1Gi
accessModes:
- ReadWriteOnce
hostPath:
path: /tmp/pvoutput
Create a PersistentVolumeClaim.
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: my-pvc
spec:
storageClassName: localdisk
accessModes:
- ReadWriteOnce
“
resources:
requests:
storage: 100Mi
Check the claim's status to make sure it is Bound to the PersistentVolume.
Create the Pod
Create a pod that uses the PersistentVolumeClaim and writes some data to it.
apiVersion: v1
kind: Pod
metadata:
name: pvc-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do echo "Successfully written to
log." >> /output/output.log; sleep 10; done']
volumeMounts:
- name: pv-storage
mountPath: /output
volumes:
- name: pv-storage
persistentVolumeClaim:
claimName: my-pvc
Verify the pod can start up.
Take note of which node the pod is running on and log in to that node via SSH.
Check the output data from the pod.
ls /tmp/pvoutput
cat /tmp/pvoutput/output.log
Recycle the Persistent Storage
In the UCP interface, navigate to Kubernetes > Pods, and use the three-dot menu for the pvc-pod pod to delete the pod.
Note: Force Remove is faster and is okay to use here since we are just testing and do not need the pod to shut down
gracefully. ”
Navigate to Kubernetes > Storage, and use the three-dot menu for the my-pvc PersistentVolumeClaim to delete the PersistentVolumeClaim.
We should see the status for the my-pv PersistentVolume return to Available.
Create a new PersistentVolumeClaim.
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: my-new-pvc
spec:
storageClassName: localdisk
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 100Mi
We should see both the PersistentVolume and PersistentVolumeClaim return their status to Bound.
Expand the PersistentVolumeClaim
Navigate to Kubernetes > Storage > my-new-pvc. Click the gear icon to edit the PersistentVolumeClaim.
Find the line that says storage: 100Mi and change it to increase the claim size:
spec:
...
resources:
requests:
storage: 200Mi
Click Save.
  
