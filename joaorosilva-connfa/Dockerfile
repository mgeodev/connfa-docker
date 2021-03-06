FROM library/ubuntu:16.04

ENV INSTALL_DIR /var/www/laravel
ENV CONNFA_VERSION "2.0.1"

## for apt to be noninteractive
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

# expose port
EXPOSE 8000

#install tzdata unattended
## preesed tzdata, update package index, upgrade packages and install needed software
RUN echo "tzdata tzdata/Areas select Europe" > /tmp/preseed.txt; \
    echo "tzdata tzdata/Zones/Europe select Warsaw" >> /tmp/preseed.txt; \
    debconf-set-selections /tmp/preseed.txt && \
    apt-get update && \
    apt-get install -y tzdata

## cleanup of files from setup
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# install PHP and laravel dependencies
RUN apt-get update
RUN apt-get install -y mysql-server-5.7 mysql-client-5.7

# Change this timezone if you need 
RUN unlink /etc/localtime
RUN echo "Europe/Warsaw" > /etc/timezone 

#Now we can install the packages unattended
RUN dpkg-reconfigure -f noninteractive tzdata
RUN apt-get install -y curl unzip
RUN apt-get install -y php php7.0-curl php7.0-mbstring php7.0-json php7.0-fpm php7.0-xmlrpc php7.0-soap php7.0-gd php7.0-xml php7.0-cli php7.0-zip php7.0-mysql

#Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

#Now we can install Connfa!
WORKDIR $INSTALL_DIR

# delete files
RUN rm -rf ./app ./vendor ./public ./config ./Parser ./resources

#copy cloned app
#ADD ./connfa $INSTALL_DIR
# Clone the conf files into the docker container
# COPY git clone https://github.com/lemberg/connfa-integration-server.git $INSTALL_DIR
# RUN curl -SL https://github.com/lemberg/connfa-integration-server/archive/master.zip | tar -xJC $INSTALL_DIR
ADD https://github.com/lemberg/connfa-integration-server/archive/master.zip /tmp
RUN unzip /tmp/master.zip connfa-integration-server-master/* -d $INSTALL_DIR \
 && mv $INSTALL_DIR/connfa-integration-server-master/* $INSTALL_DIR \
 && rm -rf $INSTALL_DIR/connfa-integration-server-master \
 && rm /tmp/master.zip
 
COPY .env $INSTALL_DIR
COPY run.sh $INSTALL_DIR

# install dependencies
RUN composer update

# set permissions
RUN chmod a+rw $INSTALL_DIR
RUN chmod +x $INSTALL_DIR/run.sh

# Start app
#ENTRYPOINT php artisan serve --host=0.0.0.0 --port=8000
ENTRYPOINT "$INSTALL_DIR/run.sh" 
#CMD ["/bin/sh", "/var/www/laravel/run.sh"]
