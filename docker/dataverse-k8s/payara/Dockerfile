# Copyright 2019 Forschungszentrum Jülich GmbH
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# http://www.apache.org/licenses/LICENSE-2.0

FROM centos:7

LABEL maintainer="FDM FZJ <forschungsdaten@fz-juelich.de>"

ARG PAYARA_VERSION=5.201
ARG PAYARA_PKG=https://github.com/payara/Payara/releases/download/payara-server-5.2020.2/payara-5.2020.2.zip
ARG PAYARA_SHA1=ea86d69233826b4d35612260ea4e8f81a9b992f2
ARG TINI_VERSION=v0.18.0
ARG JMX_EXPORTER_VERSION=0.12.0
ARG VERSION=4.20
#ENV AS_ADMIN_NEWPASSWORD secret
# Initialize the configurable environment variables
ENV HOME_DIR=/opt/dataverse
ENV PAYARA_DIR=${HOME_DIR}/appserver\
    SCRIPT_DIR=${HOME_DIR}/scripts\
    CONFIG_DIR=${HOME_DIR}/config\
    SECRETS_DIR=/secrets\
    DATA_DIR=/data\
    DOCROOT_DIR=/docroot\
    METADATA_DIR=/metadata\
    # Payara Server Domain options
    DOMAIN_NAME=domain1\
    PGDRIVER_PKG=https://jdbc.postgresql.org/download/postgresql-42.2.10.jar\
    JMX_EXPORTER_PKG=https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_EXPORTER_VERSION}/jmx_prometheus_javaagent-${JMX_EXPORTER_VERSION}.jar\
    DATAVERSE_PKG=https://github.com/IQSS/dataverse/releases/download/v${VERSION}/dvinstall.zip\
    WELD_PKG=https://search.maven.org/remotecontent?filepath=org/jboss/weld/weld-osgi-bundle/2.2.10.Final/weld-osgi-bundle-2.2.10.Final-glassfish4.jar\
    GRIZZLY_PKG=http://guides.dataverse.org/en/latest/_static/installation/files/issues/2180/grizzly-patch/glassfish-grizzly-extra-all.jar\
    AS_ADMIN_NEWPASSWORD=admin\
    # Utility environment variables
    JVM_ARGS=\
    PAYARA_ARGS=\
    DEPLOY_PROPS=\
    POSTBOOT_COMMANDS=/opt/dataverse/config/post-boot-commands.asadmin\
    PREBOOT_COMMANDS=/opt/dataverse/config/pre-boot-commands.asadmin
ENV DOMAIN_DIR=${PAYARA_DIR}/glassfish/domains/${DOMAIN_NAME}
ENV PATH="${PATH}:${PAYARA_DIR}/bin"

# Create and set the Payara user and working directory owned by the new user
RUN groupadd -g 1000 payara && \
    useradd -u 1000 -M -s /bin/bash -d ${HOME_DIR} payara -g payara && \
    echo payara:payara | chpasswd && \
#    mkdir -p ${DOMAIN_DIR} && \
    mkdir -p ${CONFIG_DIR} && \
    mkdir -p ${SCRIPT_DIR} && \
    mkdir -p ${HOME_DIR} ${SCRIPT_DIR} ${SECRETS_DIR} && \
    mkdir -p ${DATA_DIR} ${METADATA_DIR} ${DOCROOT_DIR} && \
    chown -R payara: ${HOME_DIR} ${DATA_DIR} ${METADATA_DIR} ${DOCROOT_DIR}

 # Install prerequisites
RUN yum install -y java-1.8.0-openjdk-headless epel-release zip unzip curl git maven less mlocate telnet wget && \
    yum install -y postgresql jq ImageMagick && \
    yum clean all

RUN echo "-----------------"
# Install tini as minimized init system
RUN wget --no-verbose -O /tini https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini && \
    wget --no-verbose -O /tini.asc https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini.asc && \
    gpg --batch --keyserver "hkp://p80.pool.sks-keyservers.net:80" --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 && \
    gpg --batch --verify /tini.asc /tini && \
    chmod +x /tini

USER payara
WORKDIR ${HOME_DIR}

# Download and unzip the Payara distribution
RUN wget --no-verbose -O payara.zip ${PAYARA_PKG} && \
#    echo "${PAYARA_SHA1} *payara.zip" | sha1sum -c - && \
    unzip -qq payara.zip -d ./ && \
    mv payara*/ ${PAYARA_DIR} && \
#    cd ${PAYARA_DIR}/bin/ && \
    asadmin start-domain -d && \
    echo "AS_ADMIN_PASSWORD=">/tmp/pwd.txt && \
    echo "AS_ADMIN_NEWPASSWORD=${AS_ADMIN_NEWPASSWORD}">>/tmp/pwd.txt && \
    cat /tmp/pwd.txt && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt change-admin-password && \
    echo "AS_ADMIN_PASSWORD=${AS_ADMIN_NEWPASSWORD}">/tmp/pwd.txt && \
    cat /tmp/pwd.txt && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt enable-secure-admin && \
    echo "https://stackoverflow.com/questions/46334485/payara4-jmx-configuration-using-asadmin" && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt set configs.config.default-config.admin-service.jmx-connector.system.address=127.0.0.1 && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt set configs.config.default-config.admin-service.jmx-connector.system.security-enabled=false && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt create-jvm-options "\-Ddataverse.files.file.type=file" && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt create-jvm-options "\-Ddataverse.files.file.label=file" && \
    asadmin --host localhost --port 4848 --user admin --passwordfile /tmp/pwd.txt create-jvm-options "\-Ddataverse.files.file.directory=/data" && \
    for MEMORY_JVM_OPTION in $(asadmin --user=admin --passwordfile /tmp/pwd.txt list-jvm-options | grep "Xm[sx]"); do\
        asadmin --user=admin --passwordfile /tmp/pwd.txt delete-jvm-options $MEMORY_JVM_OPTION;\
        done && \
#    rm -f /tmp/pwd.txt && \
    asadmin stop-domain && \
    mkdir -p ${DOMAIN_DIR}/autodeploy && \
    rm -rf \
       payara.zip
#
#RUN wget --no-verbose -O weld-osgi-bundle.jar ${WELD_PKG} && \
#    mv weld-osgi-bundle.jar ${PAYARA_DIR}/glassfish/modules && \
#    wget --no-verbose -O glassfish-grizzly-extra-all.jar ${GRIZZLY_PKG} && \
#    mv glassfish-grizzly-extra-all.jar ${PAYARA_DIR}/glassfish/modules && \
#    cp /etc/pki/ca-trust/extracted/java/cacerts ${PAYARA_DIR}/glassfish/domains/domain1/config/cacerts.jks
#
#

RUN rm -rf ${DOMAIN_DIR}/docroot && \
    ln -s ${DOCROOT_DIR} ${DOMAIN_DIR}/docroot && \
    mkdir -p ${DOMAIN_DIR}/generated/jsp/dataverse

# Install PostgreSQL JDBC driver in AppServer
RUN wget --no-verbose -O postgresql.jar ${PGDRIVER_PKG} && \
    mv postgresql.jar ${PAYARA_DIR}/glassfish/lib

# Get Prometheus JMX Exporter Java Agent (see https://github.com/prometheus/jmx_exporter)
RUN wget --no-verbose -O ${HOME_DIR}/jmx_exporter_agent.jar ${JMX_EXPORTER_PKG} && \
    echo -e "---\n{}" > ${HOME_DIR}/jmx_exporter_config.yaml

# Retrieve the Dataverse install package, extract and remove ZIP,
#   remove Harvard custom metadatablocks
RUN wget --no-verbose -O dvinstall.zip ${DATAVERSE_PKG} && \
    unzip -qq dvinstall.zip -d ./ && \
    rm dvinstall.zip && \
    find . -iname "custom*.tsv" -delete

RUN git clone https://github.com/ekoi/dataverse /tmp/v4.20-sshoc && \
cd /tmp/v4.20-sshoc; git fetch; git pull origin v4.20-sshoc; mvn install -DskipTests && \
rm ${HOME_DIR}/dvinstall/dataverse.war && \
mv /tmp/v4.20-sshoc/target/dataverse-4.20.war ${HOME_DIR}/dvinstall/dataverse.war

# Copy across docker scripts
RUN mkdir -p ${SCRIPT_DIR}/api && echo "supersecret" > ${SCRIPT_DIR}/api/key

# Copy across docker scripts
COPY --chown=payara:payara docker/dataverse-k8s/bin/* docker/dataverse-k8s/payara/bin/* ${SCRIPT_DIR}/
RUN mkdir -p ${SCRIPT_DIR}/init.d && \
    chmod +x ${SCRIPT_DIR}/*

RUN cat  /tmp/pwd.txt

ENTRYPOINT ["/tini", "--"]
#CMD ${SCRIPT_DIR}/entrypoint.sh
CMD ["scripts/entrypoint.sh"]
#
#CMD ["tail", "-f", "/dev/null"]