# Настройка VPS сервера


*Данная инструкция является пошаговым алгоритмом настройки VPS под определенные ниже задачи и условия:*

*1. Используется операционная система Ubuntu 14.04*

*2. Используется стек Nginx as frontend + Apache as backend*

*3. Используется база данных MySQL*

*4. Опционально: VPS настраивается для хостинга wordpress-сайтов*

*5. Опционально: автоматический deploy сайта из git-репозитория на bitbucket.org*


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
$ sudo apt-get install memcached php5-memcached
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
Поменять порт на 81 в дефолтном конфиге
```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```
Для каждого сайта создать конфиг в `/etc/apache2/sites-available/`
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

Добавить в дефолтный конфиг ссылку на backend и сделать доступным phpmyadmin
```
$ sudo nano /etc/nginx/sites-enabled/default
```
```
upstream backend {
        server 127.0.0.1:81;
}
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

Для каждого сайта создать конфиг в `/etc/nginx/sites-enabled/`
```
$ sudo nano /etc/nginx/sites-enabled/example.com
```
```
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
## Автоматический deploy из git-репозитория на bitbucket.org

*По [инструкции](http://jonathannicol.com/blog/2013/11/19/automated-git-deployments-from-bitbucket/), с дополнениями.*

Подразумевается, что на bitbucket.org уже создан приватный git-репозиторий.

Определить под каким пользователем работает Apache (скорее всего это www-data)

Сгенерировать ssh ключ для этого пользователя. Домашняя папка пользователя www-data `/var/www/`.
```
$ sudo -u www-data ssh-keygen -t rsa
```
При генерации можно оставить имя файла по умолчанию (id_rsa) или указать своё (например, bitbucket_rsa)

Добавить полученный ключ в Deployment keys репозитория (Settings -> Deployment keys)

Создать директорию для хранения репозитория на сервере. Например `/var/git-repos`.
```
$ sudo mkdir /var/www/git-repos
```
Зайти в неё и клонировать репозиторий с ключем --mirror
```
$ cd /var/www/git-repos
$ sudo -u www-data git clone --mirror git@bitbucket.org:username/example.git
```
И скопировть файлы из репозитория в папку с сайтом
```
$ cd example.git
$ sudo -u www-data GIT_WORK_TREE=/var/www/example.com/public_html git checkout -f master
```
Команды выполняются от имени пользователя www-data, чтобы сразу выявить возможные проблемы с доступом.

Создать файлы скриптов деплоя
```
$ cd /var/www/example.com/public_html
$ mkdir deploy
$ cd deploy
$ touch deploy.log
$ sudo nano deploy.php
```
Сам скрипт
```
<?php
$repo_dir = '/var/www/git-repos/example.git';
$web_root_dir = '/var/www/example.com/public_html';

// Full path to git binary is required if git is not in your PHP user's path. Otherwise just use 'git'.
$git_bin_path = 'git';

$update = false;

// Parse data from Bitbucket hook payload
$payload = json_decode($_POST['payload']);

if (empty($payload->commits)){
  // When merging and pushing to bitbucket, the commits array will be empty.
  // In this case there is no way to know what branch was pushed to, so we will do an update.
  $update = true;
} else {
  foreach ($payload->commits as $commit) {
    $branch = $commit->branch;
    if ($branch === 'master' || isset($commit->branches) && in_array('master', $commit->branches)) {
      $update =	true;
      break;
    }
  }
}

if ($update) {
  // Do a git checkout to the web root
  exec('cd ' . $repo_dir . ' && ' . $git_bin_path  . ' fetch');
  exec('cd ' . $repo_dir . ' && GIT_WORK_TREE=' . $web_root_dir . ' ' . $git_bin_path  . ' checkout -f');

  // Log the deployment
  $commit_hash = shell_exec('cd ' . $repo_dir . ' && ' . $git_bin_path  . ' rev-parse --short HEAD');
  file_put_contents('deploy.log', date('m/d/Y h:i:s a') . " Deployed branch: " .  $branch . " Commit: " . $commit_hash . "\n", FILE_APPEND);
}
?>
```

У пользователя www-data должны быть права на директорию с git-репозиторием, и на директорию с файлами сайта.
```
$ sudo chown -R www-data /var/www
$ sudo chown -R www-data /var/www/git-repos
```
