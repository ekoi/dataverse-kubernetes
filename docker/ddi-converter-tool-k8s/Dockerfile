FROM python:3.7-alpine

RUN mkdir -p /usr/src/app && mkdir /usr/src/temp
WORKDIR /usr/src/app/ddi-converter-tool
ENV FLASK_ENV=development

RUN apk add --no-cache curl && apk add --no-cache git && git clone https://github.com/ekoi/ddi-converter-tool.git /usr/src/app/ddi-converter-tool && \
    apk add --no-cache --virtual .build-deps gcc libc-dev libxslt-dev && \
    apk add --no-cache libxslt && \
    pip3 install -r /usr/src/app/ddi-converter-tool/requirements.txt && \
    pip3 install --no-cache-dir lxml connexion[swagger-ui] flask-debugtoolbar flask_cors lxml xmltodict pyDataverse && \
    apk del .build-deps



COPY config.py /usr/src/app/ddi-converter-tool/dct_server

EXPOSE 8520

ENTRYPOINT ["python3"]

CMD ["-m", "dct_server"]