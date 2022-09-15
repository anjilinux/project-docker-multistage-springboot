# springboot-docker-demo

```bash
docker build -t springboot-demo:single-stage -f Dockerfile-single .
docker run --name single-stage --rm -p 8080:8080 -d springboot-demo:single-stage
docker exec -it single-stage bash
```

```bash
docker build -t springboot-demo:multi-stage -f Dockerfile .
docker run --name multi-stage --rm -p 8081:8080 -d springboot-demo:multi-stage
docker exec -it multi-stage bash
```

```bash
# cleanup
docker rmi springboot-demo:single-stage springboot-demo:multi-stage
```
