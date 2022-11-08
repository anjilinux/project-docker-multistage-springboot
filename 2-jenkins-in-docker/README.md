# jenkins-in-docker-demo

## 1. Setup Docker Daemon and Jenkins with Docker Client

a. Go to jenkins folder

```bash
cd 2-jenkins-in-docker/jenkins
```

b. Create jenkins network:

```bash
docker network create jenkins
```

c. Create volume to host docker certificates

```bash
docker volume create docker-client-certs
```

d. Create volume to host jenkins data

```bash
docker volume create jenkins-data
```

e. Start Docker-in-Docker:

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

e. Build Jenkins with Docker CLI container

```bash
docker build -t jenkins-with-docker:latest .
```

f. Start Jenkins

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

# Wait a few seconds, then get Jenkins Initial Admin Password
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

g. Open a browser and go to [http://localhost:8080](http://localhost:8080), then do the following:

* In **Administrator password**, enter the password that you obtained in the previous step, then click on **Continue**.
* In **Customize Jenkins** page, click on **Install suggested plugins** and wait for all the plugins to finish installing.
* In **Create First Admin User** page, click **Skip and continue as admin**.
* In **Instance Configuration** page, click **Not now** or **Save and Finish**.
* In **Jenkins is ready!** page, click **Start using Jenkins**.

h. Get IP Address for Docker Daemon.

```bash
docker inspect -f \
  '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  docker-daemon
```

i. Copy the above IP address, then add it as environment variable in Jenkins, as follows:

* Click on **Manage Jenkins**.
* Click on **Configure System**.
* In the **Global properties** section, tick the **Environment variables** box, click **Add** and add the following environment variable:
  * **Name**: `DOCKER_HOST_IP`
  * **Value**: `Enter the IP Address from previous step`.
* Click **Save**, which should take you back to the Home Screen.

## 2. Create docker-pipeline Job in Jenkins

At the Jenkins Home Screen, do the following to create a new pipeline:

a. Click **New Item**, which will take you to a new page.

* In **Enter an item name** enter `docker-pipeline`.
* Click on **Pipeline**.
* Click **OK** button.

b. In the Job **Configuration** page, scroll to the **Pipeline** section and enter the following:

* In **Definition**, select `Pipeline script from SCM`.
* In **SCM**, select `Git`.
* In **Repository URL**, enter `https://github.com/fabiogomezdiaz/youtube-channel.git`.
* In **Branch Specifier (blank for 'any')**, enter `main`.
* In **Script Path**, enter `2-jenkins-in-docker/jenkins/Jenkinsfile`.
* Click **Save**, which should take you home page of the `docker-pipeline` job.

## 3. Start a new build in the "docker-pipeline" Job

* On the homepage for the `docker-pipeline` job, click **Build Now**.
* Under **Build History**, click on the latest build number (i.e. `#1`) to go to that build's page.
* Now click on **Console Output** to see the build logs.
* If you see a `Finished: SUCCESS` message to indicate that the pipeline worked!

## 4. Cleanup

To clean up your workstation, run the following commands:

```bash
docker stop jenkins
docker stop docker-daemon
docker volume rm jenkins-data
docker volume rm docker-client-certs
docker network rm jenkins
```

## Debugging

Run this command from your workstation to open a shell in a docker client container against the docker-daemon container.

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
