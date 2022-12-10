# Slim + Apache + MariaDB as Docker Stack

Three instances of PHP 8.1 as an Apache module load balanced by the swarm. Single MariaDB instance also in the swarm. This requires you to install Composer dependencies locally in the host machine.

If first time testing you need to checkout the repo, change branch and build the `slim-docker-apache` image. If you have already tested the `apache-php` branch the image has already been built for you.

```
$ git clone https://github.com/tuupola/slim-docker.git
$ cd slim-docker
$ git checkout apache-php-stack
$ composer install
$ docker build -t slim-docker-apache docker/apache/
```

If you have already tried the `apache-php` branch you can jump directly here.

```
$ docker swarm init
$ docker stack deploy -c stack.yaml slim
$ docker stack ls
$ docker service ls
```

Youcan `tail -f` Apache logs to see the load balancing in action.

```
$ docker service logs slim_apache -f
```

Verify that the [basic route](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L43-L51) is working.

```
$ curl --include localhost
HTTP/1.1 200 OK
Date: Sat, 10 Dec 2022 13:36:45 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13
Content-Length: 12
Content-Type: text/html; charset=UTF-8

Hello world!

$ curl --include localhost/mars
HTTP/1.1 200 OK
Date: Sat, 10 Dec 2022 13:37:10 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13
Content-Length: 11
Content-Type: text/html; charset=UTF-8

Hello mars!
```

Verify you can [query the database](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L26-L41) successfully.

```
$ curl --include localhost/cars
HTTP/1.1 200 OK
Date: Sat, 10 Dec 2022 13:37:32 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13
Content-Length: 15
Content-Type: text/html; charset=UTF-8

Tesla Audi BMW
```

Verify that [static files](https://github.com/tuupola/slim-docker/blob/apache-php/public/static.html) are being served.

```
$ curl --include localhost/static.html
HTTP/1.1 200 OK
Date: Sat, 10 Dec 2022 13:37:50 GMT
Server: Apache/2.4.54 (Debian)
Last-Modified: Thu, 08 Dec 2022 09:52:52 GMT
ETag: "7-5ef4e0022ff77"
Accept-Ranges: bytes
Content-Length: 7
Content-Type: text/htmls

static
```

You can also [dump the `$_SERVER`](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L17-L24) superglobal for debugging purposes.

```
$ curl --include "localhost/server?foo=bar"
HTTP/1.1 200 OK
Date: Sat, 10 Dec 2022 13:38:16 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13
Vary: Accept-Encoding
Content-Length: 1185
Content-Type: text/html; charset=UTF-8

array (
  'REDIRECT_STATUS' => '200',
  'HTTP_HOST' => 'localhost',
  'HTTP_USER_AGENT' => 'curl/7.82.0',
  'HTTP_ACCEPT' => '*/*',
  'PATH' => '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin',
  'SERVER_SIGNATURE' => '<address>Apache/2.4.54 (Debian) Server at localhost Port 80</address>
',
  'SERVER_SOFTWARE' => 'Apache/2.4.54 (Debian)',
  'SERVER_NAME' => 'localhost',
  'SERVER_ADDR' => '10.0.0.50',
  'SERVER_PORT' => '80',
  'REMOTE_ADDR' => '10.0.0.2',
  'DOCUMENT_ROOT' => '/srv/www/public',
  'REQUEST_SCHEME' => 'http',
  'CONTEXT_PREFIX' => '',
  'CONTEXT_DOCUMENT_ROOT' => '/srv/www/public',
  'SERVER_ADMIN' => 'webmaster@localhost',
  'SCRIPT_FILENAME' => '/srv/www/public/index.php',
  'REMOTE_PORT' => '43978',
  'REDIRECT_URL' => '/server',
  'REDIRECT_QUERY_STRING' => 'foo=bar',
  'GATEWAY_INTERFACE' => 'CGI/1.1',
  'SERVER_PROTOCOL' => 'HTTP/1.1',
  'REQUEST_METHOD' => 'GET',
  'QUERY_STRING' => 'foo=bar',
  'REQUEST_URI' => '/server?foo=bar',
  'SCRIPT_NAME' => '/index.php',
  'PHP_SELF' => '/index.php',
  'REQUEST_TIME_FLOAT' => 1670679496.15585,
  'REQUEST_TIME' => 1670679496,
  'argv' =>
  array (
    0 => 'foo=bar',
  ),
  'argc' => 1,
)
```