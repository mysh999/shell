## 一、role目录说明

/etc/ansible/roles下的角色目录

default：保存角色默认变量

files：普通文件

handlers：保存handlers

tasks：保存任务

templates：保存Jinja模块

meta：保存资源间的依赖关系

vars：保存变量

子目录间的文件，可以不加目录名任意调用





## 二、部署mysql示例

2.1、创建一个角色

```bash
# cd /etc/ansible/roles/
# ansible-galaxy init mysql_role

# ls
mysql_role
[root@localhost roles]# tree -L 3
.
└── mysql_role
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml

9 directories, 8 files
```



2.2、准备my.cnf

```bash
# cat ./templates/my.cnf.j2 
[mysqld]
port=3306
bind-address={{ ansible_all_ipv4_addresses[0] }}
max_connections=1500
log-bin = sz-bin
log-bin-index = sz-bin.index
binlog_format = row
expire_logs_days = 2
log-queries-not-using-indexes=1
skip-name-resolve

innodb_file_per_table = 1
innodb_buffer_pool_size = 1G
innodb_log_buffer_size = 16M
innodb_log_file_size = 512M
log-slave-updates = 1
read-only = 0
relay_log_recovery = 1

datadir=/var/lib/mysql

symbolic-links=0
socket=/var/lib/mysql/mysql.sock

[mysqld_safe]
socket=/var/lib/mysql/mysql.sock

[client]
socket=/var/lib/mysql/mysql.sock
```



2.3、准备任务文件

```bash
# cat ./tasks/main.yml 
---
# tasks file for mysql_role

- name: install MYSQL
  yum: name=mariadb-server state=present

- name: copy my.cnf
  template: src=my.cnf.j2 dest=/etc/my.cnf
  notify: restart mysql daemon

- name: start mysql daemon
  service: name=mariadb state=started enabled=yes
```





2.4、准备handlers文件

```bash
# cat ./handlers/main.yml 
---
# handlers file for mysql_role

- name: restart mysql daemon
  service: name=mariadb state=restarted
```





2.5、执行操作

```bash
# cat mysql.yaml 
- hosts: host1
  user: root
  roles:
     - mysql_role
```



执行

```bash
# ansible-playbook mysql.yaml

PLAY [host1] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************
ok: [10.10.65.153]

TASK [mysql_role : install MYSQL] ******************************************************************************************************************************************************
ok: [10.10.65.153]

TASK [mysql_role : copy my.cnf] ********************************************************************************************************************************************************
ok: [10.10.65.153]

TASK [mysql_role : start mysql daemon] *************************************************************************************************************************************************
ok: [10.10.65.153]

PLAY RECAP *****************************************************************************************************************************************************************************
10.10.65.153               : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@localhost mysql_role]# 
```

