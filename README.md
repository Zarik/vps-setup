# Настройка VPS сервера на Digital Ocean

Ubuntu + Nginx (front-end) + Apache (back-end) + PHP

## Базовые настройки

Создать дроплет, зайти по ssh под root, сменить пароль root.

Создать нового пользователя:

```
$ adduser username
```

Дать пользователю привелегии:
```
$ visudo
```
```
username ALL=(ALL:ALL) ALL
```

Залогиниться под новым пользователем.

Сменить ssh-порт по умолчанию (вместо 22) и запретить логин под root
```
$ sudo nano /etc/ssh/sshd_config
```
```
Port 22
PermitRootLogin no
```
Перезагружаем ssh
```
$ sudo service ssh restart
```

Обновляем систему
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

## Установка Apache, Nginx, PHP, MySQL, phpMyAdmin, git, postfix, memcached

Apache
```
$ sudo apt-get install apache2 apache2-mpm-prefork apache2-utils apache2-suexec libapache2-mod-rpaf
```
Nginx
```
$ sudo apt-get install nginx
```
PHP
```
$ sudo apt-get install php5 php5-mysql libapache2-mod-php5 php-pear php5-curl php5-common php5-idn php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl
```
MySQL

При установке выбрать apache2 (нажать пробел!), указать пароль root пользователя. Ответить yes на предложение настройки.

MySQL application password for phpmyadmin можно оставить пустым, он сгенерируется автоматически.
```
$ sudo apt-get install mysql-server mysql-client libmysqlclient-dev phpmyadmin
```

git
```
$ sudo apt-get install git
```

Почтовый сервер postfix
```
$ sudo apt-get install -y postfix
```

memcached
```
$ sudo apt-get install memcached
```

## Настройка Apache

Включить модули
```
$ sudo a2enmod ssl
$ sudo a2enmod rewrite
$ sudo a2enmod suexec
$ sudo a2enmod include
```

Сменить порт
```
$ sudo nano /etc/apache2/ports.conf
```
```
Listen 81
```
Для каждого сайта создать конфиг в /etc/apache2/sites-available/
```
$ sudo nano /etc/apache2/sites-available/example.com.conf
```
```
<VirtualHost *:81>
ServerName example.com
ServerAlias www.example.com
ServerAdmin webmaster@example.com
DocumentRoot /var/www/example.com/public_html
        <Directory />
                Options FollowSymLinks
                AllowOverride None
        </Directory>
        <Directory /var/www/example.com/public_html>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>
        ErrorLog /var/www/example.com/logs/apache-errors.log
        LogLevel warn
        CustomLog /var/www/example.com/logs/apache-access.log combined
</VirtualHost>
```
И папки для файлов сайта
```
$ sudo mkdir -p /var/www/example.com/public_html
$ sudo mkdir -p /var/www/example.com/logs/
```
Включить виртуальный хост
```
$ sudo a2ensite example.com
```

## Настройка Nginx

Базовый конфиг (worker_processes = количеству ядер процессора).
```
$ sudo nano /etc/nginx/nginx.conf
```
```
user www-data;
worker_processes 1;
pid /var/run/nginx.pid;
error_log /sites/nginx/error.log;
events {
        worker_connections 768;
        # multi_accept on;
} 
http { 
        ##
        # Basic Settings
        ##
 
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;
 
        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;
 
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
 
        ##
        # Logging Settings
        ##
 
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
 
        ##
        # Gzip Settings
        ##
 
        gzip on;
        gzip_disable "msie6";
 
        # gzip_vary on;
         gzip_proxied any;
         gzip_comp_level 7; #Level Compress
         gzip_buffers 16 8k;
         gzip_http_version 1.1;
         gzip_types text/plain text/css application/json application/x-javascri$
 
        ##
        # Virtual Host Configs
        ##    
 
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```
Конфиг проксирования
```
$ cd /etc/nginx
$ sudo touch proxy.conf
```
```
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size 100m;
client_body_buffer_size 128k;
proxy_connect_timeout 90;
proxy_send_timeout 90;
proxy_read_timeout 90;
proxy_buffer_size 32k;
proxy_buffers 32 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```
Для каждого сайта создать конфиг в /etc/nginx/sites-enabled/ (upstream backend указать только в 1 конфиге)
```
$ sudo nano /etc/nginx/sites-enabled/example.com
```
```
upstream backend {
        server 127.0.0.1:81;
}
server {
        listen          80;
        error_page      404     /404.html;
        error_page      403     /403.html;
        server_name     example.com www.example.com;

        access_log      /var/www/example.com/logs/nginx-access.log;
        error_log       /var/www/example.com/logs/nginx-error.log;

        location / {
                proxy_pass      http://backend;
                include         /etc/nginx/proxy.conf;
        }
        location ~* .(jpg|jpeg|gif|png|ico|css|bmp|swf|js|mov|avi|mp4|mpeg4) {
                root /var/www/example.com/public_html;
        }
        location ~ /.ht {
                deny all;
        }
}
```
Редирект с www
```
server {
 	server_name www.example.com;
 	rewrite ^(.*) http://example.com$1 permanent;
}
```
Доступ к phpmyadmin
```
$ sudo nano /etc/nginx/sites-enabled/phpmyadmin
```
```
location /phpmyadmin/ {
  proxy_pass http://127.0.0.1:81/phpmyadmin/;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $remote_addr;
  proxy_connect_timeout 120;
  proxy_send_timeout 120;
  proxy_read_timeout 180;
}
```
