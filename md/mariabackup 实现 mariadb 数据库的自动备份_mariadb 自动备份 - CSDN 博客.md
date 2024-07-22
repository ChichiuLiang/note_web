> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u010792184/article/details/129313716)

安装：

yum -y install MariaDB-backup

1、先进行一次全量备份：  
mariabackup -uroot -pmypassword --backup --target-dir=/backup/base  
2、进行一次昨天的增量备份：  
mariabackup -uroot -pmypassword --backup --target-dir=/backup/20221205 --incremental-basedir=/backup/base  
3、进行一次今天的增量备份：  
mariabackup -uroot -pmypassword --backup --target-dir=/backup/20221206 --incremental-basedir=/backup/20221205  
4、/root 目录下编辑自动备份脚本：  
vim /root/sydwMariadbBackup.sh  
输入以下内容：  
#示例 1  
#!/bin/bash  
the_day_before_increase_folder=/backup/$(date -d '1 days ago' +'%Y%m%d%H%M%S')  
increase_folder=/backup/$(date +'%Y%m%d%H%M%S')  
mariabackup -uroot -pmypassword --backup --target-dir=$increase_folder --incremental-basedir=$the_day_before_increase_folder  
#示例 2：  
#!/bin/bash  
backup_folder=/backup  
previous_folder=predata  
current_folder=curdata  
history_folder=$(date -d '120 minute ago' +'%Y%m%d%H%M')  
cd $backup_folder  
mariabackup -uroot -pmypassword --backup --target-dir=/backup/$current_folder --incremental-basedir=/backup/$previous_folder  
mv $previous_folder $history_folder  
mv $current_folder $previous_folder  
#tar -zcvf $history_folder.tar.gz $history_folder

#示例 3：如果定时任务不能执行，可加入 source /etc/profile 和 source ~/.bash_profile

#!/bin/bash  
source /etc/profile  
source ~/.bash_profile  
backup_folder=/backup/incrementalData  
previous_folder=predata  
current_folder=curdata  
history_folder=$(date -d '1 days ago' +'%Y%m%d')  
cd $backup_folder  
mkdir curdata  
[xtrabackup](https://so.csdn.net/so/search?q=xtrabackup&spm=1001.2101.3001.7020) --backup --user=ljcsqmgc --password=hnCS#hnjj0908 --target-dir=/backup/incrementalData/$current_folder  --incremental-basedir=/backup/incrementalData/$previous_folder  
mv $previous_folder $history_folder  
mv $current_folder $previous_folder  
保存  
chmod 775 /root/sydwMariadbBackup.sh  
5、添加 [linux 定时任务](https://so.csdn.net/so/search?q=linux%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1&spm=1001.2101.3001.7020)：  
[crontab](https://so.csdn.net/so/search?q=crontab&spm=1001.2101.3001.7020) -e  
添加一行：  
10 1 * * * sh /root/sydwMariadbBackup.sh 2>&1 > /root/mariadbBackup/log_$(date +\%Y-\%m-\%d)  
systemctl start crond  
数据还原：  
备份文件复制到目标服务器后  
mariabackup --prepare --target-dir=/data/backup/base  
mariabackup --prepare --target-dir=/data/backup/base --incremental-dir=/data/backup/20221205  
自动执行：  
#!/bin/bash  
base_folder=/data/basics/base  
for file in /data/backup/*  
do  
if [-d "$file"] && [ "$file" != "/data/backup/predata" ] && [ "$file" != "/data/backup/curdata" ]  
then  
# echo "$file"  
mariabackup --prepare --target-dir=$base_folder --incremental-dir=$file  
mv $file /data/historyBackup/  
fi  
done  
mariabackup --copy-back --target-dir=/data/basics/base  
chown -R mysql:mysql /data/mariadb

xtrabackup 实现 mysql 备份还原：

1、安装：下载 percona-xtrabackup-2.4.8-Linux-x86_64.tar.gz 安装文件上传到服务器，解压到 / usr/local/xtrabackup 

2、设置环境变量：export PATH=$PATH:/usr/local/xtrabackup/bin

3、全量备份：innobackupex --defaults-file=/etc/my.cnf --user=root --password=pwd --socket=/var/lib/mysql/mysql.sock /backup/base

或者：

xtrabackup --backup --user=root --password=password --target-dir=/database/base

4、增量备份：innobackupex --defaults-file=/etc/my.cnf --user=root --password=pwd --socket=/var/lib/mysql/mysql.sock --no-timestamp --incremental-basedir=/backup/base  --incremental /backup/incrementalData/predata

或者：

xtrabackup --backup --user=root --password=password --target-dir=/database/inc1 --incremental-basedir=/database/base

5、自动备份：

#!/bin/bash  
backup_folder=/backup/incrementalData  
previous_folder=predata  
current_folder=curdata  
history_folder=$(date -d '1 days ago' +'%Y%m%d%H')  
cd $backup_folder  
#xtrabackup --backup --user=root --password=pwd --target-dir=/backup/incrementalData/$current_folder  --incremental-basedir=/backup/incrementalData/$previous_folder  
innobackupex --defaults-file=/etc/my.cnf --user=root --password=pwd --socket=/var/lib/mysql/mysql.sock --no-timestamp --incremental-basedir=/backup/incrementalData/$previous_folder  --incremental /backup/incrementalData/$current_folder  
mv $previous_folder $history_folder  
mv $current_folder $previous_folder

3、全量还原： innobackupex --apply-log  --redo-only /yyzjbackup/yyzjbx/base

或者

xtrabackup --prepare --apply-log-only --target-dir=/database/base

4、增量还原： innobackupex --apply-log  --redo-only /yyzjbackup/yyzjbx/base --incremental-dir=/database/inc1

或者

xtrabackup --prepare --apply-log-only --target-dir=/database/base --incremental-dir=/database/inc1

5、自动还原：

#!/bin/bash  
source /etc/profile  
ID=`ps -ef | grep mariadb102 | grep -v grep | awk '{print $2}'`  
echo $ID  
for id in $ID  
do  
    kill -9 $id  
    echo "kill $id"  
done

for file in /winshare/yyzjbx/incrementalPacked/*  
do  
    if !([-d "$file"])  
    then  
        echo "$file"  
        mkdir /yyzjbackup/yyzjbx/incrementData/${file:35:10}  
        tar  -zxvf $file -C /yyzjbackup/yyzjbx/incrementData/${file:35:10}  
        mv $file  /winshare/yyzjbx/historyIncrementalPacked/  
    fi  
done  
cd /usr/local/xtrabackup/bin  
for file1 in /yyzjbackup/yyzjbx/incrementData/*  
do  
        if [-d "$file1"]  
        then  
                echo "$file1"  
                #xtrabackup --prepare --apply-log-only --target-dir=/yyzjbackup/yyzjbx/base --incremental-dir=$file1  
        innobackupex --apply-log  --redo-only /yyzjbackup/yyzjbx/base --incremental-dir=$file1  
                mv $file1 /yyzjbackup/yyzjbx/historyIncrementalData/  
        fi  
done  
cd /yyzjbackup/mariadb102/bin  
nohup ./mysqld_safe --defaults-file=/yyzjbackup/mariadb102/my.cnf --user=root &

压缩方式备份还原：

第一次全量：

#!/bin/bash  
current_day=$(date +"%Y-%m-%d")  
mkdir /winshare/sydwpx/$current_day  
mariabackup --backup --extra-lsndir=/winshare/sydwpx/$current_day -uroot -p****  --stream=xbstream | gzip > /winshare/sydwpx/$current_day/sydwpx.gz

增量：

#!/bin/bash  
current_day=$(date +"%Y-%m-%d")  
mkdir /winshare/sydwpx/$current_day  
prev_day=$(date -d "yesterday" +"%Y-%m-%d")  
mariabackup --backup --history --incremental-basedir=/winshare/sydwpx/$prev_day --extra-lsndir=/winshare/sydwpx/$current_day -uroot -p*** --stream=xbstream | gzip > /winshare/sydwpx/$current_day/sydwpx.gz   
 

全量还原：

#!/bin/bash  
cd /usr/nginx/sbin  
sed -i s/sydwpx-gateway:9998/sydwpx-gateway:9999/g `grep sydwpx-gateway:9998 -rl --include="nginx.conf" ../conf/`  
./nginx -s reload  
ID=`ps -ef | grep mariadb10.3 | grep -v grep | awk '{print $2}'`  
echo $ID  
for id in $ID  
do  
    kill -9 $id  
    echo "kill $id"  
done

current_day=$(date +"%Y-%m-%d")  
rm -rf /dianda/sydwDatabase/*  
cd /mariadb10.3/mariadb-10.3.37-linux-x86_64/bin  
zcat /winshare/sydwpx/$1/sydwpx.gz | ./mbstream -x -C /dianda/sydwDatabase/  
./mariabackup --prepare --target-dir /dianda/sydwDatabase/ 

cd /mariadb10.3/mariadb-10.3.37-linux-x86_64/bin  
nohup ./mysqld_safe --defaults-file=/mariadb10.3/mariadb-10.3.37-linux-x86_64/my.cnf --user=root &

cd /usr/nginx/sbin  
sed -i s/sydwpx-gateway:9999/sydwpx-gateway:9998/g `grep sydwpx-gateway:9999 -rl --include="nginx.conf" ../conf/`  
./nginx -s reload

增量还原：

#!/bin/bash  
cd /usr/nginx/sbin  
sed -i s/sydwpx-gateway:9998/sydwpx-gateway:9999/g `grep sydwpx-gateway:9998 -rl --include="nginx.conf" ../conf/`  
./nginx -s reload  
ID=`ps -ef | grep mariadb10.3 | grep -v grep | awk '{print $2}'`  
echo $ID  
for id in $ID  
do  
    kill -9 $id  
    echo "kill $id"  
done

current_day=$(date +"%Y-%m-%d")  
rm -rf /dianda/sydwIncrement/*  
cd /mariadb10.3/mariadb-10.3.37-linux-x86_64/bin  
zcat /winshare/sydwpx/$1/sydwpx.gz | ./mbstream -x -C /dianda/sydwIncrement/  
./mariabackup --prepare --target-dir /dianda/sydwDatabase/ --incremental-dir /dianda/sydwIncrement/

cd /mariadb10.3/mariadb-10.3.37-linux-x86_64/bin  
nohup ./mysqld_safe --defaults-file=/mariadb10.3/mariadb-10.3.37-linux-x86_64/my.cnf --user=root &

cd /usr/nginx/sbin  
sed -i s/sydwpx-gateway:9999/sydwpx-gateway:9998/g `grep sydwpx-gateway:9999 -rl --include="nginx.conf" ../conf/`  
./nginx -s reload