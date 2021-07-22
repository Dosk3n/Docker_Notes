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