# MariaDB

**What is MariaDB?**

> MariaDB is a relational database management system that is a fork of the MySQL RDBMS. It was created by the original developers of MySQL in 2009, after Oracle Corporation acquired MySQL AB. Like MySQL, MariaDB uses the Structured Query Language (SQL) to manage and manipulate data stored in its databases. MariaDB is designed to be fully compatible with MySQL, and many of its features are similar or identical.
> 

For the Project, I should set up a root user and another user, that has all privileges to the database created.

I created a script that automates the process of creating the database and the user, all of the environment variables that are present in the script are set in the `.env` file.

```bash
#!/bin/bash

# starting the mysql service
service mysql start

# change the bind to 0.0.0.0 only accept client connections made to 0.0.0.0 (accept connection to any address)
sed -i 's/bind-address            = 127.0.0.1/bind-address = 0.0.0.0/g' /etc/mysql/mariadb.conf.d/50-server.cnf

# create the database if not exist
mysql -u root -p$MYSQL_ROOTPASSWORD -e "CREATE DATABASE IF NOT EXISTS $MYSQL_DATABASE;"

# create the user if not exist
mysql -u root -p$MYSQL_ROOTPASSWORD -e "CREATE USER IF NOT EXISTS '$MYSQL_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD';"

# grant all priviliges on the created database to the user
mysql -u root -p$MYSQL_ROOTPASSWORD -e "GRANT ALL PRIVILEGES ON $MYSQL_DATABASE.* TO '$MYSQL_USER'@'%';"

# this command tell the MySQL or MariaDB server to reload the grant tables and update its internal data structures with the current contents of the grant tables.
mysql -u root -p$MYSQL_ROOTPASSWORD -e "FLUSH PRIVILEGES;"

# set the password to the root
mysql -u root -p$MYSQL_ROOTPASSWORD -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '$MYSQL_ROOTPASSWORD';"

# killing the porcess of mysqld to not restarting while waiting the wordpress to get setup
kill `cat /var/run/mysqld/mysqld.pid`

exec "$@"
```

For the Dockerfile of MariaDB :

```docker
# base image
FROM debian:buster

# install mariadb server
RUN apt update && apt -y install mariadb-server

# copy the script.sh into container
COPY ./tools/script.sh /tmp/script.sh

# change the permission to the script.sh
RUN chmod +x /tmp/script.sh

# run the script.sh
ENTRYPOINT [ "/tmp/script.sh" ]

# running mysqld daemon
CMD [ "mysqld" ]
```

in the docker-compose.yml file I expose the `3306` port (default port for mariadb), and linked the mariadb container the `data_db` volume connected in the `nat` network.

![mariadb section in docker-compose.yml file](MariaDB%20e72546a93ae84b89807ccb1fe420c6ab/Screen_Shot_2023-01-14_at_9.24.16_PM.png)

mariadb section in docker-compose.yml file