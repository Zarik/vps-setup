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
