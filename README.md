# NOTES
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

### Default Container Netorks
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
Create a custom network names "custom-isolated-network" that uses the bridged driver allowing all containers to talk to each other on the same 182.18.x.x range

### Inspecting the Network
```bash
docker inspect container_name
```
This will give information such as IP address of the container under "NetworkSettings"

### Host Name Communication
Docker containers can communicate with each other using their host name / container name. This is the preferred method as an IP address can change on boot up.

### Docker DNS Server Notes
Docker DNS Default IP: 127.0.0.11