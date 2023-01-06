Based on https://hub.docker.com/_/httpd/
The "latest"-image is based on httpd:2.4-alpine

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
        image: oberonamsterdam/php:7.1-fpm
        restart: always
        network_mode: "bridge"
        volumes:
            - .:/app/:delegated
```

Switch PHP version by using another PHP image (exposed PHP-FPM on port 9000).

# Switch PHP container hostname or portnumber

You can use the PHP_FPM_HOSTNAME and PHP_FPM_PORT environment variables to change the hostname and
port Apache uses to connect to PHP-FPM:

```
version: '3'
services:
    web:
        ...SNIP
        environment:
            - "VIRTUAL_HOST=my.domain.ext"
            - "APACHE_DOCUMENTROOT=/app/web"
            - "PHP_FPM_HOSTNAME=my_php_container"
            - "PHP_FPM_PORT=9001"
```

# OPENID CONNECT

The "openidc"-image is based on httpd:2.4 (since the mod_auth_openidc is not available for Alpine)

Using the openidc-tagged image, use this for example to add an initial auth to your container (instead of htpasswd).
Complete manual of openidc config can be found here: https://github.com/zmartzone/mod_auth_openidc

This an example using google and your own domain (so only people from your own company can access this container)
```
LoadModule auth_openidc_module /usr/lib/apache2/modules/mod_auth_openidc.so

OIDCProviderMetadataURL https://accounts.google.com/.well-known/openid-configuration
OIDCClientID <<GOOGLE-CLIENT-ID>>
OIDCClientSecret <<GOOGLE-CLIENT-SECRET>>

OIDCAuthRequestParams hd=<<YOUR-DOMAIN>>
OIDCRedirectURI https://${VIRTUAL_HOST}/redirect
OIDCCryptoPassphrase <<YOUR-RANDOM-PASSPHRASE>>
OIDCRemoteUserClaim email
OIDCScope "openid email"
OIDCAuthNHeader X-Forwarded-User

<Location />
    AuthType openid-connect
    Require claim hd:<<YOUR-DOMAIN>>
    Require valid-user
</Location>
```

After this, add the config to the web-container:
```
version: '3'
services:
    web:
        image: oberonamsterdam/apache24-fpm:openidc
        volumes:
            - .:/app/:delegated
            - openidc.conf:/usr/local/apache/config/other/openidc.conf

```
