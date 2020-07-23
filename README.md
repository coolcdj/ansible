
主机名       IP
centos7-1   192.168.0.11
centos7-2   192.168.0.12

将centos7-1   192.168.0.11作为ansible管理端，安装好ansible以后使用ssh key-gen与被管理端建立免密钥连接


[root@centos7-1 ~]# ansible 192.168.0.12 -m service -a "name=firewalld state=stopped enabled=0"

[root@centos7-1 ~]# vim /opt/ansible/mysqlinstall.yml

- hosts: mysql
  remote_user: root
  roles:
    - mysql

[root@centos7-1 ~]# vim /opt/ansible/mysql/files/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
[mysql]
default-character-set=utf8
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
validate_password_policy=0
validate_password_length=1
validate_password_special_char_count=0
validate_password_number_count=0
validate_password_mixed_case_count=0
lower_case_table_names=1
max_connections=50000
!includedir /etc/my.cnf.d

[root@centos7-1 ~]# vim /opt/ansible/mysql/files/setpassword.sh

#!/bin/bash
#This script is set MySQL password for the frist time!
mysqlinitpasswd=`grep 'temporary password' /var/log/mysqld.log |awk '{print $11}'`
mysql -uroot -p${mysqlinitpasswd} -S /var/lib/mysql/mysql.sock -e "set global validate_password_policy=0;set global validate_password_length=1;set global validate_password_policy=0;set glob
al validate_password_length=1;set password = password('123');grant all privileges on *.* to 'root' @'%' identified by '123';flush privileges;" --connect-expired-password

[root@centos7-1 ~]# ls /opt/ansible/mysql/files/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz

/opt/ansible/mysql/files/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz

[root@centos7-1 ~]# vim /opt/ansible/mysql/tasks/main.yml

- name: unarchive Mysql
unarchive: src=mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz dest=/usr/local/ owner=root group=root
- name: mkdir mysql datadir
file: path=/opt/mysql/data owner=root group=root state=directory
- name: mkdir mysql log dir
file: path=/opt/mysql/log owner=root group=root state=directory
- name: mkdir mysql tmp dir
file: path=/opt/mysql/tmp owner=root group=root state=directory
- name: change directory
command: mv /usr/local/mysql-5.7.30-linux-glibc2.12-x86_64 /usr/local/mysql
- name: init.d mysql file
 command: cp /usr/local/mysql/support-files/mysql.server /etc/init.d/
- name: cp my.cnf
copy: src=my.cnf dest=/etc/my.cnf
- name: initid mysql-server
command: /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --user=root
- name: start mysql
command: /usr/local/mysql/support-files/mysql.server start
- name: set password
copy: src=setpassword.sh dest=/opt/mysql/ mode=755
- name: sh setpassword
shell: sh /opt/mysql/setpassword.sh
- name: mysql binary command
command: cp /usr/local/mysql/bin/* /usr/bin/

[root@centos7-1 ~]# ansible-playbook /etc/ansible/roles/mysqlinstall.yml


PLAY [mysql] ***************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************
ok: [192.168.0.12]

TASK [mysql : unarchive Mysql] *********************************************************************************************
changed: [192.168.0.12]

TASK [mkdir mysql datadir] *************************************************************************************************
changed: [192.168.0.12]

[root@centos7-2 ~]# mysql -uroot -p

Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit
Bye

