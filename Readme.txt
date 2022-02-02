------------------------------------------------------------------------------------------------
1. Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова.
Файл и слово должны задаваться в /etc/sysconfig:

Создаем файл конфигурации для сервиса.
Заполняем его данными ```vi /etc/sysconfig/watchlog```:
# Configuration file for my watchlog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
word="ALARM"
log="/var/log/watchlog.log"

Создаем ```vi /var/log/watchlog.log``` и пишем туда строки:
ALARM
ALARM
ALARM
ALARM
ALARM
ALARM

Далее создаем скрипт sh.
Заполняем его данными ```vi /opt/watchlog.sh```:
#!/bin/bash
word=ALARM
log=/var/log/watchlog.log
date=`date`
if grep $word $log &> /dev/null
then
   logger "$date: I found word, Master!" # logger отправляет лог в системный журнал 
else
   exit 0
fi

Далее создаем unit для сервиса, создаем файл ```vi /etc/systemd/system/watchlog.service```:
[Unit]
Description=My watchlog service

[Service]
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/bin/bash '/opt/watchlog.sh'
Type=oneshot

[Install]
WantedBy=multi-user.target

Создаем unit для таймера (timer файл), ```vi /etc/systemd/system/watchlog.timer```:
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnBootSec=1m
OnUnitActiveSec=30s
Unit=watchlog.service

[Install]
WantedBy=timers.target

Даем права на исполение скрипта  ```chmod 777 /opt/watchlog.sh```
Проверить права ```ls -l file /opt/watchlog.sh```

systemctl daemon-reload
systemctl enable watchlog.service
systemctl enable watchlog.timer
systemctl start watchlog.timer
systemctl list-timers --all
tail -f /var/log/messages

Feb  2 10:25:49 localhost root: Wed Feb  2 10:25:49 UTC 2022: I found word, Master!
Feb  2 10:25:49 localhost systemd: Started watchlog.service.
Feb  2 10:26:25 localhost systemd: Starting watchlog.service...
Feb  2 10:26:25 localhost root: Wed Feb  2 10:26:25 UTC 2022: I found word, Master!
Feb  2 10:26:25 localhost systemd: Started watchlog.service.
Feb  2 10:27:25 localhost systemd: Starting watchlog.service...
Feb  2 10:27:25 localhost root: Wed Feb  2 10:27:25 UTC 2022: I found word, Master!
Feb  2 10:27:25 localhost systemd: Started watchlog.service.
Feb  2 10:28:25 localhost systemd: Starting watchlog.service...
Feb  2 10:28:25 localhost root: Wed Feb  2 10:28:25 UTC 2022: I found word, Master!
Feb  2 10:28:25 localhost systemd: Started watchlog.service.
Feb  2 10:29:25 localhost systemd: Starting watchlog.service...

------------------------------------------------------------------------------------------------
2. Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно так же называться:
Устанавливаем spawn-fcgi и необходимые для него пакеты ```yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y```
etc/rc.d/init.d/spawn-fcg - скрипт, который будет переписан
Заходим в /etc/sysconfig/spawn-fcgi и приводим его к следующему виду ```vi /etc/sysconfig/spawn-fcgi```:
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

Создаем unit файл ```vi /etc/systemd/system/spawn-fcgi.service```:
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

Убеждимся, что все успешно работает, выполним команды ```systemctl start spawn-fcgi``` и ```systemctl status spawn-fcgi```:

[root@Systemd ~]# systemctl start spawn-fcgi
[root@Systemd ~]# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-02-02 11:23:56 UTC; 6s ago
 Main PID: 3663 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─3663 /usr/bin/php-cgi
           ├─3665 /usr/bin/php-cgi
           ├─3666 /usr/bin/php-cgi
           ├─3667 /usr/bin/php-cgi
           ├─3668 /usr/bin/php-cgi
           ├─3669 /usr/bin/php-cgi
           ├─3670 /usr/bin/php-cgi
           ├─3671 /usr/bin/php-cgi
           ├─3672 /usr/bin/php-cgi
           ├─3673 /usr/bin/php-cgi
           ├─3674 /usr/bin/php-cgi
           ├─3675 /usr/bin/php-cgi
           ├─3676 /usr/bin/php-cgi
           ├─3677 /usr/bin/php-cgi
           ├─3678 /usr/bin/php-cgi
           ├─3679 /usr/bin/php-cgi
           ├─3680 /usr/bin/php-cgi
           ├─3681 /usr/bin/php-cgi
           ├─3682 /usr/bin/php-cgi
           ├─3683 /usr/bin/php-cgi
           ├─3684 /usr/bin/php-cgi
           ├─3685 /usr/bin/php-cgi
           ├─3686 /usr/bin/php-cgi
           ├─3687 /usr/bin/php-cgi
           ├─3688 /usr/bin/php-cgi
           ├─3689 /usr/bin/php-cgi
           ├─3690 /usr/bin/php-cgi
           ├─3691 /usr/bin/php-cgi
           ├─3692 /usr/bin/php-cgi
           ├─3693 /usr/bin/php-cgi
           ├─3694 /usr/bin/php-cgi
           ├─3695 /usr/bin/php-cgi
           └─3696 /usr/bin/php-cgi

Feb 02 11:23:56 Systemd systemd[1]: Started Spawn-fcgi startup service by Otus.
Feb 02 11:23:56 Systemd systemd[1]: Starting Spawn-fcgi startup service by Otus.

------------------------------------------------------------------------------------------------
3.Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами.
Для запуска нескольких экземпляров сервиса будем использовать шаблон в конфигурации файла окружения:
Создадим юнит файл для httpd ```vi /etc/systemd/system/httpd@.service```, заполним его:
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd-%I #(%I add)
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target

В самом файле окружения (которых будет два) задается опция для запуска веб-сервера с необходимым конфигурационным файлом
```vi /etc/sysconfig/httpd-first```:
OPTIONS=-f conf/first.conf

```vi /etc/sysconfig/httpd-second```
OPTIONS=-f conf/second.conf

В директории с конфигами httpd должны лежать два конфига, в нашем случае это будут first.conf и second.conf.

Для удачного запуска, в конфигурационных файлах должны бытя указаны уникальные для каждого экземпляра опции Listen и PidFile.
Конфиги можно скопировать:
```cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf```
```cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf```

И поправить только второй, в нем должны быть след опции ```vi /etc/httpd/conf/second.conf```: 
PidFile /var/run/httpd-second.pid 
Listen 8080 

Запустим сервисы 
```systemctl daemon-reload```
```systemctl start httpd@first.service```
```systemctl start httpd@second.service```
```systemctl status httpd@first.service```
```systemctl status httpd@second.service```



[root@Systemd conf]# systemctl status httpd@first.service
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-02-02 12:18:50 UTC; 13min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4540 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─4540 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─4541 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─4542 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─4543 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─4544 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─4545 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           └─4546 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

[root@Systemd conf]# systemctl status httpd@second.service
● httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-02-02 12:21:13 UTC; 13min ago
     Docs: man:httpd(8)
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4523 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           ├─4523 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─4528 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─4529 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─4530 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─4531 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           ├─4532 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           └─4533 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND


Установим пакет ```yum install net-tools```
[root@Systemd conf]# netstat -tunap
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      7837/rpcbind
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      7491/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      7672/master
tcp        0      0 10.0.2.15:22            10.0.2.2:5773           ESTABLISHED 1412/sshd: vagrant
tcp6       0      0 :::111                  :::*                    LISTEN      7837/rpcbind
tcp6       0      0 :::80                   :::*                    LISTEN      4540/httpd
tcp6       0      0 :::8080                 :::*                    LISTEN      4023/httpd