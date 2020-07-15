FROM openjdk:8-jdk-alpine
VOLUME /tmp
WORKDIR /usr/src
ARG APP_BASE_DIR=/usr/local/sword3
ARG APP_PROP
ARG APP_LIB
ARG JAR_FILE
RUN apk add --no-cache curl && apk add --no-cache git maven && \
    git clone https://github.com/ekoi/sword3-server-prototype /usr/src/sword3-server-prototype && \
    cd /usr/src/sword3-server-prototype && mvn clean install

ENTRYPOINT [ "sh", "-c", "cd /usr/src/sword3-server-prototype/target; java -Djava.security.egd=file:/dev/./urandom -jar ./sword3-server-prototype-1.0.0.jar" ]
