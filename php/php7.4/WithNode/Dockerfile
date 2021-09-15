FROM pnksh/php-alpine:php7.4
LABEL maintainer="hi@pnk.dk"

# Install Node + NPM
RUN apk add npm
RUN node -v

EXPOSE 8080

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping