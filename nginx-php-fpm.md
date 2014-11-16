# Настройка Nginx + PHP-FPM

### Установка Nginx
```
$ sudo apt-get install nginx
```

### Установка PHP-FPM
```
$ sudo apt-get install php5-cli php5-common php5-mysql php5-gd php5-fpm php5-cgi php5-fpm php-pear curl php5-curl php5-mcrypt
```
Устранение уязвимости php
```
$ sudo nano /etc/php5/fpm/php.ini
```
```
cgi.fix_pathinfo=0
```
```
$ sudo service php5-fpm restart
```
### Установка MySQL 
Указать пароль MySQL root пользователя
```
$ sudo apt-get mysql-server mysql-client mysql-common
```

### Установка phpMyAdmin
Не выбирать сервер
```
$ sudo apt-get install phpmyadmin
```

### Установка memcached
```
$ sudo apt-get install memcached php5-memcached
```

### Настройка nginx
```
$ sudo nano /etc/nginx/sites-available/default
```

```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.php index.html index.htm;

    server_name server_domain_name_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Делаем доступ к phpmyadmin
```
$sudo ln -s /usr/share/phpmyadmin/ /usr/share/nginx/html
```

```
$ sudo service nginx restart
```
