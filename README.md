# Lumen Base Image

This image is a base imagem for Lumen application served by Apache with mod_php running **php 7**.

![logo PHP](php-logo.png) ![logo Lumen](lumen-logo.png) ![Apache](httpd-logo.png)

[PHP][1]
[Lumen][2]
[Apache][3]

# Supported tags and respective `Dockerfile` links

* [`5-onbuild` (*5-onbuild/Dockerfile*)](5-onbuild/Dockerfile)

# How to use this image

## Create a `Dockerfile` in your Lumen project:

```dockerfile
FROM bfgasparin/lumen-apache:5-onbuild
```

Put this file in the root of your app, next to the `composer.json`.

This image includes the minimum packages for a Lumen project to work. It do not include database
libraries or memcache libraries, for example. It's up to you to install these libraries. For
example, if you are going to use PostgreSQL as database, you will need to install the libraries
and binaries to communicate to PostgreSQL as well as the postgres php extension.
You can do it in your `Dockerfile`:

```dockerfile
FROM bfgasparin/lumen-apache:5-onbuild

# Install postgres libraries and headers for C language
RUN apt-get update && apt-get install -y \
        libpq-dev \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*

# Install postgres php extension
RUN docker-php-ext-install \
        pdo_pgsql \
        pgsql
```

This image includes multiple `ONBUILD` triggers which should cover most Lumen applications.
The build will:

* `ONBUILD COPY composer.json composer.lock /var/www/html`
* `ONBUILD RUN composer install --prefer-dist --optimize-autoloader --no-dev --profile -vvv`
* `ONBUILD COPY . /var/www/html`
* `ONBUILD RUN rm -Rf tests/`
* `ONBUILD RUN chown -R www-data:www-data /var/www/html/storage/`
* `ONBUILD VOLUME /var/www/html/storage`


The build also sets the default command to `apache2-foreground` to start apache2 service.

You can now build and run the Docker image:

```console
$ docker build -t my-lumen-app .
$ docker run --name some-lumen-app -d my-lumen-app
```

You can test it by visiting `http://container-ip` in a browser or, if you need access outside
the daemon's host, on port 8080:

    docker run --name some-lumen-app -p 8080:80 -d my-lumen-app

You can then go to `http://localhost:8080` or `http://host-ip:8080` in a browser.

### How to install more PHP extensions

This images extends the [php:7-apache Oficial Image][3], so it provides two convenient scripts 
named `docker-php-ext-configure` and `docker-php-ext-install`, 
you can use them to easily install PHP extension. See [PHP Oficial Image Repository][3] for 
more details.

### Apache

Apache is configured to provide `Authorization header` to the request.

Apache is also configured to disable .htaccess support on the application directory
to increase its performance.

## PHP

PHP is configured to run optimized (for this image to be used in production). You can change php
configuration adding a custom `php.ini` configuration. `COPY` it into `/usr/local/etc/php` by
adding one more line to your Dockerfile:

    FROM bfgasparin/symfony-apache:5-onbuild
    COPY config/php.ini /usr/local/etc/php

## Configuring Lumen

Lumen is configured using environment variable. So to configure your Lumen application, you can use
docker:

```console
$ docker run --name some-lumen-app -e "APP_DEBUG=false" -e "DB_CONNECTION=pgsql" -p 8080:80 -d my-lumen-app
```

## Development

When using this image for development, its a good practice to mount the application source directory
from your host into the docker container. When you do that, you may probably overwrite the
vendors packages.

If it's yours first time, you will probably need to install the application vendors again:

```console
$ docker exec -u Lumen my-lumen-app composer install --prefer-dist
```

### How to configure PHP for development

This image comes with php-fpm configured to be used in production environment, containing settings which hold 
security, performance and best practices. To customize the php behavior for development environment, `COPY`
 a custom `php.ini` configuration into `/usr/local/etc/php` (you can use the `$PHP_INI_DIR` environment variable) 
 by adding one more line to the Dockerfile above and running the same commands to build and run:

```dockerfile
FROM bfgasparin/lumen-apache:5-onbuild
COPY config/php.ini $PHP_INI_DIR
```

Where `config/` contains your `php.ini` file.

# Useful Links

- [PHP Oficial Image Repository][3]

[1]: http://http://php.net/
[2]: https://lumen.laravel.com
[3]: http://httpd.apache.org
