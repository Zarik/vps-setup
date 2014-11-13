Установка Nginx
```
$ sudo apt-get install nginx
```

Установка PHP-FPM
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
$ sudo /etc/init.d/php5-fpm restart
```
Установка MySQL (указать пароль MySQL root пользователя)
```
$ sudo install mysql-server mysql-client mysql-common
```

Установка phpMyAdmin (не выбирать сервер)
```
$ sudo install phpmyadmin
```
