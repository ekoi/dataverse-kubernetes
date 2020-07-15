FROM openjdk:8-jdk-alpine
VOLUME /tmp
WORKDIR /usr/src
ARG APP_BASE_DIR=/usr/local/dts
ARG APP_PROP
ARG APP_LIB
ARG JAR_FILE
RUN apk add --no-cache curl && apk add --no-cache git maven && \
    git clone https://github.com/ekoi/datatags-service /usr/src/datatags-service && \
    cd /usr/src/datatags-service && mvn clean install

ENTRYPOINT [ "sh", "-c", "cd /usr/src/datatags-service/target; java -Djava.security.egd=file:/dev/./urandom -jar ./datatags-service-1.0.0.jar" ]
