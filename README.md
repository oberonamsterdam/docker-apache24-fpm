Based on https://hub.docker.com/_/httpd/

Default Oberon apache setup using PHP-FPM, requires a PHP-FPM docker image: official ones from PHP or the custom build one in our other repository.
Note: If you don't need PHP (like only serving plain html) then we recommend using the vanilla HTTPD container or something lightweight like nginx/lighttpd.

# This image has the following modules enabled:
* mod_alias
* mod_rewrite
* mod_proxy_wstunnel
* mod_proxy_fcgi
* mod_proxy

# Additional configurations:
* mod_expires - with 1 month on image/css/javascript assets additional minetypes for svg and fonts
* mod_deflate - on text assets (html, css, js, svg, xml, plain) production settings on signatures

# Adding your own configurations:
Add your own configurations by adding entries to the web-container volumes under "/usr/local/apache2/conf/other/":
```
    web:
        volumes:
            - myconfiguration.conf:/usr/local/apache2/conf/other/myconfiguration.conf
```

Note that HTTPS is disabled since we use an Nginx proxy ourselves so don't forget to enable that when you need it.

# Example docker-compose.yml:

```
version: '3'
services:
    web:
        container_name: "web-container"
        image: oberonamsterdam/apache24-fpm:latest
        restart: always
        network_mode: "bridge"
        environment:
            - "VIRTUAL_HOST=my.domain.ext"
            - "APACHE_DOCUMENTROOT=/app/web"
        links:
            - php
        depends_on:
            - php
        volumes:
            - .:/app/:delegated
    php:
        container_name: "php-container"
        image: jiechin/php:7.1-fpm
        restart: always
        network_mode: "bridge"
        volumes:
            - .:/app/:delegated
```

Switch PHP version by using another PHP image (exposed PHP-FPM on port 9000).