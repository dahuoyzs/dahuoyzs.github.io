

## linux上安装YAPI项目 【软件搭建】



### YAPI安装 

#### 1.安装node

```shell
curl -O https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz	#下载
tar xf  node-v10.9.0-linux-x64.tar.xz									#解压
ln -s 【node解压包路径】/bin/npm   /usr/local/bin/ 						#快捷方式
ln -s 【node解压包路径】/bin/node   /usr/local/bin/
node -v
v10.9.0
```
<!--more-->
#### 2.安装mongodb

```shell
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz  #下载
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz                                 #解压
vim /etc/profile
export PATH=【mongodb解压包路径】/bin:$PATH				 			 	#添加环境变量
source /etc/profile
```

添加在MongoDB目录下创建一个配置文件

vim  mongodb.conf

```shell
dbpath=【DB数据存放目录】
logpath=【DB日志存放目录】/mongod.log
logappend=true
fork=true					#后台运行，命令行窗口关掉也不影响
```

```shell
#启动服务端
./mongod   -f   mongodb.conf
#启动客户端
./mongo
```

#### 3.安装Yapi

```shell
npm install -g yapi-cli --registry https://registry.npm.taobao.org		#安装yapi
```

**使用yapi启动服务**

```
yapi server    			#运行起来,请先确保MongoDB启动起来
#关闭
kill -9 pid
```

> 查看提示信息,填写部署信息,建议1.60版本(建议不要用太新的版本),部署完成后根据提示操作即可
> 由于提示中使用的node 启动的，窗口关闭后会停止yapi服务，所以建议使用pm2管理，yapi在后台运行

**使用PM2启动服务**

```shell
npm install -g pm2													 #pm2安装
pm2 start 【Yapi启动的目录】/my-yapi/vendors/server/app.js			    #启动
pm2 stop  【Yapi启动的目录】/my-yapi/vendors/server/app.js			 	#关闭
```

这样以后就可以很方便的用pm2就行启动关闭啦

> 注意：pm2启动的服务 kill -9 pid 是杀不掉的，所以还要用pm2 stop关闭yapi
>





### 二、数据丢失问题?

issues：[莫名数据库里的数据就没了](https://github.com/YMFE/yapi/issues/1693)

#### 原因：默认配置

mongoddb使用默认配置端口号，且没有设置账号和密码，被黑客恶意删库。

![](D:\desktop\文本\笔记\linux各种环境的搭建\img\log.png)

所有数据都是备份的。您必须支付0.015 BTC到16Vk6GrQYwzXvNxdBq7Zp6TGnwPEhpBWgB 48小时才能恢复。在48小时到期后，我们将泄露并公开您的所有数据。如果拒绝付款，我们将联系通用数据保护条例（GDPR），并通知他们您以开放形式存储用户数据，并且不安全。根据法律规定，您将面临严厉的罚款或逮捕，您的基地垃圾将从我们的服务器上丢弃！您可以在这里购买比特币，使用本指南购买https://localbitcoins.com不需要花费太多时间https://localbitcoins.com/guides/how-to-buy-bitcoins后，使用您的DB IP:getbase@cock.li在邮件中写信给我



#### 解决：修改端口,添加账号

参考链接:https://blog.csdn.net/kxzhaohuan/article/details/81713949

修数据库端口

vim  【mongodb目录】/conf/mongodb.conf

```shell
#末尾添加，即可覆盖20
port=【端口号】
```

添加账号密码权限

```shell
#以非安全模式启动mongod
./mongod -f 【mongodb目录】/conf/mongod.conf
use admin
db.createUser({user:"root",pwd:"123456",roles:[{role:"root",db:"admin"}]});
db.auth("root","123456")
use yapi
db.createUser({user:"root",pwd:"123456",roles:[{role:"readWrite",db:"yapi"}]})
db.auth("root","123456")
#退出 kill -9 [mongod Pid]
#重新启动服务
./mongod -f 【mongodb目录】/conf/mongod.conf -auth
#尝试使用客户端连接
./mongo --port 【端口号】 -u root -p 123456
```

Yapi配置

vim  【Yapi目录】/config.json

```
{
   "port": "7001",
   "adminAccount": "admin@admin.com",
   "db": {
      "servername":"127.0.0.1",
      "DATABASE":"yapi",
      "port":"【数据库端口】",
      "user":"【用户名】",
      "pass":"【密码】"
   },
   "mail": {
      "enable": false,
      "host": "smtp.163.com",
      "port": 465,
      "from": "***@163.com",
      "auth": {
         "user": "***@163.com",
         "pass": "*****"
      }
   }
}

```

此时mongoddb的启动方式为：

```
./mongod -f 【mongodb目录】/conf/mongod.conf -auth
```

及安全模式，重新启动yapi即可。





### 三、mongoddb扩展知识

主要程序位于解压包的/bin目录下

其中`mongod`为`Server`端程序

而`mongo`为`Client`端程序

#### 1.Server端启动方式

```shell
#普通方式启动
./mongod   
#带配置文件的启动方式
./mongod -f 【mongodb目录】/conf/mongod.conf
#需要验证权限的方式启动
./mongod -f 【mongodb目录】/conf/mongod.conf -auth 
```

基本参数：
-f                            指定配置文件
--port                     指定端口，默认是27017
--dbpath                数据目录路径
--logpath               日志文件路径
--logappend           日志append而不是overwrite
--fork                     以创建子进程的方式运行
--journal                日志提交间隔，默认100ms
--nojournal            关闭日志功能，2.0版本以上是默认开启的

 参考：http://www.mongodb.org/display/DOCS/File+Based+Configuration

#### 2.Client端的一些操作

```shell
#普通方式启动
./mongo
#查看当前库
db
#切换使用库
use admin
use test
#查看所有库
show dbs
#查看当前库有哪些表
show collections
> system.indexes
> system.users
#查看当前库的user表
show users
#根据当前库，查看当前库支持哪些操作
db.help();
#根据当前表，查看当前库支持哪些操作
db.users.help()
```

#### 3.授权

```shell
use admin
db.createUser({user:"root",pwd:"123456",roles:[{role:"root",db:"admin"}]});
db.auth("root","123456")
use yapi
db.createUser({user:"root",pwd:"123456",roles:[{role:"readWrite",db:"yapi"}]})
db.auth("root","123456")
```

#### 4.关闭

```shell
#【mongo中关闭】
use admin
#正常关闭
db.shutdownServer()
#强制关闭
db.shutdownServer({force : true}) 
#【直接杀死进程关闭】
#查看进程
ps -ef|grep mongod
#杀死进程
kill -9 [pid]
```

强制关闭Mongod，应对副本集中主从时间差超过10s时不允许关闭主库的情况
不要使用kill直接杀mongo进程的方式关闭数据节点，会造成数据损坏

#### 5.常见问题

错误提示:

```shell
2020-04-23T10:25:37.656+0800 E QUERY    [thread1] Error: listCollections failed: {
    "ok" : 0,
    "errmsg" : "not authorized on admin to execute command { listCollections: 1.0, filter: {}, $db: \"admin\" }",
    "code" : 13,
    "codeName" : "Unauthorized"
} :
```

##### **快速排查权限的问题**

权限类问题，如果是用户权限问题，不太好排查，可以关闭服务端.
重新启动服务端，不要带`-auth`参数，这样就可以取消检测权限了，
可以查看一下admin库中的user表状态和相关的权限。



**安全模式与非安全模式快速切换脚本**

将switch.sh脚本放在bin目录下可以方便切换mongod的启动方式

switch.sh

```

param=
if [ -n "$1" ];
  then param=" -auth" ;
fi
#echo $param
pid=`ps -ef |grep "mongod"|grep -v "grep" |awk '{print $2}'`
if [ -n "$pid" ]; then
  echo "kill -9 $pid" 
  kill -9 $pid
fi

cd 【mongod目录】/bin
./mongod -f 【mongod目录】/conf/mongod.conf $param
newPid=`ps -ef |grep "mongod"|grep -v "grep" |awk '{print $2}'`
#echo "start  $newPid"
```

脚本来切换mongod的模式

```shell
#非安全模式
sh switch
#安全模式
sh switch  auth
```



