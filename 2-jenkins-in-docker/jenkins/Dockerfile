FROM jenkins/jenkins:lts-jdk11
USER root
RUN apt-get update && apt-get install -y \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
RUN mkdir -p /etc/apt/keyrings && \
       curl -fsSL https://download.docker.com/linux/debian/gpg | \
       gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "docker-workflow"
