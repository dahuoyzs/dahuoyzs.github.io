首先了解几个命令
mysql5.7数据库 开，关，重启命令 （安装完成后才能用）

```shell
service mysqld start
service mysqld stop
service mysqld restart
```
<!--more-->
准备工作
先安装依赖

```shell
yum install -y cmake make gcc gcc-c++ libaio ncurses ncurses-devel
yum install -y libaio
```

创建用户组和用户

```shell
groupadd mysql
useradd -r -g mysql mysql
```

通过文件上传工具把mysql.tar.gz上传到linux上
使用   tar -xzvf mysql.tar.gz 解压压缩包

```shell
cd /usr/local/
mkdir mysql
chown -R mysql:mysql mysql
chmod -R 777 /usr/local/mysql/
cd mysql
mkdir data
chown -R mysql:mysql data
chmod -R 777 /usr/local/mysql/data
```

回到刚才解压mysql的地方进入解压包内    mv * /usr/local/mysql
cd /usr/local/mysql

安装

bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data



修改配置 vi /etc/my.cnf文件  添加一下内容

```shell
[mysqld]
character_set_server=utf8service mysqld stop
init_connect='SET NAMES utf8'
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
lower_case_table_names = 1
log-error=/usr/local/mysql/mysqld.log
```

cd /usr/local/mysql
添加开机启动     cp support-files/mysql.server /etc/init.d/mysql
加入开机起动   chkconfig --add mysql
修改   vi /etc/init.d/mysql    


添加路径 在46行   

```shell
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```


启动mysql   service mysqld start 
如果出现错误 需要添加软连接  ln -s /usr/local/mysql/bin/mysql /usr/bin
登录修改密码 mysql -uroot -p 上面初始化时的密码
如果没密码在my.conf 内加skip-grant-tables保存重启数据库
再mysql -uroot -p就不要密码了直接回车


可能出现的错误
执行mysql.server start
报错：ERROR! The server quit without updating PID file
 (/usr/local/var/mysql/chenyuntekiMacBook-Air.local.pid)
解决:
chmod -R 777 /usr/local/mysql/
