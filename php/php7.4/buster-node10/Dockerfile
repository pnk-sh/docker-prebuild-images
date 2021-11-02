FROM pnksh/php:php7.4-buster
LABEL maintainer="hi@pnk.dk"

RUN apt install npm -y

EXPOSE 8080

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping