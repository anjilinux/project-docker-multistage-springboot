# springboot-docker-demo

## Setup Docker Daemon and Jenkins with Docker Client

1. Create jenkins network:

```bash
docker network create jenkins
```

2. Create volume to host docker certificates

```bash
docker volume create docker-client-certs
```

3. Create volume to host jenkins data

```bash
docker volume create jenkins-data
```

4. Start Docker-in-Docker:

```bash
docker run --name docker-daemon --rm --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --publish 2376:2376 \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-data:/var/jenkins_home \
  --volume docker-client-certs:/certs/client \
  docker:dind \
  --storage-driver overlay2
```

5. Start Jenkins

```bash
docker run --name jenkins --rm --detach \
  --network jenkins \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume docker-client-certs:/certs/client:ro \
  jenkins-with-docker:latest
```

6. Get IP Address for Docker

```bash
docker inspect -f \
  '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  docker-daemon
```

7. Use the above IP Address, then do the following in the 

* Go to springboot-pipeline and click Configure.
* Tick the box for `This project is parameterized`.
* Click `Add Parameter`.
* Click on `String Parameter`.
* Enter the following for the **String Parameter**:
  * **Name**: `DOCKER_HOST_IP`.
  * **Default Value**: Enter the IP address from the previous step.
* Click `Save`.

## Debugging

```bash
# Interact with Docker
docker run --name docker -it --rm --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --volume docker-client-certs:/certs/client:ro \
  docker:latest sh
```

## Resources

* <https://www.jenkins.io/doc/book/pipeline/docker/>
