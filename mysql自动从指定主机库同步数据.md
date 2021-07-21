## 一、需求说明

需要定时从指定主机同步mysql库数据到本地，再导入到本地



## 二、脚本内容

```bash
$ crontab  -l
*/30 * * * * /home/miyong.shu/rsync_mysql.sh 2>&1

#!/bin/bash
SRV="102.74.170.76"
USER=managers
SOURCEDIR=/opt/mysqldump

function dump_file()
{
expect <<!
spawn ssh $USER@$SRV
send "find $SOURCEDIR -mmin -10|grep -i sql|xargs -i rm -rf {}\r"
send "mysqldump -u root --password='password' ai_edu_platform>$SOURCEDIR/ai_edu_platform_`date +%Y%m%d`.sql\r"
send "mysqldump -u root --password='password' emsdb user_info>$SOURCEDIR/user_info_`date +%Y%m%d`.sql\r"
send "mysqldump -u root --password='password' emsdb class_info>$SOURCEDIR/class_info_`date +%Y%m%d`.sql\r"
expect eof
!
}

function copy_file()
{
find /home/miyong.shu/sql -mmin -10|grep -i sql|xargs -i rm -rf {}
rsync -av $USER@$SRV:$SOURCEDIR/* /home/miyong.shu/sql
}


function import_sql()
{
asqlname="ai_edu_platform_`date +%Y%m%d`.sql"
bsqlname="user_info_`date +%Y%m%d`.sql"
csqlname="class_info_`date +%Y%m%d`.sql"
dir="/home/miyong.shu/sql"
host="127.0.0.1"
user="root"
passwd="password"
adbname="ai_edu_platform"
bdbname="emsdb"

mysql -h$host -u$user -p$passwd $adbname <$dir/$asqlname
mysql -h$host -u$user -p$passwd $bdbname <$dir/$bsqlname
mysql -h$host -u$user -p$passwd $bdbname <$dir/$csqlname
}


function main()
{
        dump_file
        copy_file
        import_sql
}
main

```

