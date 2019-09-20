
# http://www.apache.org/licenses/LICENSE-2.0

FROM centos/httpd

LABEL maintainer="DANS <info@dataverse.nl>"
# RUN yum install -y sudo epel-release
# RUN yum install -y R-core R-core-devel git python3

RUN yum install -y git python3
RUN sed -i "s/80/8566/g" /etc/httpd/conf/httpd.conf
RUN mkdir -p /opt/dv && cd /opt/dv && git clone https://github.com/QualitativeDataRepository/dataverse-previewers
RUN cp -R /opt/dv/dataverse-previewers/previewers /var/www/html
RUN cp -R /opt/dv/dataverse-previewers /var/www/html
RUN cd /opt/dv && git clone https://github.com/scholarsportal/Dataverse-Data-Explorer.git
RUN cp -R /opt/dv/Dataverse-Data-Explorer /var/www/html

EXPOSE 8566