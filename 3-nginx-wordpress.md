
# Установк Wordpress


### Конфиги для повторяющихся настроек

```
$ sudo mkdir /etc/nginx/common
$ sudo nano /etc/nginx/common/upstream
```

```
upstream php-fpm
{
	server unix:/var/run/php5-fpm.sock;
}
```

### Конфиг

```
$ sudo touch /etc/nginx/sites-available/example.com
$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
$ sudo nano /etc/nginx/sites-available/example.com
```
```
include common/upstream;
server
{
  listen	80;
  root			/var/www/example.com/public_html;
  index			index.php index.html index.htm;
  server_name		example.com www.example.com;
  
  location "/"
		{
		index index.php index.html index.htm;
		try_files	$uri $uri/	=404;
		}
		
	location ~* "^(wp-config.php)((/.*)?)$"
    {
	  deny all;
	  return 404;
    }
```



