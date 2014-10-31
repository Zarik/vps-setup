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
Обновляем систему
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

## Установка Apache, Nginx, PHP, MySQL, phpMyAdmin

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

## Настройка Apache

Включить модули
```
$ sudo a2enmod ssl
$ sudo a2enmod rewrite
$ sudo a2enmod suexec
$ sudo a2enmod include
```
