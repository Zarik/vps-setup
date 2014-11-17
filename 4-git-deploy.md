# Автоматический деплой из git-репозитория

### Установка git
```
$ sudo apt-get install git
```

У пользователя www-data должны быть права на директорию `/var/www/`.
```
$ sudo chown -R www-data:www-data /var/www
```

Сгенерировать ssh ключ для этого пользователя. Домашняя папка пользователя www-data `/var/www/`.
```
$ sudo -u www-data ssh-keygen -t rsa
```
При генерации можно оставить имя файла по умолчанию (id_rsa) или указать своё (например, bitbucket_rsa)

Создать файл config в `/var/www/.ssh`
```
$ sudo -u www-data nano /var/www/.ssh/config
```
```
Host bitbucket.org
 IdentityFile ~/.ssh/id_rsa
```
Добавить полученный ключ в Deployment keys репозитория (Settings -> Deployment keys)

Права доступа на ключи
```
$ sudo chmod 0755 /var/www/.ssh
$ sudo chmod 0600 /var/www/.ssh/id_rsa
$ sudo chmod 0600 /var/www/.ssh/id_rsa.pub
$ sudo chmod 0644 /var/www/.ssh/known_hosts
```

Создать директорию для хранения репозитория на сервере. Например `/var/git-repos`.
```
$ sudo mkdir /var/www/git
```
Дать права на директорию пользователю www-data
```
$ sudo chown -R www-data:www-data /var/www/git
```

Зайти в неё и клонировать репозиторий с ключем --mirror
```
$ cd /var/www/git
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
$ sudo -u www-data mkdir deploy
$ cd deploy
$ sudo -u www-data touch deploy.log
$ sudo -u www-data nano deploy.php
```
Сам скрипт (предполагаются 2 ветки - master и production в разных папках).
```
<?php

// Examine the Bitbucket payload that’s being sent to deployment script
file_put_contents('deploy.log', serialize($_POST['payload']) . "\n", FILE_APPEND);

$repo_dir = '/var/git-repos/example.git';
$master_dir = '/var/www/dev.example.com/public_html';
$production_dir = '/var/www/example.com/public_html';

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
            $update = true;
            break;
        }
        elseif ($branch === 'production' || isset($commit->branches) && in_array('production', $commit->branches)) {
            $update = true;
            break;
        }
    }
}

if ($update) {
    // Do a git checkout to the web root
    exec('cd ' . $repo_dir . ' && ' . $git_bin_path  . ' fetch');
    exec('cd ' . $repo_dir . ' && GIT_WORK_TREE=' . $master_dir . ' ' . $git_bin_path  . ' checkout -f master');
    exec('cd ' . $repo_dir . ' && GIT_WORK_TREE=' . $production_dir . ' ' . $git_bin_path  . ' checkout -f production');

    // Log the deployment
    $commit_hash = shell_exec('cd ' . $repo_dir . ' && ' . $git_bin_path  . ' rev-parse --short HEAD');
    file_put_contents('deploy.log', date('m/d/Y h:i:s a') . " Deployed branch: " .  $branch . " Commit: " . $commit_hash . "\n", FILE_APPEND);
}
?>
```
