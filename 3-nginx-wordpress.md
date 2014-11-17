
# Установк Wordpress


### Конфиги для повторяющихся настроек

```
$ sudo mkdir /etc/nginx/common
```

PHP-FPM
```
$ sudo nano /etc/nginx/common/upstream
```
```
upstream php-fpm {
	server unix:/var/run/php5-fpm.sock;
}
```

Wordpress Super Cache
```
$ sudo nano /etc/nginx/common/wordpress-super-cache
```
```
set $cache_uri $request_uri;

if ($request_method = POST) {
	set $cache_uri 'null cache';
}

if ($query_string != "") {
	set $cache_uri 'null cache';
}   

# Don't cache uris containing the following segments
if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
	set $cache_uri 'null cache';
}   

# Don't use the cache for logged in users or recent commenters
# if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_logged_in") {
#	set $cache_uri 'null cache';
#}

# Use cached or actual file if they exists, otherwise pass request to WordPress
location / {
        try_files /wp-content/cache/supercache/$http_host/$cache_uri/index.html $uri $uri/ /index.php?$args ;
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

server {
	listen  80;
	server_name  www.example.com;
	rewrite ^ http://example.com$request_uri? permanent; #301 redirect
}
server {
	listen  80;
	server_name  example.com; 
	root   /var/www/example.com;
	index  index.php;

	location / {
		try_files $uri $uri/ /index.php?q=$uri&$args;
	}
	location ~*^.+.(jpg|jpeg|gif|png|ico|css|bmp|swf|js|mov|avi|mp4|mpeg4)$ {
		access_log off;
		expires 3d;
	}
	
	location ~ \.php$ {
        	fastcgi_split_path_info ^(.+\.php)(/.+)$;
        	fastcgi_pass unix:/var/run/php5-fpm.sock;
        	fastcgi_index index.php;
        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        	include fastcgi_params;
        }

	location = /robots.txt {
		allow all;
		log_not_found off;
		access_log off;
	}
	
	location ~ /wp-config.php {
		deny  all;
	}
	
	location ~ /\.ht {
		deny  all;
	}
}
```



