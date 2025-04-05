# Docker Tutorial

### Why is Docker useful?

Docker allows you to run each service in a seperate container. Run each serivce with its own deppendencies in seperate containers.

### What are Containers?

![image](https://i.imgur.com/lEQTxOz.png)
Containers have their own file system, network and processes except they share the same host's kernel.

> FYI : In different linux flavors also they all share the same kernel but have different softwares installed on top of the kernel.
> ![image](https://i.imgur.com/iznmXuE.png)

### What is Docker?

Docker is not meant to virtualize the host machine and run different operating systems on the same hardware. It is meant to package and containerize applications, ship them and run them on any machine.

Since Docker utilizes the kernel of the host machine, different linux flavors can run docker containers wihtout any problems. That's why you won't be able to run a windows container on a docker working with linux host machine for that you will require Docker running on a windows machine.

Then How come we can run a linux container on a windows host machine?

### Containers vs Virtual Machines

![image](https://i.imgur.com/suuU1jM.png)

1. VM are slower and heavyweight because they have their own seperate OS and kernel running directly on top of the hardware. Where as containers share the same kernel and run on top of the host machine that's why they are lightweight and faster.
2. Containers are not isolated from each other. They can communicate with each other and share the same kernel. Where as VMs are completely isolated from each other.

It's not an either or situation. Usually we use both of them together.
![image](https://i.imgur.com/mTHLok7.png)

### How is it done

There are lots of containzeried versions of applications that are available and companies make them available in the form of docker images on docker hub. A store where you can find image of services and tools like nginx, mysql, mongo, redis, etc.

`docker run nodejs` : This will pull the image of nodejs from docker hub and run it. `docker run redis` is also that same.
If you need to run multiple instances of the same service just create muliple containers and then use a load balancer like ngnix to route the traffic to the containers.

### Container vs Image

- A Docker Image is like a Package or a Template. It is a read-only template that contains everything needed to run an application. Used to create one or more containers.
- Containers are running instances of images, that are isolated and have their own environement and set of precessors.
- If you cannot find image of what you need you can create your own image by writing a Dockerfile.

### Docker in DevOps

Since developers know how to code and deploy the application, they can also write a Dockerfile and create their own images, so people from operation team can just pull the images and run them.

## Introduction to Docker Commands

- `docker version` : To verify docker in installed.
- `docker run <image-name>` : To run the image.
  - `docker run -d <image-name>` : To run the image in detached mode
  - Eg. `docker run -d nginx` : Will run the nginx image
  - If the image is not present on the system it will be downloaded from docker hub
- `docker ps -a` : To see all the containers
  - `docker ps` : To see all the running containers
- `docker stop <container-id>/<container-name>` : To stop the container. **When providing Docker id you can provide the first few characters of the id and it will automatically find the id.**
- `docker rm <container-id>/<container-name>` : To remove the container. If it prints the name back then it means the container was deleted successfully.
- `docker images` : To see all the images present in the system
- `docker rmi <image-id>/<image-name>` : To remove the image. If it prints the name back then it means the image was deleted successfully.
- `docker pull <image-name>` : To pull the image from docker hub and not run it. Just keep it in the system for future use.
- `docker run ubuntu` : You think it will an ubuntu image, but when you do `docker ps -a` you see is status that it `Exited (0)`
  - This happens because docker is not meant to run operataing systems. It is meant to run services and a docker container runs as long as the service is running.
  - If the service crashes/exits then the container will also stop.
  - Also Ubuntu is just an image that is used as a base image to create other images.
- `docker exec <container-id> <command>` : To run a command on the container
- Attached vs Detached Mode :
  - `docker run <container-name>` : This will run the container in attached mode. Where you will see all the outputs of the container in your terminal and terminal will not be free to do anything else.
  - `docker run -d <container-name>` : This will run the container in detached mode. Runs docker container in the background and you can use the terminal for other things.
  - `docker attach <container-id>` : To attach to a container later on.

## Commonly used tags with docker run

- `-d` : To run the container in detached mode
- `docker run <image_name>:version` : To run the image with a specific version
- `-i` : To run the container in interactive mode, by default docker containers when run do not listen to input from the terminal.
  - `docker run -t <container-name>` : Here docker will accept the input from the terminal. But prompts send by the container/service running in the container will not be visible in the terminal.
- `-t` : Pseudo-Terminal, disaply prompts from the container on your the terminal.
  - `docker run -it <container-name>` : Here docker will accept the input from the terminal and prompts send by the container/service running in the container will be visible in the terminal.
    ![image](https://i.imgur.com/YwX6Zsh.png)

### Working with Ports

![image](https://i.imgur.com/cWW0vxr.png)

- Commands you run to get IP of a docker container is it's internal IP, which cannot be used to access the service from outside the container. The internal PORT of the service needs to be mapped to a PORT on the host machine for external access.
- `docker run -p <host-port>:<container-port> <image-name>` : Will run the image and map the container port to the host port. So that we can access the service using host-port.
- Inside docker multiple containers can have the same container port but the Host Port should be unique.

### Volumne Mapping

- `docker run mysql` : If we run a mysql container, then data will be stored in /var/lib/mysql inside the container. If we delete the container then the data will be lost.
- `docker run -v <host-path>:<container-path> <image-name>` : Will map the host path to the container path. So that the data will be stored in the host path and not in the container. This is called volume mapping and prevents data from being lost when the container is deleted. It will implicitly mount the external directory to a folder inside the docker container.
  ![image](https://i.imgur.com/6PflRvY.png)

#### Inspect Container : `docker inspect <container-id>`

#### Container Logs : `docker logs <container-id>`

## Environment Variables

- It best to use environement variables for sensitive data like passwords and for data that you would like to change often.
- Let's say we have this python application that takes background color as an environment variable.

```python
import os
from flask import Flask

app = Flask(__name__)
...
...

color = os.environ.get('APP_COLOR')

@app.route('/')
def main():
    print(color)
    return render_template('hello.html', color=color)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

`export APP_COLOR=blue; python app.py` : To set the environment variable and run the application with blue as the background color.

### How to do this with app.py inside a container.

Once application is packaged into a iamge, it can be run using : `docker run -e APP_COLOR=blue simple-weebapp-color`
`-e` : To set the environment variable inside the container.

`docker run --name postgres-db -e POSTGRES_PASSWORD=mysecretpassword -d postgres` : To run postgres container with a password.

#### Inspect Environment Variables : `docker inspect <container-id>` under config you will find the environment variables.

## Docker Images

### How to create your own image

- First we need to decide what application we are creating the image for and how to application is build. So start by thinking what you will do to deply the application manually.
- Write the steps required in the correct order.
- Follow are the steps on how one would deploy/run flask application manually and then convert it into docker image.
  ![image](https://i.imgur.com/mjQ54vA.png)

### Dockerfile

Dockerfile is written in a Instruction Argument format.

```dockerfile
    FROM Ubuntu #From is an instruction and all docker files start with a from command
    RUN apt-get update
    RUN apt-get install -y python

    RUN apt-get install -y python-pip
    RUN pip install flask
    RUN pip install flask-mysql

    COPY . /opt/source-code
    ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run --host=0.0.0.0:8080
```

### Layered Architecture

When docker builds an image it does so in a layered fashion. Each line of instruction creates a new layer in the docker image with just the changes from the previous layer.
![image](https://i.imgur.com/UvcDwMX.png)

- Docker build is run in various stages. Each stage is a layer and all the layers build are cached, so we can start docker build from a particular stage if it fails. Also if we add new steps in the build process, we don't have to start all over again.
- If a step fails, you resolve the issue and rebuild the image, docker will automatically start from the last successful step.
- `docker build -t <image-name> --no-cache .` : To build the image without using the cache.

### What can you containerize?

You can containerize any application that can run on a Linux system, Ex Browers : chrome, firefox, Services : curl, redis, etc and applications : spotify, etc.

`docker run ubuntu sleep 5` : Start a new ubuntu conatiner. Runs sleep command for 5 seconds, enter the Ubuntu image and then exits the container because there's nothing to do.

- What if I don't want to pass sleep command again and again. Then I can just build my own image using ubuntu image.

```dockerfile
#name: ubuntu-sleeper
FROM ubuntu
CMD ["sleep", "5"] #we can pass arguments to the CMD instruction in json format ["command", "arg1", "arg2"]
```

`docker build  -t ubuntu-sleeper .` : Build the image.
`docker run ubuntu-sleeper` : Run the image,and it will run sleep command for 5 seconds automatically.

#### How to change number of seconds to sleep

Option 1 : `docker run ubuntu-sleeper sleep 10` : To pass arguments to the CMD instruction.

- But here again we have to pass the complete arguments, we want to pass only the seconds to sleep. That is what entrypoint instruction is for.

```dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"] #command that will be executed when the container is run/starts
```

- Now whatever arguments we pass to the container will be appended to the entrypoint instruction.

  - `docker run ubuntu-sleeper 10` == `docker run ubuntu-sleeper sleep 10`

- Configure a default value for the entrypoint instruction

  - ```dockerfile
    FROM Ubuntu

    ENTRYPOINT ["sleep"]
    CMD ["5"] #default value for the entrypoint instruction
    ```

- Override the default entrypoint intruction by using the --entrypoint instruction.
  - `docker run --entrypoint sleep2.0 ubuntu-sleeper 10` : Now command at startup will be sleep2.0

## Docker Networking

### Default Networks

- When we install docker it creates 3 networks automatically. Bridge, Host and None.
- Bridge network is the default network a container is connected to.
  - All containers connected to the bridge network can communicate with each other using their internal IP address.
  - To make these containers accessible from outside the docker host, we need to map the container's port to the host's port.
- Host network is used when we want to use the host's network stack.
  - Directly connected to the host's network stack. Removes any network isolation between the container and the host.
- None network is used when we want to disable all networking for a container and it is completely isolated.
  ![image](https://i.imgur.com/fDCvloA.png)

### User-Defined Networks

What if user wants to form subnetworks within the docker network, to divide containers into different groups.
![image]![alt text](image-1.png)

### Inspect Network : `docker inspect <container_name>`

### Embedded DNS Server

- Docker has an embedded DNS server that allows containers to reach each other using their container name. Containers can either use the Container IP address or container name to communicate with each other.
- **Build in DNS server always run at 127.0.0.11:53**
  ![image](https://i.imgur.com/J87CYYA.png)
- Docker uses Network Namespace to isolate containers from each other. By creating a separate network namespace for each container. Then uses Virtual Eternet pairs to connect the containers to one another.

## Docker Storage

How Docker makes itself efficient
![image](https://i.imgur.com/EX0hmO9.png)
Since Docker already build the Ubuntu, apt packagees and other dependencies till layer 3 for Dockerfile, it will just reuse those for Dockerfile2, saving a lot of space and time.

- Layers created during the build process are called Image Layers and when we run the image, it creates container layers.
- Image layers are read-only because one image layers might be used by multiple containers. Container layers are read-write.
- Container Layer is temporary and gets destroyed when the container is removed.
  ![image](https://i.imgur.com/GMVkkt6.png)
- Any extra files that you add to the container will be added to the container layer. If you make changes to file inside the container, the moment you save the file, it will be added to the container layer. This mechanism is called Copy on Write, where Docker only copies the file which it is modified. This way original image layers which used by other containers as well are not modified and when current container is run modified files are served from the container layer.
  ![image](https://i.imgur.com/35JR5kQ.png)
  We know that container layers are temporary and gets destroyed when the container is removed. So how do we persist data in the container?

### volumes

#### Volumne Mounting

- `docker volumn create <volume_name>` : Create a volume under /var/lib/docker/volumes directory on host machine.
- Now we can mount this volume to a directory inside container using the -v flag.
  - `docker volumne create data_volume`
  - `docker run -v data_volume:var/lib/mysql mysql` : Mount the data volume to the mysql container. Now all the data send to the container will be stored in the data volume on host machine. Even if the container is removed, the data will be persisted.
  - If you do not create a volumne beforehand, docker will create a volumn with the name used with -v Like here docker will create a data_volume volumn if it is not present.

#### Bind Mounting

- When we mount a directory from the host machine that is not inside /var/lib/docker/volumes directory on host to a directory inside docker. It is called bind mounting.
  - `docker run -v /data/mysql:/var/lib/mysql mysql`
    ![inage](https://i.imgur.com/WjlwpAG.png)

#### New Method : -v flag is the old way

`--mount` flag is the new way

- `docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`

### Storage Drivers

- Docker storage drivers are responsible for maintaining the layered Architecture of Docker containers and images, creating writeable layers, etc.
- Common Storage Drivers : AUFS, Btrfs, Device Mapper, Overlay, Overlay2, ZFS, VFS

## Docker Compose

- Till now we were running docker containers using docker run command. But what if we have a complex application that uses multiple services. In that case we can use docker-compose. With docker-compose we can create a `docker-compose.yml` file and run all the services using `docker-compose up` command.

### Example Application

- We will work with this application in multiple sections.
  - voting-app : python based web application that allows users to vote.
  - in-memory DB using redis
  - worker : .NET based worker that processes the votes.
  - Database : Postgresql
  - result-app : Node.js based web application that displays the results.
    ![Application structure](https://i.imgur.com/6Xi2CCt.png)

### How to start the application using docker run

1. We will start all the services
   1. `docker run -d --name redis-server redis`
   2. `docker run -d --name db postgres`
   3. `docker run -d --name vote -p 5000:80 voting-app` : starts voting-app container and maps port 5000 of host machine to port 80 of voting-app container. Since voting-app is a web application it will start on port 80 by default.
   4. `docker run -d --name result -p 5001:80 result-app`
   5. `docker run -d --name worker worker`

Now when we visit localhost:5000 we will see an Internal Server Error. Because voting-app does not know where is redis to store the votes. When working of a containers depends on another containers we create a link between them using --link flag.

2. Creating the link between containers before starting them
   1. Command i and ii will be same
   2. `docker run -d --name vote --link redis-server:redis -p 5000:80 voting-app` : Link to the redis-server container and alias it as redis in the voting-app container.
   3. `docker run -d --name result --link db:db -p 5001:80 result-app`
   4. `docker run -d --name worker --link db:db --link redis-server:redis worker`

### Creating a similar docker-compose.yml file

```yml
redis-server: # name of the container
  image: redis # image to use
db:
  image: postgres
vote:
  image: voting-app
  ports:
    - 5000:80
  links:
    - redis-server:redis
result:
  image: result-app
  ports:
    - 5001:80
  links:
    - db
worker:
  image: worker
  links:
    - redis-server:redis
    - db
```

`docker-compose up` : To create containers for all the services and start the entire application stack.

### Docker compose -build

- In the above dockerfile we assumed all the images are already build. But vote, result and worker are our own application, it is not necessary that they are already build and available in a docker registry.
- To instruct docker to build an image instead of pulling it from a registry we can use the build option in docker-compose.yml file and point build to the directory which contains the application code and a dockerfile with instructions to build the docker image.
  ![image](https://i.imgur.com/knuZWyV.png)
- `docker-compose up -d --build` : We can run this command from the directory where docker-compose.yml file is present.

### Different Versions of docker-compose.yml file

- Starting from version 2 we put all services below service: tag
  ![image](https://i.imgur.com/9MGVhBL.png)
- Version 3 introduced support for docker swarm mode.

### Networks

- Let's say we wanted seperate out frontend and backend services to different networks this way wecan isolate client facing and client accessible network from backend network.

```yml
version: "3.8" # Updated version for better network control

services:
  redis-server:
    image: redis
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

  vote:
    image: voting-app
    ports:
      - 5000:80
    networks:
      - frontend
      - backend # Needs to talk to redis (backend)
    depends_on:
      - redis-server

  result:
    image: result-app
    ports:
      - 5001:80
    networks:
      - frontend
      - backend # Needs to talk to db (backend)
    depends_on:
      - db

  worker:
    image: worker
    networks:
      - backend
    depends_on:
      - redis-server
      - db

# Define custom networks
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

## Docker Engine

It is simply a host with docker installed on it.

- When you install docker you are installed 3 components

  - Docker CLI
  - Docker Daemon : Background process that manages docker objects such as images, containers, networks, and volumes.
  - REST API : Docker REST API server is the REST API interface that the docker CLI uses to communicate with the docker daemon. User can create their own tools using this REST API.

- It is not necessary to have Docker CLI and other 2 parts on the same system. We can have docker daemon running on one system and docker CLI on another system.
  - `docker -H <host-ip>:<port></port> <command>` : To connect to a remote docker daemon. Eg. `docker -H 192.168.1.10:2375 ps`

### How are application containerized?

- Docker uses namespaces to isolate the workspaces. Process ID, Network, InterProcess, Mount and Unix Timesharing are created in their own namespaces thereby providing isolating.

## Docker Orchestration

- When load increases we need to scale our application, that's when we deploy muliple instances of our application that run on our docker host.
- What if some containers crash or host itself crash.
- To deal with these cases we have Container Orchestration. It consists of tools and scripts that help us manage our containers and hosts in a production environment.
- Container Orchestration solutions allow us to deploy multilp replicas of our application and scale them up and down based on the load and they also provide scripts to do this automatically. They also provide support for advanced features such as rolling updates, rolling restarts, health checks, and networking between these containers.
- Container Orchestration Solutions:
  - Docker Swarm
  - Kubernetes : Most famous and widely used
  - Apache Mesos

## Docker Scout : Docker's free tool to monitor and secure your docker environment.

## Common Questions and Doubts

### 1. How dockerfile is linked with docker-compose.yml file?

- dockerfile: Generally docker hub has images for commonly used applications and services. But if we want to create our own custom image then we need to create a dockerfile.
- docker-compose.yml: When we have to work with multiple containers, link them, expose them to ports, etc. Instead of having to run multiple docker commands, we create a docker-compose.yml and then use `docker-compose up -d --build` to create and start all the containers in one go.
- Now if we want to create a container using our custom image we can specify it in docker-file to use our dockerfile instead of image pulling from docker hub.

```dockerfile
FROM node

WORKDIR /

COPY package.json package.json #Copy package.json from current directory to / directory in container

RUN npm install

COPY . . #copy everything from current directory to / directory in container

# Set default PORT environment variable
ENV PORT 3000

# Run the application, without sh -c command docker will not expand environment variables
CMD ["sh", "-c", "node app.js"]
#example use `docker build -t <image-name> <path to this file>.`
#`docker run -e PORT=4000 -p 4000:4000 <image-name>`
```

```yml
version: "3.8"
services:
  backend:
    build: /src #use build instead of image
    ports:
      - "3000:3000"
    volumns:
      - ./src:/src #mount host machine directory to container directory to persist data

  database:
    image: mongodb
    ports:
      - "27017:27017"
    volumns:
      - ./data/db:/data/db
```

`docker-compose up -d --build` : Will build and run the containers in docker-compose.yml file which will inturn build dockerfile and mongodb iamges.

### 2. Can we create docker-compose for a backend service without creating docker file for backend service?

- Yes, we can use docker-compose to create a container for backend service without creating a dockerfile for backend service. We can use the image of the service on which backend is running from docker hub and create a container for it using docker-compose.yml file. But this would require us to pass all command line args in docker-compose.yml file which is not a good practice.
- Use docker-compose.yml directly when your use cases are simple and don't require a custom image.

```yml
version: "3.8"

services:
  backend:
    image: node # Use official Ubuntu image
    working_dir: /app # Set working directory
    volumes:
      - .:/app #instead of copying files to docker container, directly  mount host directory to container directory
    environment:
      - PORT=3000
    ports:
      - "3000:3000" # Expose port
    command: >
      sh -c "apt-get update &&
             apt-get install -y nodejs npm &&
             npm install &&
             node app.js"
```

### 3. How to execute commands on docker container service?

1. `docker exec -it <container-id> <command>` : Use when you have to only execute 1 or 2 commands on running container.
2. `docker exec -it <container-id> <bash or sh>` : Use when you have to execute multiple commands on running container.
