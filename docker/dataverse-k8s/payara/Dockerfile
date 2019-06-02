# Copyright 2019 Forschungszentrum Jülich GmbH
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# http://www.apache.org/licenses/LICENSE-2.0

FROM payara/server-full:5.192

LABEL maintainer="FDM FZJ <forschungsdaten@fz-juelich.de>"

ARG VERSION=4.14

ENV DATA_DIR=/data\
    SECRETS_DIR=/secrets\
    PGDRIVER_PKG=https://jdbc.postgresql.org/download/postgresql-42.2.5.jar\
    DATAVERSE_VERSION=${VERSION}\
    DATAVERSE_PKG=https://github.com/IQSS/dataverse/releases/download/v${VERSION}/dvinstall.zip

# Create basic pathes
USER root
RUN mkdir -p ${DATA_DIR} ${SECRETS_DIR} && \
    chown -R payara: ${DATA_DIR} ${SECRETS_DIR}

# Install prerequisites
RUN apt-get -qq update && \
    apt-get -qqy install postgresql-client jq imagemagick

# Install PostgreSQL JDBC driver in AppServer
# TODO: remove this once upstream includes the Postgres Client lib in the WAR.
USER payara
RUN wget --no-verbose -O postgresql.jar ${PGDRIVER_PKG} && \
    mv postgresql.jar ${PAYARA_DIR}/glassfish/lib

# Retrieve the Dataverse install package, extract and remove ZIP
RUN wget --no-verbose -O dvinstall.zip ${DATAVERSE_PKG} && \
    unzip -qq dvinstall.zip -d ${HOME_DIR}/ && \
    rm dvinstall.zip && \
    ln -s ${HOME_DIR}/dvinstall/dataverse.war ${DEPLOY_DIR}/dataverse.war

# Copy across docker scripts
COPY --chown=payara:payara bin/*.sh bin/default.config payara/bin/*.sh ${SCRIPT_DIR}/
RUN chmod +x ${SCRIPT_DIR}/*
