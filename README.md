# svcgw - Service Gateway
A readme on me making a Varnish service gateway

# The "Service"
I fired up an apache with php setup so that I can see what we're doing

## Apache
```
Server version: Apache/2.4.53 (Rocky Linux)
Server built:   Apr 28 2023 00:00:00
```

A basic config for the service in /etc/httpd/conf.d/the-service.conf
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

## PHP
```
PHP 8.0.27 (cli) (built: Jan  3 2023 16:17:26) ( NTS gcc x86_64 )
Copyright (c) The PHP Group
Zend Engine v4.0.27, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.27, Copyright (c), by Zend Technologies
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

## Testing
```
curl -XPUT -H'Host: service' http://127.0.0.1:4080/1/derp/test
{"method":"PUT","data":"1\/derp\/test"}
```

# Varnish Configuration
