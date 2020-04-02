FROM python:3.7-alpine

#RUN mkdir -p /usr/src/app && mkdir /usr/src/temp
WORKDIR /usr/src
ENV FLASK_ENV=development

RUN apk add --no-cache curl && apk add --no-cache git && git clone https://github.com/ekoi/speeltuin.git /usr/src/app && \
    pip3 install --no-cache-dir requests flask-debugtoolbar flask_cors



EXPOSE 9266

#ENTRYPOINT ["python3"]

CMD ["python", "/usr/src/app/python-scripts/cvm.py"]