---
layout: post
title:  "Docker technical notes - basic"
date:   2020-01-24 00:00:00 -0500
categories: tech-docker-container
---

https://blog.csdn.net/DreamSeeker_1314/article/details/84403166

## Docker index vs docker Registry vs docker Repository

### docker index  

https://index.docker.io, https://hub.docker.com/, http://dockerhub.com  
login account: see dev-accounts.md

### Docker Registry

Docker Registry (Docker Trusted Registry – DTR) is an enterprise-grade storage solution for Docker images. 

existing and well-established cloud registries like Docker Hub, Quay, Google Container Registry, Amazon Elastic Container Registry.

### Docker Repository
Docker Repository is a collection of Docker images with the same name and different tags.

repositoryName format: <namespace, usually userName>/&lt;imageName>:Tag   
If Tag is not specified, Docker will apply the :latest tag to it.

	docker login	// login to dockerhub.com
	docker push userName/imageName:tag
	docker pull userName/imageName:tag	
	docker search <imageName or repositoryName>	

docker registry is similar to git, having remote registry & local registry .

	
## install docker on windows MobylinuxVM
Steps:
- set Intel Virtualization Tech enabled in BIOS 
		To access your BIOS on a Windows 10 PC, you must follow these steps.
		Navigate to settings. ...
		Select Update & security.
		Select Recovery from the left menu.
		Click Restart Now under Advanced startup. ...
		Click Troubleshoot.
		Click Advanced options.
		Select UEFI Firmware Settings.
		Click Restart.
		when pc restarts, it will enter BIOS set up -> security -> Intel Virtualization Tech, both enabled.
- Windows settings -> turn on/off windows features -> virtual machine managers: Hyper-V manager app ON
- go to https://cloud.docker.com/, download "Docker for Windows Installer.exe", 
- run "Docker for Windows Installer.exe" with administrator. select Linux container type during installation.
- start up docker VM with administrator, verify or change settings. 
		first time start up docker VM, it will auto create a MobyLinuxVM VM instance using hyper-V driver based on docker VM settings.
		docker vm user account: see dev-accounts.md
- start up Hyper-V manager, verify settings.
- restart PC. It will add path C:\Program Files\Docker\Docker\resources\bin

### config and start docker daemon

docker-for-desktop Factory setting: docker-for-desktop use dockerNAT network. the windows host IP is 10.0.72.1, the MobylinuxVM IP is 10.0.72.2

Shared drives require port 445 to be open between the host machine and the virtual machine that runs Linux containers.  
To share the drive C:/, allow connections between the Windows host machine and the virtual machine in Windows Firewall or your third party firewall software (Norton)
.
config Norton firewall: click Norton -> settings -> firewall -> networking -> devices -> config -> add, add a device with "Docker, 10.0.72.2, FullTrust".

click cube whale icon to start up docker daemon. It will also start up MobylinuxVM to run on.
right click docker icon -> settings -> configs, share C: drive, do NOT use kubernetes included in docker-for-desktop.
restart docker

docker client (CLI commands) connect to docker daemon with TCP socket. docker client sends request to docker daemon, and receives response.


## install docker on Ubuntu of Windows 
using Get Docker Script to perform the installation. This script is meant for quick & easy install	

	sudo apt-get install -y  linux-image-extra-virtual
	sudo curl -sSL https://get.docker.com/ | sh	
	
Problem:  
not able to start up docker deamon after installation. error "Your kernel does not support cgroup memory limit". give up using docker on Ubuntu on windows. 
  
## docker engine - docker engine is a group of static libs
  
Extend Docker engine libs with plugins

Install the vieux/sshfs plugin:
	docker plugin install --grant-all-permissions vieux/sshfs

  
## docker daemon   

### Linux
The docker daemon always runs as the root user.
The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can only access it using sudo. 	

###
docker daemon is a Linux or WIN process, which runs from installed docker libs.
check whether a docker daemon is running: 
	docker version
	docker info

### start/stop docker daemon   
start on Linux : 
	dockerd 	//	to start up a docker daemon, ctrl-c to stop docker daemon.
	OR systemctl start|stop|restart docker
	OR sudo service docker start|stop|restart
	
start on Windows: 
click the cute whale icon to start a docker daemon. Docker Menu on the right side of the Windows Taskbar -> restart.
docker -> settings -> Network -> default Internal Virtual Switch : 10.0.75.0, this is the docker default network interface. 


### Docker daemon configs location:
Linux: /etc/docker/daemon.json
Win: C:\ProgramData\docker\config\daemon.json, C:\Users\janef\AppData\Roaming\Docker\last-start-linux-daemon.json 

### Docker Daemon Listen Socket:
Docker listens on a socket by default: /var/run/docker.sock. but it can be overriden by addiing config in daemon.json "hosts: tcp://localhost:<port>"
Default: if there is no "hosts: ..." config in daemon.json, docker daemon default listens to /var/run/docker.sock.
Connfigurable: daemon.json config file:
use "hosts: tcp://0.0.0.0:2375" to config Docker Daemon listens on port 2375 on all network interfaces , 
Use "hosts: tcp://10.0.75.1:2375" to config Docker Daemon to listen to a particular network interface.
It is conventional to use port 2375 for un-encrypted, and port 2376 for encrypted communication with the daemon.
connect to docker daemon at locations: /var/run/docker.sock or tcp://10.0.75.1:2375

### Docker Daemon log location:
Linux: /var/log/messages
Win: C:\Users\janef\AppData\Local\Docker 

### Docker Daemon data location:
Linix: /var/lib/docker
Win: C:\ProgramData\docker

### environment variables
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://10.0.75.1:2375"
export DOCKER_CERT_PATH="C:\Users\janef\.minikube\certs"


## docker daemon running on Linux VM 
When a docker daemon is started on Windows, it will start up MobyLinuxVM VM if it is not started. 
WIN: MobyLinuxVM. it is managed by Hyper-V manager
Hyper-V VM harddrive (it’s typically in C:\Users\Public\Documents\Hyper-V\Virtual hard disks)

## docker log
@Each Docker daemon has a default logging driver, which each container uses unless you configure it to use a different logging driver.
log driver: json-file (default), gelf, awslogs, gcplogs, splunk
docker logs <container>
docker service logs <container>
docker log files: C:\Users\jaal\AppData\Local\Docker  

### Configure the default logging driver by passing the --log-driver option to the Docker daemon. Each log file contains information about only one container..
dockerd --log-driver=<logDriver>
# To set the logging driver for a specific container:
docker run --log-driver=<logDriver> 


## login to a local Docker container - "docker exec"
# run a command inside a running docker container. -i interactrive, -t pseudo terminal tty for stdin/stdout/stderr pipes 
# SSH into a Container
docker exec -it cibcapi_myaccounts_1 /bin/sh  // start a shell in container. 
exit or Ctrl-D to exit SH shell

Generically, use "docker exec -it <container id or name> <command> <args>" to execute whatever command you specify in the container.
docker exec -it cardinal-db <cmd> <args>  // run "cmd args" in container
docker exec -it cardinal-db ls -l  	// run in SH shell, suitable for interactive commands
docker exec -it cardinal-db /bin/bash -c "ls -l"    // use 'exec' to run it. recommend for all non-interactive commands

## login to a remote docker container - "ssh"  
see book C:\UserResource\C_docker\books\'Docker技术入门与实战（第3版）.杨保华(详细书签).pdf' chapter 10

in remote container, install 'apt-get openssh-server' package, then start up SSH daemon service listening on port 22
in remote container, import local host userA's public key (in /home/userA/.ssh/id_rsa.pub) into remote host /home/userB/.ssh/authorized_keys
in remote contianer, map a remote host port123 to container port 22
in local host, run 
	ssh <userB>@<remote host ip address> -p <remote host port123>	// it will ask for 'userB' password. 
	// this will login to the remote docker container as userB

ssh default login user is 'root' if not specified. 

	
## docker network
https://docs.docker.com/network/host/

docker network driver types: none/bridge/overlay/remote  
- none - no network
- bridge - default dirver type. is used to create a network inside a node.
- overlay - is used to create a network across multiple nodes inside a cluster.
- remote - for third party network implementations

there is a default already created network in a docker installation, network name "bridge" with driver type "bridge"

	docker create network [--driver bridge] <networkName>	// the default driver type is bridge
	docker connect <networkName> <containerName> 
	docker disconnect <networkName> <containerName> 
	docker network ls
	docker network inspect <networkName>
	docker network prune
	docker network rm <networkName>

A container can join multiple networks. "docker run" can only accept one network in --network option. use "docker network connect" to join a container into another network. 
	docker run -d -it --network <networkName> --name <containerName> <imageName:tag> <a dummy shell command like 'ls> 
	docker network connect|disconnect <networkName> <containerName> 

docker exec -it <container name> /bin/sh

From inside the container, you can connect to the Internet like google.com, 
or other containers in the same network using their container name or subnet ip address.
you can NOT ping other containers in a different network.
	e.g. ping google.com -c 2  // -c limit to 2 ping attemps
	e.g.  ping -c 2 cibcapi_accounts-service_1 

Containers on the default bridge network can only access each other by IP addresses
	e.g. ping -c 2 172.18.0.1 

clean up networks which aren’t used by any containers
	docker network prune
 

== docker volume. 
# Volumes are never removed automatically, because to do so could destroy data.
Named volumes have a specific source form outside the container, for example awesome:/bar.
Anonymous volumes have no specific source so when the container is deleted, instruct the Docker Engine daemon to remove them.

types: bind mount (mounted to machine filesyste, used in dev), volume (mounted to docker vm storage, use in prd), tmpfs (mounted in machine memory)
driver: local, NFS, cloud object storage system

#docker vm folder starting with double forward slashes. e.g. //var/lib/docker.sock
docker create volume <volume name>	// create a volume in docker vm disk
docker volume ls
docker volume inspect <volume name>
docker volume rm <volume name>   
#remove all volumes not used by at least one container:
docker volume prune

== docker build image
# build a image from a Dockerfile. the built image is stored in local docker repository managed by docker engine.
# The build is run by the Docker daemon, not by the CLI. The first thing a build process does is send the entire context (recursively) to the daemon. 
# Traditionally, the Dockerfile is called Dockerfile and located in the root of the context. You use the -f flag with docker build to point to a Dockerfile anywhere in your file system
docker build -t <repositoryName:tag> [-f /a/b/<Dockerfile>] <a build context path, which tells Docker where to look for any files that need to be added to the image>



== docker image
# https://docs.docker.com/storage/storagedriver/overlayfs-driver/#image-and-container-layers-on-disk
# storeage driver: aufx, overlay2.  
# save docker image:
docker save -o <image archive file> IMAGE [IMAGE...]
# upload docker image to Docker Hub or a private registry
docker push IMAGE
docker image ls -a
docker rmi <image name or id>
# To examine the layers on the filesystem, list the contents of 
ls -l /var/lib/docker/<storage-driver>/layers/
# Create a new image from a container’s changes
docker commit
# '-a': remove all images which are not used by existing containers
docker image prune [-a]	


== docker container
# create and start a container,  and run a one-off command on a container
# When you stop a container, it is not automatically removed unless you started it with the --rm flag
# -dit: d - run in background, i - interactive, t - tty
# --restart unless-stopped : Restart the container unless it is explicitly stopped or Docker itself is stopped or restarted.
docker run --rm -dit [--restart unless-stopped] --network <networkName> --name <containerName> <repositoryName:tag> <command> <args>
-- not working: docker run -v "C:/UserApps/sam-app/hello_world":/var/task lambci/lambda:python3.7 sh echo "hello"

docker ps // list all started containers
docker ps -a  // list all started and stopped containers
# same as 'docker container ls -a'
docker start <container-name or id>
docker stop <container-name or id>
docker rm <container-name or id>	// remove stopped container
docker logs -f <container-name or id>
docker inspect <containerName>
# To view the approximate size of a running container:
docker ps -s 
#remove all stopped containers
docker container prune

== clean up docker host
# This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
docker system prune
#  you must specify the --volumes flag for docker system prune to prune volumes.
docker system prune --volume

== docker service
==== use 'docker' command to start/stop/remove a service and its containers
docker service create -d --replicas=4 --name <serviceName> --mount source=myvol2,target=/app <image:tag>
docker service stop <serviceName>
docker service rm <serviceName>
# Removing the service does not remove any volumes created by the service. Volume removal is a separate step.
  

## PORTAINER - dockerUI, manages docker containers on local host or docker swam:

find out Docker VM IP: run 'ipconfig'

Ethernet adapter vEthernet (DockerNAT):
   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::cc78:1804:2e8:b3a5%29
   IPv4 Address. . . . . . . . . . . : 10.0.75.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
   
steps:
--1. run "docker volume create portainer_data"  // if using "docker run -v portainer_data:/data ..."
1. docker setting -> Shared Drive, select "C:"
   create folder C:\\ProgramData\\Portainer // if using "docker run -v C:\\ProgramData\\Portainer:/data ..."
   
2. Open the Docker Menu on the right side of the Windows Taskbar and go to Settings (3rd. Option).
	On the Tab general, activate the option Expose daemon on tcp://localhost:2375 without TLS (last Option). 
	
3. open a PowerShell with administrator rights and type the following:
	netsh interface portproxy add v4tov4 listenaddress=10.0.75.1 listenport=2375 connectaddress=127.0.0.1 connectport=2375
	netsh advfirewall firewall add rule name="docker management" dir=in action=allow protocol=TCP localport=2375	
	
	# to delete
	netsh advfirewall firewall show rule name=all
	netsh advfirewall firewall delete rule name="docker management"
	
	netsh interface portproxy show all
	netsh interface portproxy delete v4tov4 listenaddress=10.0.75.1 listenport=2375
	
	
	# if do not set firewall rule for TCP 2375 port, portainer container can not connect to Docker Daemon (connecting timeout)
	
4. start the Portainer container using an user-mode PowerShell
	docker run -d -p 9000:9000 -v C:\\ProgramData\\Portainer:/data --name portainer portainer/portainer:1.18.1 -H tcp://10.0.75.1:2375
		Note: 1. where "portainer/portainer" is the repository name.
			  2. connect to Docker Deamon at tcp://10.0.75.1:2375. This is portainer specific parameter
			  3. portainer container HTTP port: 9000. -p <vm host port>:<container port>. 
			     docker also automatically creates a portproxy from <vm host ip>:port to <localhost ip>:port
	docker ps -a
    docker logs -f <a randomly generated container name>
	if portainer container is not started, run "docker start <portainer container id>"
5. http://localhost:9000 -> admin account: admin / admin123 -> 
	5.1 endpoints -> add endpoint -> create "A docker daemon" endpoint, url "10.0.75.1:2375" to connect to Docker Daemon.

	
== container runtime metrics
# CPU, MEM, Network IP metrics for containers.
docker stats [container ...]


== errors and fixes
ERROR: Windows named pipe error: The system cannot find the file specified. (code: 2)
fix: docker machine is not started. start a docker machine on windows.

ERROR: standard_init_linux.go:178: exec user process caused "no such file or directory"
fix: $userhome/.gitconfig git global config file, set : core.autocrlf = input
check all those *.sh scripts added to docker image are ending with LF, not CRLF
