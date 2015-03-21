#Installation and config guide

##Installation
====
####1. Install mysql, nginx, php-fpm, memcache, php
---



    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install mysql-server mysql-client nginx memcached
    sudo apt-get install php5-fpm php5 php5-cli php5-curl php5-mcrypt php5-intl php5-mysql php5-mongo php5-redis php5-memcache php5-memcached php5-gd php-apc
    
####2. Install redis (optional)
---



    sudo apt-get install redis-server php5-redis

####3. Install mongodb (optional)
---



    sudo apt-get install mongodb php5-mongo

####4. Install tools and utility
---



    sudo apt-get install git htop curl
##Configuration
======
###User
====
####1. Create user `web`
---



    sudo mkdir /web
    sudo mkdir /web/www
    sudo useradd -d /web/www web
    sudo chown -R web:web /web/www
    
####2. Create project dirs
---


    sudo su web    
    mkdir /web/www/project.tld
    mkdir /web/www/project.tld/log
    mkdir /web/www/project.tld/html
### php-fpm
===
####1. Create php mods file `/etc/php5/mods-available/nginx.ini`
---


    cgi.fix_pathinfo= 0
    expose_php = Off
####2. Create php mods file `/etc/php5/mods-available/project.ini`
---


    memory_limit = 512M
    post_max_size = 1G
    upload_max_filesize = 1G
####3. Enable this mods
---


    sudo php5enmod nginx
    sudo php5enmod project

####4. Update file `/etc/php5/fpm/pool.d/www.conf` set
---


    user = web
    group = web
    listen.owner = web
    listen.group = web
####5. Restart php-fpm
---


    sudo service php5-fpm restart
###mysql
===
####1. Login to mysql console.
---


    mysql -u root -p
> Hint: for reset mysql root password use


    sudo dpkg-reconfigure mysql-server-5.5
####2. Create mysql project db and user
---
#####Create mysql project db
 
 
    CREATE DATABASE projectdatabase;
#####Create mysql project user


    CREATE USER 'projectuser'@'localhost' IDENTIFIED BY 'password';
#####Grant privileges
    
    
    GRANT ALL PRIVILEGES ON `projectdatabase`. * TO 'projectuser'@'localhost';
#####Reload all the privileges.
    
    
    
    FLUSH PRIVILEGES;
###nginx
===
####1. Update file `/etc/nginx/nginx.conf` set
---


    user web
####2. Disable default site
---


    sudo rm /etc/nginx/sites-enabled/default
    
####3. Create config helpers
---

#####- `/etc/nginx/conf.d/charset.conf`

```
#Specify a charset
charset utf-8;

client_max_body_size 128M;
server_tokens off;
```

#####- `/etc/nginx/conf.d/gzip.conf`

```
    # Gzip Settings
    #gzip on;               # enabled by default
    #gzip_disable "msie6";  # enabled by default
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```

#####- `/etc/nginx/expires.conf`

```
    # Expire rules for static content
    
    # No default expire rule. This config mirrors that of apache as outlined in the
    # html5-boilerplate .htaccess file. However, nginx applies rules by location,
    # the apache rules are defined by type. A consequence of this difference is that
    # if you use no file extension in the url and serve html, with apache you get an
    # expire time of 0s, with nginx you'd get an expire header of one month in the
    # future (if the default expire rule is 1 month). Therefore, do not use a
    # default expire rule with nginx unless your site is completely static
    
    # cache.appcache, your document html and data
    location ~* \.(?:manifest|appcache|html?|xml|json)$ {
      expires -1;
      #access_log logs/static.log;
    }
    
    # Feed
    location ~* \.(?:rss|atom)$ {
      expires 1h;
      add_header Cache-Control "public";
    }
    
    # Media: images, icons, video, audio, HTC
    location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc|webp|mp3)$ {
      expires 1M;
      access_log off;
      add_header Cache-Control "public";
    }
    
    # CSS and Javascript
    location ~* \.(?:css|js)$ {
      expires 1y;
      access_log off;
      add_header Cache-Control "public";
    }
    # favicon disable logs
    location = /favicon.ico {
      access_log off;
      log_not_found off;
    }
```

#####- `/etc/nginx/cross-domain-fonts.conf`

```
    # Cross domain webfont access
    location ~* \.(?:ttf|ttc|otf|eot|woff|woff2)$ {
        # Also, set cache rules for webfonts.
        #
        # See http://wiki.nginx.org/HttpCoreModule#location
        # And https://github.com/h5bp/server-configs/issues/85
        # And https://github.com/h5bp/server-configs/issues/86
        expires 1M;
        access_log off;
        add_header Cache-Control "public";
    }
```

#####- `/etc/nginx/protect-system-files.conf`

```
    # Prevent clients from accessing hidden files (starting with a dot)
    # This is particularly important if you store .htpasswd files in the site hierarchy
    location ~* (?:^|/)\. {
        deny all;
    }
    
    # Prevent clients from accessing to backup/config/source files
    location ~* (?:\.(?:bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist|md)|~)$ {
        deny all;
    }
```

####4. Create project site config `/etc/nginx/sites-available/project.tld`
---

```
    ##
    # For new site create folders newsite.com/log and newsite.com/html at /web/www
    # and replace project.tld with newsite.com in this file
    #
    # For product environment uncomment include expires.conf
    ##
    
    server {
            server_name project.tld;
            listen 80;
    
            access_log    /web/www/project.tld/log/access.log combined buffer=50k;
            error_log     /web/www/project.tld/log/error.log notice;
    
            set           $host_path      /web/www/project.tld/html;
            set           $yii_bootstrap  index.php;
    
            index         $yii_bootstrap;
            root          $host_path/frontend/web;
    
            location / {
                    try_files $uri $uri/ /$yii_bootstrap$is_args$args;
            }
    
            location ~ \.php$ {
                    try_files $uri =404;
    
                    fastcgi_split_path_info ^(.+\.php)(/.+)$;
                    fastcgi_index           $yii_bootstrap;
    
                    # Connect to php-fpm via socket
                    fastcgi_pass unix:/var/run/php5-fpm.sock;
    
                    fastcgi_connect_timeout     30s;
                    fastcgi_read_timeout        30s;
                    fastcgi_send_timeout        60s;
                    fastcgi_ignore_client_abort on;
                    fastcgi_pass_header         "X-Accel-Expires";
    
                    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                    fastcgi_param  PATH_INFO        $fastcgi_path_info;
                    fastcgi_param  HTTP_REFERER     $http_referer;
                    include fastcgi_params;
            }
            
            # You can add backend here 
            # location /admin {
            #        root          $host_path/backend/web;
            #        try_files $uri $uri/ /$yii_bootstrap$is_args$args;
            # }
            
            include cross-domain-fonts.conf;
            include protect-system-files.conf;
            
            # Uncomment this on product environment
            #include expires.conf;
    }
    
    server {
            server_name backend.project.tld;
            listen 80;
    
            access_log    /web/www/project.tld/log/access.backend.log combined buffer=50k;
            error_log     /web/www/project.tld/log/error.backend.log notice;
    
            set           $host_path      /web/www/project.tld/html;
            set           $yii_bootstrap  index.php;
    
            index         $yii_bootstrap;
            root          $host_path/backend/web;
    
            location / {
                    try_files $uri $uri/ /$yii_bootstrap$is_args$args;
            }
    
            location ~ \.php$ {
                    try_files $uri =404;
    
                    fastcgi_split_path_info ^(.+\.php)(/.+)$;
                    fastcgi_index           $yii_bootstrap;
    
                    # Connect to php-fpm via socket
                    fastcgi_pass unix:/var/run/php5-fpm.sock;
    
                    fastcgi_connect_timeout     30s;
                    fastcgi_read_timeout        30s;
                    fastcgi_send_timeout        60s;
                    fastcgi_ignore_client_abort on;
                    fastcgi_pass_header         "X-Accel-Expires";
    
                    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                    fastcgi_param  PATH_INFO        $fastcgi_path_info;
                    fastcgi_param  HTTP_REFERER     $http_referer;
                    include fastcgi_params;
            }
            
            include cross-domain-fonts.conf;
            include protect-system-files.conf;
            
            # Uncomment this on product environment
            #include expires.conf;
    }
```

####5. Enable project.tld site config
---


    sudo ln -s /etc/nginx/sites-available/project.tld /etc/nginx/sites-enabled/
####6. Check config
---


    sudo nginx -t

####7. Restart nginx
---


    sudo service nginx restart

###Clone project
===
####1. Generate deployment ssh key
---


    ssh-keygen -t rsa -C "web@project.tld"
 and add `project.tld` to the repo deployment key


    cat ~/.ssh/id_rsa

####2. Clone project
---


    sudo su web
    cd ~/project.tld/
    git clone git@bitbucket.org:gitaccount/project.git html
####3. Install composer.phar
---


    cd ~/project.tld/html/
    curl -s http://getcomposer.org/installer | php
####4. Add composer assets plugin
---


    php composer.phar global require "fxp/composer-asset-plugin:@dev"
####5. Install packages and init project environment
---


    cd ~/project.tld/html/
    php composer.phar install
    ./init
####6. Update config/main-local.php files
---

####7. Run migrations
---


    ./yii migrate
