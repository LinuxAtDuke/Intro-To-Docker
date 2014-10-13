# This is a basic webserver image, for
# Linux@Duke's Introduction To Docker: Do Your Own Docker class

FROM centos:centos6
MAINTAINER Chris Collins <collins.christopher@gmail.com>

RUN yum install -y httpd git
RUN yum clean all

ADD https://upload.wikimedia.org/wikipedia/commons/e/ee/Grumpy_Cat_by_Gage_Skidmore.jpg /var/www/html/grumpycat.jpg

VOLUME ["/var/www/html"]

EXPOSE 80

CMD ["/usr/sbin/httpd", "-DFOREGROUND"]

