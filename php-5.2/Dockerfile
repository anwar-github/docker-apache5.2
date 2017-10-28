FROM centos:5.11

COPY ./yum.repos.d/ /etc/yum.repos.d/

ENV PHP_VERSION 5.2.17

ARG PUID=1000
ARG PGID=1000

ARG PHP_APACHE_HTTP_PORT=80
ARG PHP_APACHE_HTTPS_PORT=443

ENV PUID ${PUID}
ENV PGID ${PGID}

ENV PHP_APACHE_HTTP_PORT ${PHP_APACHE_HTTP_PORT}
ENV PHP_APACHE_HTTPS_PORT ${PHP_APACHE_HTTPS_PORT}

RUN groupadd -g ${PGID} apache && \
    useradd -u ${PUID} -g apache -m apache

# Initial setup
RUN yum update -y
RUN yum groupinstall -y 'Development Tools'

# Apache installation
RUN yum install -y httpd httpd-devel

# PHP 5.2 dependency installation
RUN yum install -y \
  libaio-devel \
  libmcrypt-devel \
  libjpeg-devel \
  libpng-devel \
  libxml2-devel \
  libxslt-devel \
  curl-devel \
  freetype-devel \
  gmp-devel \
  mysql-devel \
  openssl-devel \
  postgresql-devel \
  sqlite-devel
WORKDIR /usr/local/src
RUN ldconfig

# PHP 5.2 installation
COPY ./php5.2 /usr/local/src/
RUN tar xf ./php-${PHP_VERSION}.tar.bz2 -C ./
WORKDIR /usr/local/src/php-${PHP_VERSION}
RUN ./configure \
  --enable-gd-native-ttf \
  --enable-mbregex \
  --enable-mbstring \
  --enable-soap \
  --enable-zend-multibyte \
  --enable-zip \
  --with-apxs2 \
  --with-curl \
  --with-freetype-dir=/usr \
  --with-gd \
  --with-gettext \
  --with-gmp \
  --with-jpeg-dir=/usr \
  --with-mcrypt \
  --with-mysql-sock \
  --with-openssl \
  --with-pear \
  --with-pdo-mysql \
  --with-png-dir=/usr \
  --with-xsl \
  --with-zlib \
  --with-mysqli \
  --with-xmlrpc
RUN make && make test && make install

RUN sed -i "s/Listen 80/Listen ${PHP_APACHE_HTTP_PORT}/" /etc/httpd/conf/httpd.conf
RUN sed -i "s/Listen 443/Listen ${PHP_APACHE_HTTPS_PORT}/" /etc/httpd/conf/httpd.conf

# Apache setup and launching
COPY ./sites /etc/httpd/conf/sites-available/
COPY ./httpd.conf /etc/httpd/conf/sites-available/extra.conf
RUN echo 'Include /etc/httpd/conf/sites-available/*' >> /etc/httpd/conf/httpd.conf
EXPOSE ${PHP_APACHE_HTTP_PORT} ${PHP_APACHE_HTTPS_PORT}

VOLUME [ "/data", "/var/www/html" ]

CMD [ "/usr/sbin/httpd", "-D", "FOREGROUND" ]
