```bash
#!/usr/bin/env bash

function a_config_parameter(){
    cat /etc/security/limits.conf |grep -i "* soft nofile 65535"
	if [ $? -eq 0 ];then
	    echo "have configure limits file"
	else
		echo -e '* soft nofile 65535 \n* hard nofile 65535' > /etc/security/limits.conf	
	fi
}



function b_selinx_set_down() {
	sed -i '/^SELINUX/ s/\(SELINUX=\).*/\1disabled/g' /etc/selinux/config
	setenforce 0
}

function c_firewalld_rules() {
	systemctl enable firewalld && systemctl start firewalld
	firewall-cmd --add-rich-rule="rule family="ipv4" source address="0.0.0.0/0" accept" --permanent
	iptables-save
	systemctl disable firewalld && systemctl stop firewalld
}



function d_disable_password_login() {
	# 禁止密码登录
    sed -i '/^PasswordAuthentication no/ s/yes/no/g' /etc/ssh/sshd_config 
    # SSH Session超时
    sed -i '/^#ClientAliveCountMax/ a\\nClientAliveInterval 600\nClientAliveCountMax 0\n' /etc/ssh/sshd_config 
}



function e_install_base_packages() {
    echo 'step.1 install base packages'
cat > nginx.repo <<\EOF
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
    yum install bash-completion lrzsz wget net-tools sysstat htop jq iotop bind-utils psmisc strace bzip2 bzip2-devel sqlite-devel httpd-tools numactl gcc gcc-c++ ncurses-devel rsync pcre-devel openssl-devel gcc curl unzip telnet ntp -y -q
}



function f_ntp_config() {
	cat > /etc/ntp.conf  <<EOF
        server ntp-sz.ch1.la prefer
        server 0.cn.pool.ntp.org
        server ntp1.aliyun.com iburst minpoll 4 maxpoll 10
        restrict ntp1.aliyun.com nomodify notrap nopeer noquery
        server ntp1.cloud.aliyuncs.com iburst minpoll 4 maxpoll 10
        restrict ntp1.cloud.aliyuncs.com nomodify notrap nopeer noquery
        server 127.127.1.0 iburst local clock       
EOF
	systemctl restart ntpd.service && systemctl enable ntpd.service
	test  $? -eq 0  && echo "Install NTP Completion" || exit 1
	ntpstat
	ntpq -p
}



function g_collect_operation_log () {
    echo 'step.2 Collect operation log'
cat >> /etc/bashrc << \EOF
LOGINUSER=`who am i|awk '{print $1}'`
export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger -p local2.info "{\"Loginer\":\"${LOGINUSER}\","\"Executor\":\"$(whoami)\","\"Path\":\"`pwd`\"","\"Command\":\"$msg\",\"At\":\"$(date)\",\"Host\":\"`hostname`\"}";}'
EOF
    echo 'local2.* /var/log/todo.log' >> /etc/rsyslog.conf&& service rsyslog restart
}



function h_upgrade_kernal() {
    yum update -y
    test $? -eq 0 && reboot || exit 1
}

function main() {
    a_config_parameter
    b_selinx_set_down 
    c_firewalld_rules 
    d_disable_password_login 
    e_install_base_packages 
    f_ntp_config 
    g_collect_operation_log  
    h_upgrade_kernal   
}

main
```

