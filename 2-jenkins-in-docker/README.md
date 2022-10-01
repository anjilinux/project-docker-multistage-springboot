# springboot-docker-demo

1. Create jenkins network:

```bash
docker network create jenkins
```

2. Start Docker-in-Docker:

```bash
docker run --name docker-daemon --rm --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --publish 2376:2376 \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client \
  docker:dind \
  --storage-driver overlay2

# Get IP Address
docker inspect -f \
  '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  docker-daemon
```

3. Start Jenkins

```bash
docker run --name jenkins --rm --detach \
  --network jenkins \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  jenkins-with-docker:latest


  # --env DOCKER_TLS_CERTDIR=/certs \
```

## Resources

* <https://www.jenkins.io/doc/book/pipeline/docker/>
