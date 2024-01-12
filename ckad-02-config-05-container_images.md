# CONFIGURATION - Container images

## **1. Define Dockerfile**
---

Sample Dockerfile for Python:

```yaml
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

## **2. Comandos básicos Docker**

List images: `docker images`

Build container: `docker build -t webapp-color .`

Run container and change port: `docker run -p 8282:8080 --name webapp-color webapp-color` 

docker tag

docker image rm

docker push/pull


## Possible exam questions:

1. Using a predefined Dockerfile, build an image and push to a repo (or build in a worker node)
2. Use the previous image in a Pod
3. Fix problems in a Dockerfile and build the image
4. Delete an image
