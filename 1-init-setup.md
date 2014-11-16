## Базовые настройки

Создать дроплет, зайти по ssh под root, сменить пароль root.

Создать нового пользователя:

```
$ adduser username
```

Добавить пользователя в группу sudo:
```
$ adduser username sudo
```

Залогиниться под новым пользователем.
```
$ login username
```

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
