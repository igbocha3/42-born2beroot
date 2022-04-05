# Команды Born2beRoot

## SSH

Запустите OpenSSH

```sh
$ sudo /etc/init.d/ssh start
$ sudo service ssh start
$ sudo systemctl start ssh
```

Остановите OpenSSH

```sh
$ sudo /etc/init.d/ssh stop
$ sudo service ssh stop
$ sudo systemctl stop ssh
```

Перезапустите OpenSSH

```sh
$ sudo /etc/init.d/ssh restart
$ sudo service ssh restart
$ sudo systemctl restart ssh
```

Состояние OpenSSH

```sh
$ sudo /etc/init.d/ssh status
$ sudo service ssh status
$ sudo systemctl status ssh
```

Включение и запуск OpenSSh во время загрузки

```sh
$ sudo systemctl enable ssh
```

Проверьте, включен ли OpenSSH во время загрузки

```sh
$ sudo systemctl is-enabled ssh
```

## UFW

Отображение правил UFW

```
$ sudo ufw status numbered
$ sudo ufw status
```

Включить UFW

```
$ sudo ufw enable
$ sudo systemctl enable ufw
```

Отключить UFW

```
$ sudo ufw disable
```

Добавьте правило UFW

```
$ sudo ufw allow <номер порта>
```

Удалить правило UFW

```
$ sudo ufw delete <номер правила>
$ sudo ufw delete allow <номер порта>
```

## Имя хоста

Показать текущую информацию о хосте

```sh
$ sudo hostnamectl status
```

Изменить имя хоста

```sh
$ sudo hostnamectl set-hostname <новое имя хоста>
```

```sh
$ sudo vim /etc/hostname
```

Измените файл `etc/hosts`

```
$ sudo vim /etc/hosts
```

Измените `старое имя хоста` на `новое имя хоста`

```
127.0.0.1 localhost
127.0.0.1 new_hostname
```

Перезагрузитесь и проверьте изменения

```
$ sudo reboot
```

## Пользователь

- `less /etc/passwd | cut -d ":" -f 1` - показать список всех пользователей на компьютере;
- `users` - показать список всех пользователей, которые в данный момент вошли в систему;
- `useradd <username>` - создать нового пользователя с домашним каталогом;
- `usermod <username>` - изменить настройки пользователя, `-l` для имени пользователя, `-c` для комментариев/полного имени и `-g` для GID;
- `userdel -r <username>` - удаляет пользователя и все прикрепленные к нему файлы;
- `id -u <username>` - показывает UID пользователя.

## Группа

- `less /etc/group | cut -d ":" -f 1` - показать список всех пользователей на компьютере;
- `groups <username>` - показывает группы пользователя;
- `groupadd <имя группы>` - создать новую группу;
- `groupdel <имя группы>` - удалить группу;
- `gpasswd -a <username> <groupname>` - добавляет пользователя в группу;
- `gpasswd -d <username> <groupname>` - удаляет пользователя из группы;
- `getent group <groupname>` - показать пользователей в группе;
- `id -g <username>` - показать GID основной группы пользователя.