MySQL User-defined function (UDF) for HTTP GET/POST
==========

MySQL User-defined function (UDF) for HTTP REST

**Note:** It is a fork repository. See original repositories below :  
 * https://github.com/y-ken/mysql-udf-http
 * https://code.google.com/archive/p/mysql-udf-http/

## Overview

| HTTP Method | CRUD Action |      Description       |
|-------------|-------------|------------------------|
| POST        |  CREATE     |  Create a new resource |
| GET         |  READ       |  Read a resource       |
| PUT         |  UPDATE     |  Update a resource     |
| DELETE      |  DELETE     |  Delete a resource     |

Tested on MySQL version 8.0.x on Linux Ubuntu 22.04

## User Guide

Two options, install on the systema manualy or run it with docker-compose

### 1. Install on system manualy

#### 1.1. Prepare

make sure these dependencies are installed.

* mysql server
* mysql_config command
* development tools

For Linux Ubuntu, like this.

```
apt-get install make 
apt-get install gcc
apt-get install libmysqlclient-dev
apt-get install pkg-config
apt-get install wget
apt-get install git
apt-get install libcurl4-openssl-dev
```

#### 2.1. Install on Linux Ubuntu

First, clone this project localy

```bash
git clone https://github.com/egosselin-dev/mysql-udf-http.git
```

Then, compile the library

```bash
cd mysql-udf-http 
chmod +x configure

./configure --with-mysql=/usr/bin/mysql_config
make && make install && \
cd ..
```

Copy the generated library on the mysql plugin folder

```bash
cp mysql-udf-http/src/.libs/mysql-udf-http.so /usr/lib/mysql/plugin/
```

Restart Mysql

```bash
service mysql restart
```

#### 3.1. Enable the UDF function in the MySQL console

```sql
create function http_get returns string soname 'mysql-udf-http.so';
create function http_post returns string soname 'mysql-udf-http.so';
create function http_put returns string soname 'mysql-udf-http.so';
create function http_delete returns string soname 'mysql-udf-http.so';
```

### 2. Run as a docker container

#### 2.1. Setting up docker-compose.yml

Use this in your docker-compose.yml file

```yml
version: "3"

networks:
  synchro:

services:
  database:
    build: ./docker
    image: mysql-http-udf
    tty: true
    container_name: database
    volumes:
      - rest_database:/var/lib/mysql
    environment:
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_ROOT_PASSWORD: ${DATABASE_PASSWORD}
    ports:
      - 3306:3306
    networks:
      - synchro

volumes:
  rest_database:
```

#### 2.2. Setting up Dockerfile

Create the following dockerfile in a 'docker' folder at the root of your project

```Dockerfile
# base image
FROM ubuntu:22.04

# installing mysql
RUN apt-get update && \
    apt-get install -y mysql-server

# installing build tools
RUN apt-get install -y make gcc libmysqlclient-dev pkg-config wget git libcurl4-openssl-dev

RUN git clone https://github.com/egosselin-dev/mysql-udf-http.git

# setting file permissions
RUN cd mysql-udf-http && chmod +x configure 

RUN cd mysql-udf-http && ./configure --with-mysql=/usr/bin/mysql_config && \
    make && make install && \
    cd ../

# copying generated library to mysql plugin directory
RUN cp /mysql-udf-http/src/.libs/mysql-udf-http.so /usr/lib/mysql/plugin/

EXPOSE 3306

# adding entrypoint to register custom mysql functions
ADD ./entrypoint.sh /entrypoint.sh
RUN chmod 755 /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# Change mysql to listen on 0.0.0.0 (otherwise you can't connect from outside)
RUN sed -i 's/127\.0\.0\.1/0.0.0.0/g' /etc/mysql/mysql.conf.d/mysqld.cnf

CMD ["mysqld"]
```

Create the following entrypoint.sh file in the same folder

```bash
#!/bin/bash

if [ -n "$MYSQL_ROOT_PASSWORD" ] ; then

	TEMP_FILE='/tmp/mysql-first-time.sql'
	cat > "$TEMP_FILE" <<-EOSQL
		-- setting up root password
        DELETE FROM mysql.user WHERE user = 'root' AND host = '%';
		CREATE USER 'root'@'%' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
		GRANT ALL ON *.* TO 'root'@'%' WITH GRANT OPTION ;
		FLUSH PRIVILEGES ;

		-- default database
		create database if not exists ${MYSQL_DATABASE};

        -- registering custom rest functions
        create function http_get returns string soname 'mysql-udf-http.so';
        create function http_post returns string soname 'mysql-udf-http.so';
        create function http_put returns string soname 'mysql-udf-http.so';
        create function http_delete returns string soname 'mysql-udf-http.so';
	EOSQL

	# set this as an init-file to execute on startup
	set -- "$@" --init-file="$TEMP_FILE"
fi

# execute the command supplied
exec "$@"
```

#### 2.3. Build the container

```bash
docker-compose up -d
```


### 3. Usage

#### Description:

```sql
SELECT http_get('<url>');
SELECT http_post('<url>', '<data>');
SELECT http_put('<url>', '<data>');
SELECT http_delete('<url>');
```

### Examples

Syntax for a GET request
```sql
select convert(http_get('https://reqres.in/api/users'), char);
```

Syntax for a POST request
```sql
select convert(http_post('https://reqres.in/api/users', '{\"name\":\"Roberto Testowski\, \"movies\": [\"The Big Lebowski\", \"The Matrix\"]}'), char);
```
