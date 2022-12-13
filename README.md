# Caddy + Slim + Apache + MariaDB as Docker Stack

Three instances of PHP 8.1 as an Apache module load balanced by Caddy. Single MariaDB instance also in the swarm. This requires you to install Composer dependencies locally in the host machine.

If first time testing you need to checkout the repo, change branch and build the `slim-docker-apache` image. If you have already tested the `caddy-apache-php` branch the image has already been built for you.

```
$ git clone https://github.com/tuupola/slim-docker.git
$ cd slim-docker
$ git checkout caddy-apache-php-stack
$ composer install
$ docker build -t slim-docker-apache docker/apache/
$ docker build -t slim-docker-caddy docker/caddy/
```

If you have already tried the `caddy-apache-php` branch you can jump directly here.

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
$ curl --ipv4 --include apache.localhost
HTTP/1.1 308 Permanent Redirect
Connection: close
Location: https://apache.localhost/
Server: Caddy
Date: Tue, 13 Dec 2022 13:17:50 GMT
Content-Length: 0

$ curl --ipv4 --include --insecure https://apache.localhost
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
date: Tue, 13 Dec 2022 13:18:30 GMT
server: Caddy
server: Apache/2.4.54 (Debian)
x-powered-by: PHP/8.1.13
content-length: 12

Hello world!

$ curl --ipv4 --include --insecure https://apache.localhost/mars
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
date: Tue, 13 Dec 2022 13:18:52 GMT
server: Caddy
server: Apache/2.4.54 (Debian)
x-powered-by: PHP/8.1.13
content-length: 11

Hello mars!
```

Verify you can [query the database](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L26-L41) successfully.

```
$ curl --ipv4 --include --insecure https://apache.localhost/cars
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
date: Tue, 13 Dec 2022 13:19:30 GMT
server: Caddy
server: Apache/2.4.54 (Debian)
x-powered-by: PHP/8.1.13
content-length: 15

Tesla Audi BMW
```

Verify that [static files](https://github.com/tuupola/slim-docker/blob/apache-php/public/static.html) are being served.

```
$ curl --ipv4 --include --insecure https://apache.localhost/static.html
HTTP/2 200
accept-ranges: bytes
alt-svc: h3=":443"; ma=2592000
content-type: text/html
date: Tue, 13 Dec 2022 13:20:12 GMT
etag: "7-5ef4e0022ff77"
last-modified: Thu, 08 Dec 2022 09:52:52 GMT
server: Caddy
server: Apache/2.4.54 (Debian)
content-length: 7

static
```

You can also [dump the `$_SERVER`](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L17-L24) superglobal for debugging purposes.

```
$ curl --ipv4 --include --insecure "https://apache.localhost/server?foo=bar"
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
date: Tue, 13 Dec 2022 13:21:15 GMT
server: Caddy
server: Apache/2.4.54 (Debian)
vary: Accept-Encoding
x-powered-by: PHP/8.1.13

array (
  'REDIRECT_STATUS' => '200',
  'HTTP_HOST' => 'apache.localhost',
  'HTTP_USER_AGENT' => 'curl/7.82.0',
  'HTTP_ACCEPT' => '*/*',
  'HTTP_X_FORWARDED_FOR' => '10.0.0.2',
  'HTTP_X_FORWARDED_HOST' => 'apache.localhost',
  'HTTP_X_FORWARDED_PROTO' => 'https',
  'HTTP_ACCEPT_ENCODING' => 'gzip',
  'PATH' => '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin',
  'SERVER_SIGNATURE' => '<address>Apache/2.4.54 (Debian) Server at apache.localhost Port 80</address>
',
  'SERVER_SOFTWARE' => 'Apache/2.4.54 (Debian)',
  'SERVER_NAME' => 'apache.localhost',
  'SERVER_ADDR' => '10.0.12.13',
  'SERVER_PORT' => '80',
  'REMOTE_ADDR' => '10.0.12.6',
  'DOCUMENT_ROOT' => '/srv/www/public',
  'REQUEST_SCHEME' => 'http',
  'CONTEXT_PREFIX' => '',
  'CONTEXT_DOCUMENT_ROOT' => '/srv/www/public',
  'SERVER_ADMIN' => 'webmaster@localhost',
  'SCRIPT_FILENAME' => '/srv/www/public/index.php',
  'REMOTE_PORT' => '54994',
  'REDIRECT_URL' => '/server',
  'REDIRECT_QUERY_STRING' => 'foo=bar',
  'GATEWAY_INTERFACE' => 'CGI/1.1',
  'SERVER_PROTOCOL' => 'HTTP/1.1',
  'REQUEST_METHOD' => 'GET',
  'QUERY_STRING' => 'foo=bar',
  'REQUEST_URI' => '/server?foo=bar',
  'SCRIPT_NAME' => '/index.php',
  'PHP_SELF' => '/index.php',
  'REQUEST_TIME_FLOAT' => 1670937675.623871,
  'REQUEST_TIME' => 1670937675,
  'argv' =>
  array (
    0 => 'foo=bar',
  ),
  'argc' => 1,
)
```