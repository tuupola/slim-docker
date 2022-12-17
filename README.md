# Traefik + Slim + Apache + MariaDB as Docker Stack

Three instances of PHP 8.1 as an Apache module in swarm load balanced by Traefik. Single MariaDB instance also in the swarm. This requires you to install Composer dependencies locally in the host machine.

If first time testing you need to checkout the repo, change branch and build the `slim-docker-apache` image. If you have already tested the `apache-php` branch the image has already been built for you.

```
$ git clone https://github.com/tuupola/slim-docker.git
$ cd slim-docker
$ git checkout traefik-apache-php-stack
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

Verify you can access the [dashboard](http://traefik.localhost/dashboard/).

```
$ curl --ipv4 --include traefik.localhost
HTTP/1.1 302 Found
Content-Type: text/html; charset=utf-8
Location: /dashboard/
Date: Sun, 11 Dec 2022 16:49:35 GMT
Content-Length: 34

<a href="/dashboard/">Found</a>.
```

You can `tail -f` Apache logs to see the load balancing in action.

```
$ docker service logs slim_apache -f
```

Verify that the [basic route](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L43-L51) is working.

```
$ curl --ipv4 --include apache.localhost
HTTP/1.1 200 OK
Content-Length: 12
Content-Type: text/html; charset=UTF-8
Date: Sun, 11 Dec 2022 16:53:38 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13

Hello world!

$ curl --ipv4 --include apache.localhost/mars
HTTP/1.1 200 OK
Content-Length: 11
Content-Type: text/html; charset=UTF-8
Date: Sun, 11 Dec 2022 16:54:14 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13

Hello mars!
```

Verify you can [query the database](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L26-L41) successfully.

```
$ curl --ipv4 --include apache.localhost/cars
HTTP/1.1 200 OK
Content-Length: 15
Content-Type: text/html; charset=UTF-8
Date: Sun, 11 Dec 2022 16:55:00 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/8.1.13

Tesla Audi BMW
```

Verify that [static files](https://github.com/tuupola/slim-docker/blob/apache-php/public/static.html) are being served.

```
$ curl --ipv4 --include apache.localhost/static.html
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
$ curl --ipv4 --include "apache.localhost/server?foo=bar"
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Date: Sun, 11 Dec 2022 16:55:54 GMT
Server: Apache/2.4.54 (Debian)
Vary: Accept-Encoding
X-Powered-By: PHP/8.1.13
Transfer-Encoding: chunked

array (
  'REDIRECT_STATUS' => '200',
  'HTTP_HOST' => 'apache.localhost',
  'HTTP_USER_AGENT' => 'curl/7.82.0',
  'HTTP_ACCEPT' => '*/*',
  'HTTP_X_FORWARDED_FOR' => '10.0.0.2',
  'HTTP_X_FORWARDED_HOST' => 'apache.localhost',
  'HTTP_X_FORWARDED_PORT' => '80',
  'HTTP_X_FORWARDED_PROTO' => 'http',
  'HTTP_X_FORWARDED_SERVER' => 'a70a47c6effa',
  'HTTP_X_REAL_IP' => '10.0.0.2',
  'HTTP_ACCEPT_ENCODING' => 'gzip',
  'PATH' => '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin',
  'SERVER_SIGNATURE' => '<address>Apache/2.4.54 (Debian) Server at apache.localhost Port 80</address>
',
  'SERVER_SOFTWARE' => 'Apache/2.4.54 (Debian)',
  'SERVER_NAME' => 'apache.localhost',
  'SERVER_ADDR' => '10.0.3.8',
  'SERVER_PORT' => '80',
  'REMOTE_ADDR' => '10.0.3.14',
  'DOCUMENT_ROOT' => '/srv/www/public',
  'REQUEST_SCHEME' => 'http',
  'CONTEXT_PREFIX' => '',
  'CONTEXT_DOCUMENT_ROOT' => '/srv/www/public',
  'SERVER_ADMIN' => 'webmaster@localhost',
  'SCRIPT_FILENAME' => '/srv/www/public/index.php',
  'REMOTE_PORT' => '40578',
  'REDIRECT_URL' => '/server',
  'REDIRECT_QUERY_STRING' => 'foo=bar',
  'GATEWAY_INTERFACE' => 'CGI/1.1',
  'SERVER_PROTOCOL' => 'HTTP/1.1',
  'REQUEST_METHOD' => 'GET',
  'QUERY_STRING' => 'foo=bar',
  'REQUEST_URI' => '/server?foo=bar',
  'SCRIPT_NAME' => '/index.php',
  'PHP_SELF' => '/index.php',
  'REQUEST_TIME_FLOAT' => 1670777754.033114,
  'REQUEST_TIME' => 1670777754,
  'argv' =>
  array (
    0 => 'foo=bar',
  ),
  'argc' => 1,
)
```