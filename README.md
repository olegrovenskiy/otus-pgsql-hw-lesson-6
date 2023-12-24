# otus-pgsql-hw-lesson-6

Инсталировал PostgreSQL-15 на ВМ в виртуализации VMware, Centos-7, так как это корпоративный стандарт, в компании где работаю.


1. Centos 7 10.102.6.27

2. yum update

Использовал информацию с официальных рессурсов:

https://www.postgresql.org/download/linux/redhat/
https://orcacore.com/install-postgresql-15-centos-7/

Набор комманд:

3. yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

4. yum install -y postgresql15-server

получил ошибку, отсутствует пакет

           Error: Package: postgresql15-15.5-1PGDG.rhel7.x86_64 (pgdg15)
                      Requires: libzstd >= 1.4.0

для решения нашёл соответствующий пакет на https://pkgs.org/ (нужный пакет в соответствии с архитектурой)

6. yum localinstall https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libzstd-1.5.5-1.el7.x86_64.rpm


7. yum install -y postgresql15-server

                      Installed:
                        postgresql15-server.x86_64 0:15.5-1PGDG.rhel7
                      
                      Dependency Installed:
                        libicu.x86_64 0:50.2-4.el7_7                             postgresql15.x86_64 0:15.5-1PGDG.rhel7
                        postgresql15-libs.x86_64 0:15.5-1PGDG.rhel7



8. Проверка инсталяции и версии командой psql -V

                              [root@mck-network-test ~]#  psql -V
                              psql (PostgreSQL) 15.5
                              [root@mck-network-test ~]#


9. Запуск сервиса БД и инициализация

        [root@mck-network-test ~]# postgresql-15-setup initdb
        Initializing database ... OK
        
        [root@mck-network-test ~]#
        [root@mck-network-test ~]#
        [root@mck-network-test ~]# systemctl enable postgresql-15
        Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-15.service to /usr/lib/systemd/system/postgresql-15.service.
        [root@mck-network-test ~]# systemctl start postgresql-15
        [root@mck-network-test ~]# systemctl status postgresql-15
        ● postgresql-15.service - PostgreSQL 15 database server
           Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
           Active: active (running) since Fri 2023-12-01 03:09:52 EST; 9s ago
             Docs: https://www.postgresql.org/docs/15/static/
          Process: 24300 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
         Main PID: 24306 (postmaster)
           CGroup: /system.slice/postgresql-15.service
                   ├─24306 /usr/pgsql-15/bin/postmaster -D /var/lib/pgsql/15/data/
                   ├─24308 postgres: logger
                   ├─24309 postgres: checkpointer
                   ├─24310 postgres: background writer
                   ├─24312 postgres: walwriter
                   ├─24313 postgres: autovacuum launcher
                   └─24314 postgres: logical replication launcher
        
        Dec 01 03:09:52 mck-network-test.mgc.local systemd[1]: Starting PostgreSQL 15 database server...
        Dec 01 03:09:52 mck-network-test.mgc.local postmaster[24306]: 2023-12-01 03:09:52.256 EST [24306] LOG:  re...ss
        Dec 01 03:09:52 mck-network-test.mgc.local postmaster[24306]: 2023-12-01 03:09:52.256 EST [24306] HINT:  F...".
        Dec 01 03:09:52 mck-network-test.mgc.local systemd[1]: Started PostgreSQL 15 database server.
        Hint: Some lines were ellipsized, use -l to show in full.
        [root@mck-network-test ~]#




открытие 5432 для внешних пользователей

https://www.dmosk.ru/miniinstruktions.php?mini=pgsql-remote&ysclid=lpmjvsqk9g399750990
+
firewall
https://www.server-world.info/en/note?os=CentOS_Stream_9&p=postgresql15&f=2


To change the PostgreSQL user's password, follow these steps:

log in into the psql console:

sudo -u postgres psql
Then in the psql console, change the password and quit:

postgres=# \password postgres
Enter new password: <new-password>
postgres=# \q

смог подключиться PgAdminom

Создание таблицы

postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=#
postgres=# select * from test;
 c1
----
 1
(1 row)

postgres=#

Остановка сервиса

[root@mck-network-test postgres]# systemctl stop postgresql-15
[root@mck-network-test postgres]#
[root@mck-network-test postgres]# systemctl status postgresql-15
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Sun 2023-12-24 05:52:42 EST; 8s ago
     Docs: https://www.postgresql.org/docs/15/static/
  Process: 1204 ExecStart=/usr/pgsql-15/bin/postmaster -D ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 1204 (code=exited, status=0/SUCCESS)

Dec 01 06:55:35 mck-network-test.mgc.local systemd[1]: Starting PostgreSQL 15 database server...
Dec 01 06:55:36 mck-network-test.mgc.local postmaster[1204]: 2023-12-01 06:55:36.262 EST [1204] LOG:  redirecting log output to logg...ocess
Dec 01 06:55:36 mck-network-test.mgc.local postmaster[1204]: 2023-12-01 06:55:36.262 EST [1204] HINT:  Future log output will appear...log".
Dec 01 06:55:36 mck-network-test.mgc.local systemd[1]: Started PostgreSQL 15 database server.
Dec 24 05:52:42 mck-network-test.mgc.local systemd[1]: Stopping PostgreSQL 15 database server...
Dec 24 05:52:42 mck-network-test.mgc.local systemd[1]: Stopped PostgreSQL 15 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@mck-network-test postgres]#


Добавление диска к ВМ

На Вицентр добавил к ВМ новый диск на 20Г


с помошщью fdisk -l видим что диск как устройство появился

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

и через lsblk

[root@mck-network-test postgres]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  100G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   29G  0 part
  ├─centos-root 253:0    0   27G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0   20G  0 disk
sr0              11:0    1 1024M  0 rom


Созданы партиции и диск отформатирован

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x24287bbd

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    41943039    20970496   83  Linux
[root@mck-network-test postgres]#

и готов к использованию

Примонтуруем к /mnt/data

[root@mck-network-test mnt]# mount /dev/sdb1 /mnt/data
[root@mck-network-test mnt]#
[root@mck-network-test mnt]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G   20M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   27G  3.1G   24G  12% /
/dev/sda1               1014M  198M  817M  20% /boot
tmpfs                    783M     0  783M   0% /run/user/0
tmpfs                    783M     0  783M   0% /run/user/26
overlay                   27G  3.1G   24G  12% /var/lib/docker/overlay2/e5b17778c5a7265816eef0922f4d24a98ce647f86106cbf38331c0d25ef415d5/merged
/dev/sdb1                 20G   33M   20G   1% /mnt/data
[root@mck-network-test mnt]#

После ребута раздел слетел, для исправления добавил в файл /etc/fstab

/dev/sdb1 /mnt/data xfs defaults 0 0

После чего ребут прошёл ок

[root@mck-network-test ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G  1.1M  3.9G   1% /dev/shm
tmpfs                    3.9G   12M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   27G  3.1G   24G  12% /
/dev/sdb1                 20G   33M   20G   1% /mnt/data
/dev/sda1               1014M  198M  817M  20% /boot
tmpfs                    783M     0  783M   0% /run/user/0
[root@mck-network-test ~]#


сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

Сделал перенос директории

postgres=# SHOW data_directory;
     data_directory
------------------------
 /var/lib/pgsql/15/data
(1 row)

mv /var/lib/pgsql/15 /mnt/data

[root@mck-network-test data]# ls -l
total 0
drwx------. 4 postgres postgres 51 Dec  1 03:09 15
[root@mck-network-test data]#

[root@mck-network-test data]# cd /var/lib/pgsql
[root@mck-network-test pgsql]# ls -l
total 0
[root@mck-network-test pgsql]#

Директория действительно переехала

Запускаю базу

[root@mck-network-test postgres]# systemctl start postgresql-15
Job for postgresql-15.service failed because the control process exited with error code. See "systemctl status postgresql-15.service" and "journalctl -xe" for details.
[root@mck-network-test postgres]#

Сервис не запустился

https://dev.to/fitodic/how-to-change-postgresql-s-data-directory-on-linux-2n2b

vim /lib/systemd/system/postgresql.service

Environment=PGDATA=/home/pgdata/data





root@mck-network-test 15]# systemctl daemon-reload
[root@mck-network-test 15]#
[root@mck-network-test 15]# systemctl start postgresql-15
[root@mck-network-test 15]#
[root@mck-network-test 15]# systemctl status postgresql-15
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-12-24 08:58:50 EST; 10s ago
     Docs: https://www.postgresql.org/docs/15/static/
  Process: 12116 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 12122 (postmaster)
    Tasks: 8
   Memory: 27.3M
   CGroup: /system.slice/postgresql-15.service
           ├─12122 /usr/pgsql-15/bin/postmaster -D /mnt/data/15/data/
           ├─12124 postgres: logger
           ├─12125 postgres: checkpointer
           ├─12126 postgres: background writer
           ├─12128 postgres: walwriter
           ├─12129 postgres: autovacuum launcher
           ├─12130 postgres: logical replication launcher
           └─12138 postgres: postgres postgres 10.102.1.108(35478) idle

Dec 24 08:58:50 mck-network-test.mgc.local systemd[1]: Starting PostgreSQL 15 database server...
Dec 24 08:58:50 mck-network-test.mgc.local postmaster[12122]: 2023-12-24 08:58:50.817 EST [12122] LOG:  redirecting log output t...ocess
Dec 24 08:58:50 mck-network-test.mgc.local postmaster[12122]: 2023-12-24 08:58:50.817 EST [12122] HINT:  Future log output will ...log".
Dec 24 08:58:50 mck-network-test.mgc.local systemd[1]: Started PostgreSQL 15 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@mck-network-test 15]#

После чего смог подключиться к Базе

[root@mck-network-test 15]# su postgres
bash-4.2$
bash-4.2$
bash-4.2$ psql
psql (15.5)
Type "help" for help.

postgres=#
postgres=#
postgres=# select * from test;
 c1
----
 1
(1 row)

postgres=#

postgres=# SHOW data_directory;
  data_directory
-------------------
 /mnt/data/15/data
(1 row)

postgres=#























