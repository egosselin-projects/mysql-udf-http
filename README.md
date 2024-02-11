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


### 1. Prepare

make sure these depandencies are installed.

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

### 2. Install on Linux Ubuntu

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

### 3. Enable the UDF function in the MySQL console

```sql
create function http_get returns string soname 'mysql-udf-http.so';
create function http_post returns string soname 'mysql-udf-http.so';
create function http_put returns string soname 'mysql-udf-http.so';
create function http_delete returns string soname 'mysql-udf-http.so';
```

### 4. Usage

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
