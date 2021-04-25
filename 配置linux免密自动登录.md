## 一、需求说明

以Linux A主机为跳板，需要免密登录到B、C、D等主机



## 二、配置步骤

2.1、在A主机生成密钥

```bash
# ssh-keygen -t rsa   #一路回车
```

2.2、将生成的密钥文件拷贝至B、C、D等主机的/root/.ssh/authorized_keys文件里



2.3、在A主机创建自动登录脚本

```bash
$ cat login.sh 
#!/usr/bin/env bash

#连接主机
function connect_host() {
    ssh -p 22 root@$1
}


function menu {
    clear
    echo
    echo -e "welcome login"
    echo -e "1. master-1"
    echo -e "2. node-1"
    echo -e "3. node-2"
    echo -e "0. Exit menu\n\n"
    #-en 选项会去掉末尾的换行符，这让菜单看起来更专业一些
    echo -en "Enter option:"
    #read 命令读取用户输入
    read -n 1 option
}

menu
case $option in
0)
    exit ;;
1)
    connect_host 192.168.188.61  ;;
2)
    connect_host 192.168.188.62 ;;
3)
    connect_host 192.168.188.63 ;;
*)
    clear
    echo "sorry,wrong selection" ;;
esac

echo -en "thit any to contunue"
```



2.4、登录效果

```bash
$ ./login.sh 

welcome login
1. master-1
2. node-1
3. node-2
0. Exit menu


Enter option:1Last login: Sun Apr 25 22:34:08 2021 from 192.168.188.1
'[root@k8s-master ~]# '

```

