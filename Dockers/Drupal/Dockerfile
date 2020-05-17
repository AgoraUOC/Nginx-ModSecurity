FROM drupal:8.5.0-apache

LABEL version="1.0.0"
LABEL maintainer="Manuel Nader <mnader@uoc.edu>"

WORKDIR /var/www/html

RUN mkdir /tmp/drupal && mv /var/www/html /tmp/drupal/ \
    && mkdir /var/www/html/ && mv /tmp/drupal/html/ /var/www/html/drupal
RUN rm -r /tmp/drupal/

# TODO: Automate installation

# Copy whole site
#COPY files /tmp
#COPY files/drupal.tar.gz drupal.tar.gz
#RUN tar -C /var/www/html -zxvf drupal.tar.gz
# RUN rm drupal.tar.gz

# Copy SQLite file
#COPY files/sqlite sites/default/files/.ht.sqlite

# Copy config file
#COPY files/settings.php sites/default/settings.php
#RUN chmod a+w sites/default/settings.php
#RUN chmod a+w sites/default

# ENTRYPOINT ["docker-php-entrypoint"]
# CMD ["nginx", "-g", "daemon off;"]
