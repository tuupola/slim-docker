# Caddy + PHP-FPM + MariaDB as a Stack

Modern simple setup. Three instances of PHP 8.1 as PHP-FPM load balanced by Caddy and MariaDB as the database. All in separate containers. Caddy handles TLS automatically. Current directory mounted into webserver so code changes can be seen immediately. This requires you to install Composer dependencies locally in the host machine.

```
$ git clone https://github.com/tuupola/slim-docker.git
$ cd slim-docker
$ git checkout caddy-phpfpm-stack
$ composer install
```

If this is the first time testing you need to build the local Docker images.

```
$ docker build -t slim-docker-caddy docker/caddy/
$ docker build -t slim-docker-fpm docker/fpm/
```

```
$ docker swarm init
$ docker stack deploy -c stack.yaml slim
$ docker stack ls
$ docker service ls
```

You can tail -f FPM logs to see the load balancing in action.

```
$ docker service logs slim_fpm -f
```

Verify that the [basic route](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L43-L51) is working.

```
$ curl --ipv4 --include example.localhost
HTTP/1.1 308 Permanent Redirect
Connection: close
Location: https://example.localhost/
Server: Caddy
Date: Wed, 28 Dec 2022 12:57:13 GMT
Content-Length: 0

$ curl --ipv4 --include --insecure https://example.localhost
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
server: Caddy
x-powered-by: PHP/8.1.13
content-length: 12
date: Wed, 28 Dec 2022 12:57:40 GMT

Hello world!%


$ curl --ipv4 --include --insecure https://example.localhost/mars
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
server: Caddy
x-powered-by: PHP/8.1.13
content-length: 11
date: Wed, 28 Dec 2022 12:58:06 GMT


Hello mars!
```

Verify you can [query the database](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L26-L41) successfully.


```
$ curl --ipv4 --include --insecure https://example.localhost/cars
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
server: Caddy
x-powered-by: PHP/8.1.13
content-length: 15
date: Wed, 28 Dec 2022 12:58:23 GMT

Tesla Audi BMW
```

Verify that [static files](https://github.com/tuupola/slim-docker/blob/apache-php/public/static.html) are being served.

```
$ curl --ipv4 --include --insecure https://example.localhost/static.html
HTTP/2 200
accept-ranges: bytes
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=utf-8
etag: "rnc56y7"
last-modified: Fri, 23 Dec 2022 08:35:22 GMT
server: Caddy
content-length: 7
date: Wed, 28 Dec 2022 12:58:46 GMT


static
```

You can also [dump the `$_SERVER`](https://github.com/tuupola/slim-docker/blob/apache-php/app.php#L17-L24) superglobal for debugging purposes.

```
$ curl --ipv4 --include --insecure "https://example.localhost/server?foo=bar"
HTTP/2 200
alt-svc: h3=":443"; ma=2592000
content-type: text/html; charset=UTF-8
server: Caddy
x-powered-by: PHP/8.1.13
content-length: 2225
date: Wed, 28 Dec 2022 13:00:18 GMT

array (
  'HOSTNAME' => '09c18e800d6f',
  'PHP_INI_DIR' => '/usr/local/etc/php',
  'SHLVL' => '1',
  'HOME' => '/home/www-data',
  'PHP_LDFLAGS' => '-Wl,-O1 -pie',
  'PHP_CFLAGS' => '-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64',
  'PHP_VERSION' => '8.1.13',
  'GPG_KEYS' => '528995BFEDFBA7191D46839EF9BA0ADA31CBD89E 39B641343D8C104B2B146DC3F9C39DC0B9698544 F1F692238FBC1666E5A5CCD4199F9DFEF6FFBAFD',
  'PHP_CPPFLAGS' => '-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64',
  'PHP_ASC_URL' => 'https://www.php.net/distributions/php-8.1.13.tar.xz.asc',
  'PHP_URL' => 'https://www.php.net/distributions/php-8.1.13.tar.xz',
  'PATH' => '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin',
  'PHPIZE_DEPS' => 'autoconf 		dpkg-dev dpkg 		file 		g++ 		gcc 		libc-dev 		make 		pkgconf 		re2c',
  'PWD' => '/srv/www',
  'PHP_SHA256' => 'b15ef0ccdd6760825604b3c4e3e73558dcf87c75ef1d68ef4289d8fd261ac856',
  'USER' => 'www-data',
  'PATH_INFO' => '',
  'HTTP_X_FORWARDED_FOR' => '10.0.0.2',
  'SSL_PROTOCOL' => 'TLSv1.3',
  'REMOTE_HOST' => '10.0.0.2',
  'QUERY_STRING' => 'foo=bar',
  'CONTENT_LENGTH' => '0',
  'GATEWAY_INTERFACE' => 'CGI/1.1',
  'AUTH_TYPE' => '',
  'HTTP_ACCEPT' => '*/*',
  'HTTP_HOST' => 'example.localhost',
  'REMOTE_USER' => '',
  'REMOTE_ADDR' => '10.0.0.2',
  'SERVER_SOFTWARE' => 'Caddy/v2.6.2',
  'REQUEST_METHOD' => 'GET',
  'SERVER_PORT' => '443',
  'REQUEST_URI' => '/server?foo=bar',
  'SERVER_PROTOCOL' => 'HTTP/2.0',
  'SCRIPT_FILENAME' => '/srv/www/public/index.php',
  'SERVER_NAME' => 'example.localhost',
  'REMOTE_PORT' => '34118',
  'REMOTE_IDENT' => '',
  'SCRIPT_NAME' => '/index.php',
  'CONTENT_TYPE' => '',
  'HTTP_X_FORWARDED_PROTO' => 'https',
  'HTTP_USER_AGENT' => 'curl/7.82.0',
  'SSL_CIPHER' => 'TLS_AES_128_GCM_SHA256',
  'HTTPS' => 'on',
  'DOCUMENT_URI' => '/index.php',
  'DOCUMENT_ROOT' => '/srv/www/public',
  'REQUEST_SCHEME' => 'https',
  'HTTP_X_FORWARDED_HOST' => 'example.localhost',
  'FCGI_ROLE' => 'RESPONDER',
  'PHP_SELF' => '/index.php',
  'REQUEST_TIME_FLOAT' => 1672232418.095838,
  'REQUEST_TIME' => 1672232418,
  'argv' =>
  array (
    0 => 'foo=bar',
  ),
  'argc' => 1,
)
```