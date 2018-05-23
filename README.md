# Docker Apache-PHP-MySQL stack

This stack uses three containers :

 1. Apache HTTP server
 2. PHP-FPM server
 3. MySQL server

Each `ENTRYPOINT` or `COMMAND` **must be** run in foreground, otherwise the command will launch the daemon and exit immediately, since the Docker engine monitor the command with `PID=1`, the container will also exit.

A network bridge **must be** created to provide containers with DNS server access and inbound ports opened between themselves.

Volumes are also needed, a volume for the MySQL data storage and bind mounts for the `DocumentRoot` folder.

Build the `apache` and `phpfpm` images :

    cd apache && docker build -t apache .
    cd phpfpm && docker build -t phpfpm .

Create the user-defined bridge network and the volume for the MySQL data storage :

    docker network create my-project && docker volume create my-project-mysql

Create and launch the MySQL container :

    docker run -d -v my-project-mysql:/var/lib/mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes --name my-project-mysql --network=my-project --restart=always mysql

Create the `public` folder for the Apache `DocumentRoot` :

    mkdir -p ~/Sites/my-project/public

Create and launch the PHP-FPM container :

    docker run -d -v ~/Sites/my-project:/var/www/my-project --name my-project-phpfpm --network=my-project --restart=always phpfpm

Create and launch the Apache container :

    docker run -d -p 8080:80 -p 8443:443 -v ~/Sites/my-project:/var/www/my-project -e DOCUMENT_ROOT=/var/www/my-project/public -e PHPFPM=my-project-phpfpm --name my-project-apache --network=my-project --restart=always apache

>**Important :** the `DocumentRoot` set in the PHP-FPM container must be the same in the Apache container.

You can check if everything is running fine with :

    docker ps -a
