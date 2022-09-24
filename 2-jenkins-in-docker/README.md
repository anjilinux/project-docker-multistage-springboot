# springboot-docker-demo

```bash
# Build containers
time docker build -t springboot-demo:single-stage -f Dockerfile-single .
time docker build -t springboot-demo:multi-stage -f Dockerfile .

# Start containers
docker run --name single-stage --rm -p 8080:8080 -d springboot-demo:single-stage
docker run --name multi-stage --rm -p 8081:8080 -d springboot-demo:multi-stage

# On separate shell sessions, do this
docker exec -it single-stage bash
docker exec -it multi-stage bash

# From inside the container, do this
gradle --version
javac --version
java --version
```

```bash
# cleanup
docker rmi springboot-demo:single-stage springboot-demo:multi-stage
```

## Resources

* <https://www.jenkins.io/doc/book/pipeline/docker/>
