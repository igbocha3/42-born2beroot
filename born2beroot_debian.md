# Born2beRoot с Debian OS

## Установка Debian

- Скачайте виртуальную машину VirtualBox [здесь](https://www.virtualbox.org/)
- Скачайте Debian OS [здесь](https://www.debian.org/)
- Смотрите настройку разделения диска (бонус) [здесь](https://youtu.be/OQEdjt38ZJA)


## Информация о разделах

### Обязательная часть

- sda (место на жестком диске) : 8 GB
  - sda1 [/boot] : 487 MB
  - sda5_crypt : max
    - wil--vg-root [/] : 2.8 GB
    - wil--vg-swap [SWAP] : 976 MB
    - wil--vg-home [/home] : 3.8 GB

### Бонус

- sda (место на жестком диске) : 31540 MB
  - sda1 [/boot] : 524.934 MB
  - sda5_crypt : max
    - LVMGroup-root [/] : 10.732 GB
    - LVMGroup-swap [SWAP] : 2.464 GB
    - LVMGroup-home [/home] : 5.339 GB
    - LVMGroup-var [/var] : 3.189 GB
    - LVMGroup-srv [/srv] : 3.189 GB
    - LVMGroup-tmp [/tmp] : 3.189 GB
    - LVMGroup-var--log [/var/log] : 4345.366 MB

## Sudo

### Шаг 1: Установите Sudo

Смените пользователя на учетную запись root, просто запустите `su` или `su -` без каких-либо аргументов

```sh
$ su
Password: (символы здесь не отображаются)
```

Установите sudo с помощью `apt install sudo`.

```sh
$ apt install sudo
```

Чтобы проверить, успешно ли установлен sudo, запустите `dpkg -l | grep sudo` или `sudo`.

```sh
$ dpkg -l | grep sudo
```

```sh
$ sudo
# если sudo установлен, он выведет краткую справку
# если нет, то появится сообщение "sudo command not found"
```

### Шаг 2: Добавление пользователя в группу sudo

Добавьте пользователя в группу sudo, выполнив одну из команд: `adduser <имя пользователя> sudo` или `usermod -aG sudo <имя пользователя>`

```sh
$ sudo adduser <имя пользователя> sudo
```

```sh
$ usermod -aG sudo <имя пользователя>
```

Чтобы проверить, был ли пользователь добавлен в sudo, выполните следующие команды:

```sh
$ groups <имя пользователя>
<имя пользователя> : <имя пользователя> cdrom floppy sudo audio dip video plugdev netdev bluetooth
$ getent group sudo
sudo:x:27:<имя пользователя>
$ id -Gn <имя пользователя>
<имя пользователя> cdrom floppy sudo audio dip video plugdev netdev bluetooth
```

Чтобы изменения вступили в силу, запустите `reboot` и войдите в систему под своим пользователем

```sh
$ reboot
<--->
Debian GNU/Linux 10 <имя хоста> tty1

<имя хоста> login: <имя пользователя>
Password: <password>
<--->
```

Чтобы проверить sudopowers пользователя, выполните следующие команды:

```sh
# Обычно используется для продления таймаута пароля sudo, но может быть использована для
# определения наличия у вас привилегий sudo. Она не покажет никакого сообщения, если пользователь
# имеет sudopowers
$ sudo -v
[sudo] password for <имя пользователя>: <пароль>
Sorry, user [имя пользователя] may not run sudo on [имя хоста]
```

```sh
# Будет выведен список всех привилегий sudo, которые у вас есть
$ sudo -l
[sudo] password for <имя пользователя>: <пароль>
```

### Шаг 3: Выполнение команд с привилегированным статусом root

Запускайте команды с правами root через префикс `sudo`. Например:

```sh
$ sudo apt install vim
# Vim (Vi IMproved) - это текстовый редактор, улучшенная версия vi
$ sudo apt update
# Обновить установленные пакеты
```

### Шаг 4: Настройка группы sudo

Создайте каталог `/var/log/sudo`. Сюда будет сохраняться файл журнала

```sh
$ sudo mkdir /var/log/sudo
```

Настройте sudo по этому пути `/etc/sudoers.d/<имя файла>`. `<имя файла>` не должно заканчиваться на `~` или содержать `.`

```sh
$ sudo vim /etc/sudoers.d/sudo_config

# Зачем использовать /etc/sudoers.d/ ?

# Обычно /etc/sudoers находится под контролем менеджера пакетов вашего дистрибутива.
# Если вы внесли изменения в этот файл и менеджер пакетов хочет обновить его,
# вам придется вручную проверять изменения и утверждать, как они будут объединены в
# новую версию. Поместив свои локальные изменения в файл в каталоге /etc/sudoers.d/
# вы избежите этого ручного шага, и обновление может происходить автоматически.
```

Отредактируйте файл `/etc/sudoers.d/<имя файла>`. Затем настройте новые правила sudo. Чтобы убедиться в этом, сразу после `Defaults` поставьте `TAB`

```sh
# Ограничить аутентификацию с помощью sudo до 3 попыток (по умолчанию все равно 3) в случае неправильного пароля
Defaults  passwd_tries=3

# Добавить пользовательское сообщение об ошибке в случае неправильного пароля
Defaults  badpass_message="Sorry, wrong password! Try again!"

# Записывать все команды sudo в /var/log/sudo/<имя файла>.
Defaults  logfile="/var/log/sudo/sudo.log"

# Чтобы архивировать все входы и выходы sudo в /var/log/sudo/:
# Каталог журнала ввода/вывода по умолчанию - /var/log/sudo-io
Defaults  log_input,log_output
Defaults  iolog_dir="/var/log/sudo"

# Требуется TTY:
# Зачем использовать tty? Если эксплуатируется какой-то не рутовый код (PHP скрипт, например), опция requiretty означает, что код эксплойта не сможет напрямую повысить свои привилегии, запустив sudo)
Defaults  requiretty

# Определить безопасные пути
Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```

## SSH и UFW

### Шаг 1: Установка и настройка SSH

Установите openssh-server

```sh
$ sudo apt install openssh-server
```

Проверьте, установлен ли openssh-server

```
$ ssh -V
```

```
$ dpkg -l | grep ssh
```

Настройте SSH в `/etc/ssh/sshd_config`

```
$ sudo vim /etc/ssh/sshd_config
```

Найдите и измените следующие строки в `/etc/ssh/sshd_config`

```
# Настройка SSH с использованием порта 4242
#Port22 -> Port 4242

# Запретить вход по SSH от имени root
#PermitRootLogin prohibit-password -> PermitRootLogin no
```

Проверьте статус SSH

```sh
$ sudo service ssh status
```

```sh
$ sudo systemctl status ssh
```

### Шаг 2: Установка и настройка UFW (Uncomplicated Firewall)

Установите UFW

```sh
$ sudo apt install ufw
```

Проверьте, установлен ли UFW

```sh
$ dpkg -l | grep ufw
```

Включите UFW (брандмауэр)

```sh
$ sudo ufw enable
```

Разрешите входящие соединения, используя порт `4242`

```sh
$ sudo ufw allow 4242
```

Проверьте состояние UFW

```sh
$ sudo ufw status
```

### Шаг 3: Подключение к серверу через SSH

Зайдите по SSH на виртуальную машину, используя порт `4242`. Чтобы проверить IP-адрес, используйте команду `ip addr show` или `ip a`.

```sh
$ ssh <имя пользователя>@<IP-адрес> -p 4242
```

В качестве альтернативы вы можете добавить правило переадресации на вашем компьютере

```
1. Перейдите в VirtualBox -> Выберите виртуальную машину -> Выберите настройки
2. Выберите "Network" -> "Adapter 1" -> "Advanced" -> "Port Forwarding"
```
Добавьте правило и установите значения `Host Port` и `Huest Port` на `4242`:

Перезапустите SSH-сервер

```sh
$ sudo systemctl restart ssh
```

Проверка состояния SSH

```sh
$ sudo service sshd status
```

Со стороны хоста с помощью iTerm2 или терминала введите, как показано ниже:

```sh
$ ssh <имя пользователя>@127.0.0.1 -p 4242
```

Чтобы выйти из SSH-соединения, используйте одну из следующих команд:

```
$ logout
```

```
$ exit
```

## Управление пользователями

### Шаг 1: Установите строгую политику паролей

#### Возраст пароля

Настройте политику возраста паролей с помощью команды: `sudo vim /etc/login.defs`

```sh
# Установите срок действия пароля каждые 30 дней
| строка 160 | PASS_MAX_DAYS 99999 -> PASS_MAX_DAYS 30

# Установите минимальное количество дней, разрешенных для изменения пароля, равное 2 дням
| строка 161 | PASS_MIN_DAYS 0 -> PASS_MIN_DAYS 2

# Посылать предупреждение пользователю за 7 дней до истечения срока действия пароля (по умолчанию 7 дней)
| строка 161 | PASS_WARN_AGE 7
```

#### Устойчивость пароля

Установите пакет `libpam-pwquality` для настройки политик надежности паролей

```sh
$ sudo apt install libpam-pwquality
```

Проверьте, успешно ли установлен пакет `libpam-pwquality`

```
$ dpkg -l | grep libpam-pwquality
```

Настройте политику надежности паролей, перейдя в файл `sudo vim /etc/pam.d/common-password`

```sh
25 password requisite pam_pwquality.so retry=3 <добавить политики здесь>
```

Выходные данные должны выглядеть следующим образом:

```
25 password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

Объяснение политик паролей

```sh
# Установите минимальную длину пароля в 10 символов
minlen=10

# Требуется наличие хотя бы одного символа в верхнем регистре и цифр.
# -1 означает, что требуется хотя бы один символ
ucredit=-1
dcredit=-1

# Установите максимум 3 последовательных одинаковых символа
maxrepeat=3

# Отклонять пароль, если он содержит имя пользователя
reject_username

# Установите число изменений, необходимых для нового пароля по сравнению со старым паролем, равное 7
difok=7

# Примените ту же политику к root
enforce_for_root
```

Установите новые пароли для уже созданных пользователей (включая root), следуя новой политике паролей:

```
$ passwd <- изменить пароль пользователя
$ sudo passwd <- изменить пароль root
```

Новые пароли

**flim** : `42AbuDhabi123!`

**root** : `42AbuDhabi123!`

### Шаг 2: Создание нового пользователя

Создайте нового пользователя

```sh
$ sudo adduser <имя пользователя>
```

Проверьте, был ли создан пользователь

```sh
getent passwd <имя пользователя>
```

Проверить информацию об истечении срока действия пароля пользователя

```sh
$ sudo chage -l <имя пользователя>
```

### Шаг 3: Создание новой группы

Создайте новую группу `user42`

```sh
$ sudo addgroup user42
```

Добавьте пользователя в группу `user42`, используя одну из следующих команд:

```sh
$ sudo adduser <имя пользователя> user42
```

```sh
$ sudo usermod -aG user42 <имя пользователя>
```

Проверьте, был ли пользователь добавлен в `user42`

```sh
$ getent group user42
```

## Cron

Установите инструменты netstat (для проверки TCP-соединений, которые будут использоваться в файле monitoring.sh)

```sh
$ sudo apt install net-tools
```

Смените пользователя на учетную запись root с помощью команды `su`

```sh
$ su
```

Затем вставьте команды bash с помощью команды `sudo vim /usr/local/bin/monitoring.sh`

Содержание файла `monitoring.sh` можно посмотреть [здесь](https://github.com/fidellim/42-Cursus-Project-Born2beRoot/blob/main/scripts/monitoring.sh)

Сделайте файл исполняемым

```sh
$ sudo chmod 755 /usr/local/bin/monitoring.sh
```

После добавления содержимого в `monitoring.sh`, вы можете вернуться к своей учетной записи `username` через `su <username>`. Затем протестируйте скрипт, выполнив его

```sh
$ sh /usr/local/bin/monitoring.sh
```

Если вы хотите увеличить загрузку процессора, выполните команду:

```sh
$ for i in $(seq $(getconf _NPROCESSORS_ONLN)); do yes > /dev/null & done
```

Чтобы остановить вышеуказанную программу, выполните команду:

```sh
$ pkill --signal STOP yes
```

Настройте cron от имени root

```sh
$ sudo crontab -u root -e
```

Запланируйте запуск сценария оболочки каждые 10 минут

```sh
# строка 23
# m h dom mon dow command
*/10 * * * * sh /usr/local/bin/monitoring.sh | wall
```

Проверьте запланированные задания cron для root

```sh
$ sudo crontab -u root -l
```

## Бонус

### Установка Wordpress с помощью Lighttpd, MariaDB, PHP

#### Шаг 1: Установите Lighttpd

Установите lighttpd

```sh
$ sudo apt install lighttpd
```

Проверьте, был ли установлен lighttpd

```sh
$ dpkg -l | grep lighttpd
```

Разрешить входящие соединения с использованием порта 80

```sh
$ sudo ufw allow 80
```

Добавьте новое правило переадресации портов для `Port 80`

#### Шаг 2: Установка и настройка MariaDB

Установите _mariadb-server_

```
$ sudo apt install mariadb-server
```

Проверьте, был ли успешно установлен _mariadb-server_

```
$ dpkg -l | grep mariadb-server
```

Запустите интерактивный скрипт для удаления небезопасных настроек по умолчанию через `sudo mysql_secure_installation`

```
$ sudo mysql_secure_installation

Enter current password for root (enter for none): #Просто нажмите Enter (не путайте корень базы данных с корнем системы)
Switch to unix-socket authentication [Y/n] n
Set root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

Войдите в консоль MariaDB через `sudo mariadb`

```
$ sudo mariadb
MariaDB [(none)]>
```

Создайте новую базу данных с помощью команды `CREATE DATABASE <имя базы данных>;`

```
MariaDB [(none)]> CREATE DATABASE wordpress;
```

Создайте нового пользователя базы данных и предоставьте ему полные привилегии на вновь созданную базу данных через `GRANT ALL ON <имя базы данных>.* TO '<имя пользователя>'@'localhost' IDENTIFIED BY '<пароль>' WITH GRANT OPTION;`

```
MariaDB [(none)]> GRANT ALL ON wordpress.* TO 'flim'@'localhost' IDENTIFIED BY 'flimAD' WITH GRANT OPTION;
```

Сбросьте привилегии

```
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Выход из оболочки MariaDB через `exit`

```
MariaDB [(none)]> exit
```

Проверьте, был ли пользователь базы данных успешно создан, войдя в консоль MariaDB через `mariadb -u <username-2> -p`

```
$ mariadb -u flim -p
Введите пароль: flimAD
MariaDB [(none)]>
```

Подтвердите, имеет ли пользователь базы данных доступ к базе данных через `SHOW DATABASES;`

```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| wordpress |
| information_schema |
+--------------------+
```

Выход из оболочки MariaDB через `exit`

```
MariaDB [(none)]> exit
```

#### Шаг 3: Установите PHP

Установите php-cgi и php-mysql

```sh
$ sudo apt install php-cgi php-mysql
```

Проверьте, успешно ли установлены php-cgi и php-mysql

```sh
$ dpkg -l | grep php
```

#### Шаг 4: Загрузка и настройка Wordpress

Установите _wget_

```
$ sudo apt install wget
```

Загрузите WordPress в `/var/www/html`

```
$ sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
```

Извлечение загруженного содержимого (tar-файл Wordpress)

```
$ sudo tar -xzvf /var/www/html/latest.tar.gz
```

Удалить tar-файл

```
$ sudo rm /var/www/html/latest.tar.gz
```

Копируем содержимое `/var/www/html/wordpress` в `/var/www/html`

```
$ sudo cp -r /var/www/html/wordpress/* /var/www/html
```

Удалите `/var/www/html/wordpress`

```
$ sudo rm -rf /var/www/html/wordpress
```

Создайте файл конфигурации WordPress из его образца с помощью `sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php`

```
$ sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```

Настройте WordPress для обращения к ранее созданной базе данных MariaDB и пользователю

```
$ sudo vi /var/www/html/wp-config.php
```

Замените команды ниже:

```
***ДО***
23 define( 'DB_NAME', 'database_name_here' );^M
26 define( 'DB_USER', 'имя_пользователя_здесь' );^M
29 define( 'DB_PASSWORD', 'password_here' );^M

***ПОСЛЕ***
23 define( 'DB_NAME', 'wordpress' );^M
26 define( 'DB_USER', 'flim' );^M
29 define( 'DB_PASSWORD', 'flimAD' );^M
```

Изменение прав доступа к папкам Wordpress

```sh
$ sudo chown -R www-data:www-data /var/www/html/
$ sudo chmod -R 755 /var/www/html/
```

#### Шаг 5: Настройка Lighttpd

Выполните следующие команды

```
$ sudo lighty-enable-mod fastcgi
$ sudo lighty-enable-mod fastcgi-php
$ sudo service lighttpd force-reload
```

#### Шаг 6: Проверка информации о PHP (ВАРИАНТ)

Добавьте код в `/var/www/html/php.php`

```php
<?php phpinfo(); ?>
```

Чтобы проверить информацию о PHP, перейдите в браузер и введите эту ссылку

```
localhost:80/php.php
```

#### Шаг 7: Открыть Wordpress

К этому времени вы сможете открыть и настроить Wordpress. Просто зайдите в браузер и введите эту ссылку

```sh
# Port-80 is used for HTTP connection by default.
# Either of the links should work
localhost:80 || localhost
```

### Cockpit (Free Choice Service)

Установите cockpit

```sh
$ sudo apt install cockpit
```

Разрешите `порт 9090` в брандмауэре

```sh
$ sudo ufw allow 9090
```

Добавьте новое правило перенаправления портов для `Port 9090`

Откройте cockpit в браузере, используя `localhost:9090`

```sh
localhost:9090
```

## Создайте Signature.txt

1. Перейдите к пути вашей виртуальной машины
   - `Windows`: %HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\
   - `Linux`: ~/VirtualBox VMs/
   - `MacM1`: ~/Library/Containers/com.utmapp.UTM/Data/Documents/
   - `MacOS`: ~/VirtualBox VMs
2. Выполните приведенную ниже команду для генерации подписи
   - `Windows`: certUtil -hashfile centos_serv.vdi sha1
   - `Linux`: sha1sum centos_serv.vdi
   - `Для Mac M1`: shasum Centos.utm/Images/disk-0.qcow2
   - `MacOS`: shasum centos_serv.vd
   - **Пример вывода**: 6e657c4619944be17df3c31faa030c25e43e40a
3. Скопируйте подпись и вставьте ее в файл с именем `signature.txt`
