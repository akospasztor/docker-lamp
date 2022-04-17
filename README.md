# Simple Docker LAMP Stack

This project contains a minimal LAMP (Linux, Apache, MySQL, PHP/Perl/Python)
stack built with Docker Compose. This simple stack enables to easily and
convinently develop and run websites locally. There is no more need to go
though the pain of setting up and configuring XAMPP or similar tools.
Furthermore with Docker, each development environment can be customized and set
up depending on the needs and requirements, independently from each other and
without affecting the host environment.

The stack consists of three Docker images:
  - Apache server with PHP version 7.4
  - MySQL server version 5.7
  - PhpMyAdmin with the latest docker image version

## Configuration

The `.env` file contains some environmental variables to configure the LAMP
stack.

Project settings:
  - `COMPOSE_PROJECT_NAME`: This variable defines the name of the virtual host
    (see below) and prefixes the docker container names as well as the created
    network for the containers.
  - `DOCUMENT_ROOT`: The specified folder is mounted to the apache container as
    a bind mount. The contents of this folder is served with the apache server.

Port settings:
  - `HOST_APACHE_HTTP_PORT`: Defines the HTTP (unsecured) port of the apache
    server.
  - `HOST_APACHE_HTTPS_PORT`: Defines the HTTPS (secured) port of the apache
    server.
  - `HOST_MYSQL_PORT`: Defines the port of the MySQL server.
  - `HOST_PMA_PORT`: Defines the port of phpMyAdmin. The phpMyAdmin interface
    can be reached from `localhost:HOST_PMA_PORT` or `127.0.0.1:HOST_PMA_PORT`.

Apache settings:
  - `APACHE_DOCUMENT_ROOT`: The DocumentRoot folder of Apache; this is the top-
    level directory in the document tree from which files are served.

MySQL settings:
  - `MYSQL_ROOT_PASSWORD`: The MqSQL root password.
  - `MYSQL_DATABASE`: The name of the created default database upon the startup
    of the docker image.

PhpMyAdmin settings:
  - `PMA_MEMORY_LIMIT`: Override the default memory limit.
  - `PMA_UPLOAD_LIMIT`: Override the default upload size limit.

## Virtual Hosts
The served website can be reached by typing `localhost` or `127.0.0.1` (if the
`HOST_APACHE_HTTP_PORT` is _not_ set to `80`, then
`localhost:HOST_APACHE_HTTP_PORT` or `127.0.0.1:HOST_APACHE_HTTP_PORT`).

In oder to make life easier, the virtual host functionality is enabled in the
Apache docker container. Therefore, multiple websites with different domain
names can be run on a single server. This enables the possibility to reach the
served website via a the domain name defined by the `COMPOSE_PROJECT_NAME`
environmental variable. To expose this domain name to the host (who runs
the docker containers), the domain name needs to be added to the hosts file:

1. On MacOS and Linux, open the hosts file (`/etc/hosts`) with elevated
   privileges:
   ```
   sudo nano /etc/hosts
   ```

2. Add the following entry to a new line:
   ```
   ...
   127.0.0.1       lamp
   ...
   ```
   where `lamp` is the name of the virtual host.

3. Save the modifications and the served website now can be reached by simply
   typing `lamp` into the browser.

**Note:** On Windows, the hosts file is located at:
`c:\Windows\System32\Drivers\etc\hosts`.

## Serving Multiple Websites

With the help of Virutal Hosts, multiple websites can be served simultaneously.

Steps to serve multiple websites with the same webserver:

1. Define a new DocumentRoot and server name within the Apache docker container
   (e.g. with new environmental variables in the `docker-compose.yml` file):
   ```
   environment:
       ...
       SECOND_DOCUMENT_ROOT: /var/www/second
       SECOND_VIRTUALHOST_SERVER_NAME: second
       ...
   ```

2. Mount the folder of the additional site to the Apache docker container by
   adding a new bind mount:
   ```
   volumes:
      ...
      - path/on/host/to/second/site:${SECOND_DOCUMENT_ROOT}
      ...
   ```

3. Edit the `config/vhosts/default.conf` file and add a new virtual host entry
   for the additional site:
   ```
   ...
   <VirtualHost *:80>
       ServerName ${SECOND_VIRTUALHOST_SERVER_NAME}
       DocumentRoot ${SECOND_DOCUMENT_ROOT}
       <Directory ${SECOND_DOCUMENT_ROOT}>
           AllowOverride all
       </Directory>
   </VirtualHost>
   ...
   ```

4. Add the server name of the additional site to the `hosts` file:
   ```
   ...
   127.0.0.1       second
   ...
   ```

After restaring the LAMP stack, both sites are available with their respective
domain names simultaneously.

## Usage

Execute the following command (from the folder where the `docker-compose.yml`
file resides) to build the containers and run the LAMP stack:
```
docker-compose up -d
```

The served website can be accessed from the addresses of `localhost` and
`127.0.0.1`. If the virtual host name is exposed to the host (see above), then
the website can also be accessed via this domain name.

The phpMyAdmin interface can be accessed via `localhost:8080` by default.

To stop the containers and tear down the LAMP stack, execute:
```
docker-compose down
```
This removes all created containers and networks defined in the compose file.


## Testing

A basic GitHub Actions workflow is used to test whether the LAMP stack is
operational. The workflow file (`.github/workflows/ci.yml`) runs automatically
on each git push and checks whether the docker containers can be started and
the served test website, as well as the phpMyAdmin interface is reachable.
