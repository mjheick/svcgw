# Service Gateway
A readme on me making a Varnish service gateway

# The "Service"
I fired up an apache with php setup so that I can see what we're doing

## "Service" Configuration
A basic config for Apache in /etc/httpd/conf.d/the-service.conf
```
Listen 4080
<VirtualHost *:4080>
  ServerName service
  DocumentRoot /var/www/html
  RewriteEngine on
  RewriteCond %{REQUEST_URI} /.*
  RewriteRule ^/(.*) /service.php?data=$1
</VirtualHost>
```

/var/www/html/service.php contains the following:
```
<?php
$method = $_SERVER['REQUEST_METHOD'];
$data = isset($_GET['data']) ? $_GET['data'] : '';
echo json_encode([
'method' => $method,
'data' => $data
]);
```

## Testing "Service"
```
curl -XPUT -H'Host: service' http://127.0.0.1:4080/1/derp/test
{"method":"PUT","data":"1\/derp\/test"}
```

## Varnish
Installed and created a custom /etc/varnish/service.vcl
```
vcl 4.1;
import directors;

# Define a backend
backend service_01_4080 {
    .host = "192.168.1.108";
    .port = "4080";
    .max_connections = 200;
    .connect_timeout = 5s;
    .probe = {
        .url = "/";
        .timeout = 1s;
        .interval = 5s;
        .window = 5;
        .threshold = 3;
    }
}
backend service_02_4080 {
    .host = "192.168.2.108";
    .port = "4080";
    .max_connections = 200;
    .connect_timeout = 5s;
    .probe = {
        .url = "/";
        .timeout = 1s;
        .interval = 5s;
        .window = 5;
        .threshold = 3;
    }
}

sub vcl_init {
    new dir_service_4080 = directors.random();
    dir_service_4080.add_backend(service_01_4080, 1);
    dir_service_4080.add_backend(service_02_4080, 1);
}

sub vcl_recv {
    # unset all cookies
    unset req.http.cookie;
    unset req.http.authorization;

    # If the inbound hostname
    if (req.http.Host ~ "service.gw") {
        set req.http.Host = "service";
        set req.backend_hint = dir_service_4080.backend();
    }
}
```

## Testing Varnish
```
curl -XNERD -H'Host: service.gw' http://localhost/1/flubs
{"method":"NERD","data":"1\/flubs"}
```

## Software Versions
*Apache*
```
Server version: Apache/2.4.53 (Rocky Linux)
Server built:   Apr 28 2023 00:00:00
```

*PHP*
```
PHP 8.0.27 (cli) (built: Jan  3 2023 16:17:26) ( NTS gcc x86_64 )
Copyright (c) The PHP Group
Zend Engine v4.0.27, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.27, Copyright (c), by Zend Technologies
```

*Varnish*
```
varnishd (varnish-6.6.2 revision 17c51b08e037fc8533fb3687a042a867235fc72f)
Copyright (c) 2006 Verdens Gang AS
Copyright (c) 2006-2020 Varnish Software
```
