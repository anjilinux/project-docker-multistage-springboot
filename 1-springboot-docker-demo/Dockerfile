# STAGE: Build
FROM gradle:7.4.2-jdk11 as builder

# Create Working Directory
ENV BUILD_DIR=/home/build/
RUN mkdir $BUILD_DIR
WORKDIR $BUILD_DIR

# Download Dependencies
COPY build.gradle settings.gradle $BUILD_DIR
RUN gradle build -x test --continue

# Copy Source
COPY src src

# Build jar
RUN gradle build -x test

# Rename jar
RUN export PROJECT_NAME=$(gradle properties -q | grep "name:" | awk '{print $2}') && \
  export PROJECT_VERSION=$(gradle properties -q | grep "version:" | awk '{print $2}') && \
  cd build/libs/ && \
  mv ./${PROJECT_NAME}-${PROJECT_VERSION}.jar app.jar

# STAGE: Deploy
FROM openjdk:11-jre-slim-buster

# Create app directory
ENV APP_HOME=/home/app
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME

# Copy jar file over from builder stage
COPY --from=builder /home/build/build/libs/app.jar $APP_HOME

EXPOSE 8080

CMD exec java $JAVA_OPTS -jar ./app.jar