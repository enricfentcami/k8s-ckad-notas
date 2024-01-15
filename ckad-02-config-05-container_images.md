# CONFIGURATION - Container images

## **1. Define Dockerfile**

Sample Dockerfile for Python:

```yaml
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

Sample Dockerfile for Ubuntu sleep "ubuntu-sleeper":

```yaml
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

Test cases:

* `docker run ubuntu-sleeper` -> Will execute the "sleep" with the default parameter "5"

* `docker run ubuntu-sleeper 10` -> Will execute the "sleep" with the parameter "10" that overrides the CMD value

## **2. Docker commands**

### Images

Docker images are a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

Build an Image from a Dockerfile (version tag is optional):

`docker build -t <image_name> .` -> `docker build -t webapp-color:latest .`

Build an Image from a Dockerfile without the cache:

`docker build -t <image_name> . –no-cache`

List local images:

`docker images`

Delete an Image:

`docker rmi <image_name>`

Remove all unused images:

`docker image prune`

### Containers

A container is a runtime instance of a docker image. A container will always run the same, regardless of the infrastructure. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.

Create and run a container from an image, with a custom name:

`docker run --name <container_name> <image_name>`

Run a container with and publish a container’s port(s) to the host:

`docker run -p <host_port>:<container_port> <image_name>` -> `docker run -p 8282:8080 --name webapp-color webapp-color:latest` 

Run a container in the background:

`docker run -d <image_name>`

Start or stop an existing container:

`docker start|stop <container_name> (or <container-id>)`

Remove a stopped container:

`docker rm <container_name>`

Open a shell inside a running container:

`docker exec -it <container_name> sh`

Fetch and follow the logs of a container:

`docker logs -f <container_name>`

To inspect a running container:

`docker inspect <container_name> (or <container_id>)`

To list currently running containers:

`docker ps`

List all docker containers (running and stopped):

`docker ps --all`

View resource usage stats:

`docker container stats`

### DockerHub

Docker Hub is a service provided by Docker for finding and sharing container images with your team. Learn more and find images at https://hub.docker.com

Login into Docker:

`docker login -u <username>`

Publish an image to Docker Hub:

`docker push <repository>/<image_name>` -> ` docker push hub.docker.com/webapp-color:latest`

Search Hub for an image:

`docker search <image_name>`

Pull an image from a Docker Hub:

`docker pull <image_name>`

## Possible exam questions:

1. Using a predefined Dockerfile, build an image and push to a repo (or build in a worker node)
2. Use the previous image in a Pod
3. Fix problems in a Dockerfile and build the image
4. Delete an image
