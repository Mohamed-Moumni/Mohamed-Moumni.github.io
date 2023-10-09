# WORDPRESS

**What is WordPress?**

> WordPress is a content management system (CMS) used to create websites and blogs. It is based on PHP and MySQL, and it is known for its ease of use and flexibility. WordPress is open-source software, which means that it is free to use, modify, and distribute. It is the most popular CMS in the world, and it is used by millions of websites. It allows users to create and manage content, add media, and install plugins and themes to customize the look and functionality of their website.
> 

For this service I install WordPress in the container using WP CLI (WordPress command line interface), I created a script that automates that, and also created two users, one is the administrator and one has the author role, and also set up the red is cache and enable it, all of that is described in the script.

```
#!/bin/bash

#curl is a command-line tool that is used to transfer data from or to a server. it support various protocols like HTTP, HTTPS, FTP,
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar  

# change the owner of the directory of /var/www/html and all subdirectories
chown -R www-data:www-data /var/www/html/

#changing the owner of /var/www/html/ to 755 
chmod -R 755 /var/www/html/ 

# is an archive that contains WP-CLI tool.
chmod +x wp-cli.phar

# move the wp-cli.phar to user-installed executable, so wp command-line will available system-wide.
mv wp-cli.phar /usr/local/bin/wp

# move to /var/www/html/ directory
cd /var/www/html/

# download wordpress using wp-CLI 
wp core download --allow-root

# create wp-config.php file
touch wp-config.php

# copy the default configuration to wp-config.php
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# change is modifying the unix socket used for the connection of PHP-FPM with the web server,
# from the default /run/php/php7.3-fpm.sock to TCP/IP port 9000 .
sed -i '36 s/\/run\/php\/php7.3-fpm.sock/9000/' /etc/php/7.3/fpm/pool.d/www.conf

# Set The Database that will be connected with wordpress
sed -i 's/database_name_here/'$MYSQL_DATABASE'/g' /var/www/html/wp-config.php

# Set the Username of The database
sed -i 's/username_here/'$MYSQL_USER'/g' /var/www/html/wp-config.php

# Set the Password
sed -i 's/password_here/'$MYSQL_PASSWORD'/g' /var/www/html/wp-config.php

# set The Hostname of the That base
sed -i 's/localhost/'$HOST'/g' /var/www/html/wp-config.php

wp config set FORCE_SSL_ADMIN 'false' --allow-root

# set Hostname of redis container
wp config set WP_REDIS_HOST 'redis' --allow-root

# set The Port of Redis, This command is also assuming that Redis is running and listen on port 6379,
wp config set WP_REDIS_PORT '6379' --allow-root

# The instruction "wp config set WP_CACHE 'true'" is a command that sets the value of the WP_CACHE constant in the WordPress configuration file to "true".
# This constant controls whether caching is enabled in WordPress or not.
wp config set WP_CACHE 'true' --allow-root

# instal the wordpress
wp core install --url=$DOMAIN_NAME --title="My Wordpress Site" --admin_user=$ADMIN_USER --admin_password=$ADMIN_PASSWORD --admin_email=$ADMIN_EMAIL --allow-root

# create second user in wordpress
wp user create $USER $USER_EMAIL --user_pass=$USER_PASSWORD --role='author' --allow-root

# install redi-cache plugin
wp plugin install redis-cache --allow-root

# activate the plugin of redis-cache
wp plugin activate redis-cache --allow-root 

# enable the plugin of redis-cache 
wp redis enable --allow-root

exec "$@"
```

In the dockerfile, I installed all dependencies for WordPress and run PHP-fmp7. 3 in the foreground mode.

```docker
# base image
FROM    debian:buster

# install php and php-curl and php-mysql and php7.3-fpm and curl and sendmail
RUN apt update && apt install -y php php-curl php-mysql\
    && apt install -y php7.3-fpm \
    && apt install -y curl \
    && apt install -y sendmail

# create the /run/php/ used for storing session data, temporary files, or other types of data that is generated and used by PHP while it's running
RUN mkdir -p /run/php/

# copy the script.sh into the container
COPY ./tools/script.sh /tmp/script.sh

# change the permission of script.sh
RUN chmod +x /tmp/script.sh

# run the script.sh
ENTRYPOINT [ "/tmp/script.sh" ]

# PHP-FPM to run in the foreground, that is, to not fork into the background as a daemon. This makes it so that the process will run and output any logs to the console where it was started.
CMD [ "php-fpm7.3", "-F" ]
```

then you can go to [https://yourlogin.1337.ma](https://mmoumni.1337.ma) you can see the WordPress page.

![Screen Shot 2023-01-14 at 9.31.25 PM.png](WORDPRESS%200d6cceabe3d04d04958a909587888df7/Screen_Shot_2023-01-14_at_9.31.25_PM.png)