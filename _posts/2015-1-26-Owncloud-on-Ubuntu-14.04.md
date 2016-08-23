---
layout: post
title: Owncloud on ubuntu 14.04
---

The following instructions are for Owncloud 7 on Ubuntu 14.04.

First download and install owncloud from the website.

Make sure all necessary php5 packages are installed:

    sudo apt-get install php5-common php5-fpm php5-cli php5-json php5-mysql php5-curl php5-intl php5-mcrypt php5-memcache php-xml-parser php-pear

Next add the following file `owncloud` to nginx `sites-available` directory, also link to `sites-enabled` directory. This will only enable http access:

    upstream php-handler {
        server unix:/var/run/php5-fpm.sock;
    }

    server {
        listen 80;
        server_name cloud.example.com;

        # Path to the root of your installation
        root /var/www/owncloud/;
        # set max upload size
        client_max_body_size 10G;
        fastcgi_buffers 64 4K;

        rewrite ^/caldav(.*)$ /remote.php/caldav$1 redirect;
        rewrite ^/carddav(.*)$ /remote.php/carddav$1 redirect;
        rewrite ^/webdav(.*)$ /remote.php/webdav$1 redirect;

        index index.php;
        error_page 403 /core/templates/403.php;
        error_page 404 /core/templates/404.php;

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }

        location ~ ^/(?:\.htaccess|data|config|db_structure\.xml|README){
            deny all;
        }

        location / {
            # The following 2 rules are only needed with webfinger
            rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
            rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

            rewrite ^/.well-known/carddav /remote.php/carddav/ redirect;
            rewrite ^/.well-known/caldav /remote.php/caldav/ redirect;

            rewrite ^(/core/doc/[^\/]+/)$ $1/index.html;

            try_files $uri $uri/ /index.php;
        }

        location ~ \.php(?:$|/) {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
            #fastcgi_param HTTPS on;
            fastcgi_pass php-handler;
        }

        # Optional: set long EXPIRES header on static assets
        location ~* \.(?:jpg|jpeg|gif|bmp|ico|png|css|js|swf)$ {
            expires 30d;
            # Optional: Don't log access to assets
            access_log off;
        }
    }

Restart nginx and php5-fpm

    sudo service php5-fpm restart && sudo service nginx restart

Make sure that `/etc/hosts` has the appropriate entries. For example, this example uses the domain name `cloud.example.com` so an entry corresponding entry is needed.

# MySQL

Next setup MySQL with the appropriate settings:

    sudo mysql -u root -p

Create a database for owncloud,

    CREATE DATABASE owncloud;

Assign privileges to a new MySQL user to handle database operations for ownCloud:

    GRANT ALL ON owncloud.* to 'owncloud'@'localhost' IDENTIFIED BY 'database_password';
exit;

Next goto cloud.example.com and use the MySQL settings when completing ownCloud server setup.
HTTPS

To start using HTTPS first create the ssl certificates,

    sudo mkdir /etc/nginx/ssl
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/owncloud.key -out /etc/nginx/ssl/owncloud.crt

and make sure to specify the `Common Name`.

Then make the following changes to the `owncloud` nginx config file

    server {
        listen 80;
        server_name cloud.example.com;
        # enforce https
        return 301 https://$server_name$request_uri;
    }

    ....

    server {
        listen 443 ssl;
        server_name cloud.example.com;

        ssl_certificate /etc/nginx/ssl/nginx/owncloud.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx/owncloud.key;

        ....
        ....

        fastcgi_param HTTPS on;
    }

Then restart nginx and it should work!

*Links*

http://www.surject.com/setup-owncloud-7-server-nginx-ubuntu/
http://doc.owncloud.org/server/7.0/admin_manual/installation/nginx_configuration.html
https://www.howtoforge.com/how-to-install-owncloud-7-on-ubuntu-14.04
