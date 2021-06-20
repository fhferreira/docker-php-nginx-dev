# PHP 7.4 FPM + NGINX Dev Enviroment

Image composition and versions below:

- PHP FPM 7.4
- XDebug 3.0.2
- NGINX 1.18.0-r15
- Composer 2.1.3
- Git 2.30.1-r0
- Redis 5.3.4
- MongoDB 2.30.1-r0

## Docker Compose

### Minimal to use

The snipet below show the **minimal setup**:

```yaml
version: '3.5'
services:
  app:
    container_name: project-app
    image: dersonsena/php-nginx-dev
    volumes:
      - ./:/var/www/html
    ports:
      - '80:80'
      - '443:443'
    networks:
      - project-network

networks:
  project-network:
    driver: bridge
```

> **IMPORTANT**: You must be a `/public` because this folder is the Nginx Document Root. If your project doesn't have it, you can create a symbolic link: `ln -fs ./your-document-root ./public`

### PHP

You can custumize some php ini params. The posibles variables and their default values are shown below:

```yaml
version: '3.5'
services:
  app:
    container_name: project-app
    image: dersonsena/php-nginx-dev
    volumes:
      - ./:/var/www/html
    ports:
      - '80:80'
      - '443:443'
    environment:
      - PHP_DATE_TIMEZONE=America/Sao_Paulo
      - PHP_DISPLAY_ERRORS=On
      - PHP_MEMORY_LIMIT=256M
      - PHP_MAX_EXECUTION_TIME=60
      - PHP_POST_MAX_SIZE=50M
      - PHP_UPLOAD_MAX_FILESIZE=50M
    networks:
      - project-network

networks:
  project-network:
    driver: bridge
```

If you want to use a custom `php.ini` file, just create one in your project, such as `.docker/php/php.ini`, and map volume like below:

```yaml
...
volumes:
  - ./:/var/www/html
  - ./.docker/php:/usr/local/etc/php
...
```

> **TIP**: to make your life easier you can use the original php.ini file placed in root level of this repository.

(BETA) You can change the PHP version (see [PHP Supported Versions](https://www.php.net/supported-versions.php)). Just set the arg `PHP_VERSION`. The default version is `7.4.20`:

```yml
...
build:
  args:
    PHP_VERSION: 8.0.7
...
```

### XDebug

You can customize some xdebug params. The possibles variables and their default values are shown below:

```yaml
version: '3.5'
services:
  app:
    container_name: project-app
    image: dersonsena/php-nginx-dev
    volumes:
      - ./:/var/www/html
    ports:
      - '80:80'
      - '443:443'
    environment:
      - XDEBUG_MODE=debug
      - XDEBUG_START_WITH_REQUEST=yes
      - XDEBUG_DISCOVER_CLIENT_HOST=false
      - XDEBUG_CLIENT_HOST=host.docker.internal
      - XDEBUG_CLIENT_PORT=9000
      - XDEBUG_MAX_NESTING_LEVEL=1500
      - XDEBUG_IDE_KEY=PHPSTORM
      - XDEBUG_LOG=/tmp/xdebug.log
    networks:
      - project-network

networks:
  project-network:
    driver: bridge
```

Just on **MacOs environment**, use this configurations to use xdebug:

```yaml
...
environment:
  - XDEBUG_START_WITH_REQUEST=yes
  - XDEBUG_DISCOVER_CLIENT_HOST=false
  - XDEBUG_CLIENT_PORT=<your-ide-port-here>
...
```

### NGINX

#### Change Document Root

You can change the nginx document root. If the project's document is `public_html`, for example, you can do:

```yaml
...
environment:
  - NGINX_DOCUMENT_ROOT=/var/www/html/public_html
...
```

#### SSL Certificate

This docker image already have a SSL certificate, but if you want to use a custom SSL certificates to NGINX web server, just create the files in folder in your project, such as `.docker/nginx/`:

- Private Key: `.docker/nginx/certificate.key`
- Certificate: `.docker/nginx/certificate.crt`

and place your certificate and key files there. After this, just map volumes as below:

```yaml
version: '3.5'
services:
  app:
    container_name: project-app
    image: dersonsena/php-nginx-dev
    volumes:
      - ./:/var/www/html
      - ./.docker/nginx/certs:/etc/nginx/certs
    ports:
      - '80:80'
      - '443:443'
    networks:
      - project-network

networks:
  project-network:
    driver: bridge
```