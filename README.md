# DOCKER NOTES
Course followed: https://www.youtube.com/watch?v=fqMOX6JJhGo&ab_channel=freeCodeCamp.org

## Creation Notes

### Pull an image
```bash
docker pull ubuntu
```
This will pull the latest image of ubuntu to the machine.
Similar to git clone.

### Create a container from image
```bash
docker run -t -d --name test_ubuntu ubuntu
```
run will create the container
-t assigns a terminal
-d runs the container in the background
--name assigns a name to the container

### Map a local folder to the container at creation
```
run -t -d -v D:\Docker\hello_docker:/mnt/hellodocker --name ubuntu_mapped ubuntu
```
-v Volume mapping (local_folder:container_folder)

### Enviromental Variables
```bash
docker run -e APP_COLOR=blue docker_container_name
```
Had you written a python script with a variable called color
which you had set to blue, you could change that variable at
container creation using an enviromental variable.
In python your variable would be set as:
```python
import os
color = os.environ.get('APP_COLOR')
```
-e will pass the value in as the variable content.

## Networking Notes

### Default Container Networks
```bash
docker run ubuntu
```
Creates a defualt bridged network usually on the 172.x.x.x range. This allows all containers to talk with each other.
```bash
docker run ubuntu --network=none
```
Disables network access on that container
```bash
docker run ubuntu --network=host
```
The host option will use all the same network configurations as the host machine so ports from the host machine will link directly from the host to the container. This will stop multiple containers being able to use the same port.

### Custom Network Creation
```bash
docker network create \
    --driver bridge \
    --subnet 182.18.0.0/16
    custom-isolated-network
```
Create a custom network named "custom-isolated-network" that uses the bridged driver allowing all containers to talk to each other on the same 182.18.x.x range

### Inspecting the Network
```bash
docker inspect container_name
```
This will give information such as IP address of the container under "NetworkSettings"

### Host Name Communication
Docker containers can communicate with each other using their host name / container name. This is the preferred method as an IP address can change on boot up.

### Linking Containers Networks
NOTE: If using docker-compose & version 2+ yml format, linking is not required as all containers will use a default bridged network.

For a container to use another containers host name to communicate between each other, they need to be told about the container at creation.
```bash
docker run -d --name ubuntu_server --link mysql:mysql --link apache:apache ubuntu
```
This command will create a container with the name "ubuntu_server" using the "ubuntu" image. This container will be linked to the mysql and apache containers.

### General Docker DNS Server Notes
Docker DNS Default IP: 127.0.0.11

## Docker Volumes
NOTE: -v is the old way of working with mounting and the --mount option is the new preferred way:
```bash
docker run \
    --mount type=bind, source=/data/mysql, target=/var/lib/mysql mysql
```

```bash
docker volume create data_volume_name
```
This will allow you to create a docker volume that you can assign to docker containers.

### Volume mounting
```bash
docker run -v data_volume_name:/var/lib/mysql mysql
```
Supplying a volume name will assign it to the location supplied after the colon. If the volume name does not exist it will be created.

### Bind mounting
```bash
docker run -v /var/mysql:/var/lib/mysql mysql
```
By supplying a local folder instead of a docker volume it will bind the supplied host directory to the containers supplied directory (host:container).

# DOCKER COMPOSE NOTES

## Docker Compose
Docker compose allows the creation of a "config" file to create the containers. Docker is a quick way of bringing up an image on the command line, however if there are a lot of options being added on the command line it can be confusing. This is where creating a config file and using docker compose is the better choice.

By using docker compose we are able to create a single yml file that can include networking, container links, port configuration and more all in one easily readable place.

### Docker Compose Versions
#### Version 1
There are multiple versions of docker-compose which can be distinguised from the format of the yaml file. Version 1 you will see the container names as the root of each section and has very limited config options. Networking in Version 1 requires --link options.

#### Version 2
Version 2 needs to start with "version: 2" and the container names will be under a "services:" section.

Networking in Version 1 requires --link options network the containers together while in Version 2+ there is a default bridged network allowing all containers to talk to each other and the links section of docker-compose.yml can be removed.

#### Version 3
Version 3 continues from Version 2 with the same features listed above but will start with "version: 3" and has additional options such as support for docker swarm.

### Multiple Containers via Docker Compose
Assuming we have a situation where we want 5 different containers set up. To do this with regular docker we would need to use docker run 5 times like this:
```bash
docker run -d --name redis redis
```
```bash
docker run -d --name db postgres:9.4
```
```bash
docker run -d --name vote -p 5000:80 --link redis:redis voting-app
```
```bash
docker run -d --name result -p 5001:80 --link db:db result-app
```
```bash
docker run -d --name worker --link db:db --link redis:redis worker
```

To create this in a "docker-compose.yml" file would look like this (This example is docker-compose.yml v1 format):
```yml
redis:
    image: redis
db:
    image: postgres:9.4
vote:
    image: voting-app
    ports:
        - 5000:80
    links:
        - redis
result:
    image: result-app
    ports:
        - 5001:80
    links:
        - db
worker:
    image: worker
    links:
        - redis
        - db
```
Then by running docker compose you can bring up all containers
```bash
docker-compose up
```
NOTE: If using docker-compose & version 2+ yml format, linking is not required as all containers will use a default bridged network.

### Custom Unbuilt Images
If you are not using an image that can be pulled from dockerhub or you have an unbuilt image (folder with docker file and files), you can replace "image" with "build" and supply the location to the folder with dockerfile to build the image at the time you bring this up with docker compose
```yml
version: 2
services:
    redis:
        image: redis
    db:
        image: postgres:9.4
    vote:
        build: ./vote_app_location
        ports:
            - 5000:80
        links:
            - redis
```

### Container Start Up Order
```yml
version: 2
services:
    redis:
        image: redis
    db:
        image: postgres:9.4
    vote:
        image: voting-app
        ports:
            - 5000:80
        depends_on:
            - redis
```
By adding a "depends_on" option, we can make sure that the container will only start if the depended on container successfully starts.

### Networking
Multiple networks can be set up with docker compose and assigned to specific containers allowing some networks to be front end facing and others back end facing or with no external network connections. This is done by adding a "networks:" section.
In this example "vote" and "result" apps are websites so front facing for user access but also need to talk to back end databases, while db and redis only need a back end connection.
```yml
version: 2
services:
    redis:
        image: redis
        networks:
            - back-end
    db:
        image: postgres:9.4
        networks:
            - back-end
    vote:
        image: voting-app
        networks:
            - front-end
            - back-end
    result:
        image: result
        networks:
            - front-end
            - back-end

networks:
    front-end:
    back-end:
```