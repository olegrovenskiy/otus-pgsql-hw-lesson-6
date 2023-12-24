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





