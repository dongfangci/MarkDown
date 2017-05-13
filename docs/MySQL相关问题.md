#MySQL相关问题#
##1.安装配置问题##
使用MySQL5.7.18进行安装
在Windows系统中如果选择解压缩版本（.zip），需要进行如下配置：
###1.配置环境变量Path
我的电脑->属性->高级->环境变量选择PATH,在其后面添加: 你的mysql bin文件夹的路径
PATH=.......;C:\Program Files\MySQL\MySQL Server 5.6\bin (注意是追加,不是覆盖)
###2.修改配置文件
配置完环境变量之后先别忙着启动mysql，我们还需要修改一下配置文件（如果没有配置，之后启动的时候就会出现图中的错误哦！:错误2 系统找不到文件），自己建立一个my.ini文件，
![](http://i.imgur.com/8T64WnH.png)
在其中修改或添加配置（如图）： 
```
[mysqld]

#绑定IPv4和3306端口
bind-address = 0.0.0.0
port = 3306

# 设置mysql的安装目录
basedir=C:\Program Files\mysql-5.7.18-winx64

# 设置mysql数据库的数据的存放目录
datadir=D:\SqlData

# 允许最大连接数
max_connections=200
```
###3.安装MySQL服务###
以管理员身份运行cmd（一定要用管理员身份运行，不然权限不够），
输入：cd C:\Program Files\MySQL\MySQL Server 5.6\bin 进入mysql的bin文件夹(不管有没有配置过环境变量，也要进入bin文件夹，否则之后启动服务仍然会报错误2)输入
```
mysqld -install
```
(如果不用管理员身份运行，将会因为权限不够而出现错误：Install/Remove of the Service Denied!) 
安装成功
![](http://i.imgur.com/iqiSwoI.png)
###4.启动mysql服务###
安装成功后就要启动服务了，继续在cmd中输入:
```
net start mysql
```
（如图）,服务启动成功！
此时很多人会出现错误，请看注意：
注意：这个时候经常会出现错误2和错误1067。
如果出现“错误2 系统找不到文件”，检查一下是否修改过配置文件或者是否进入在bin目录下操作，如果配置文件修改正确并且进入了bin文件夹，需要先删除mysql（输入 mysqld -remove）再重新安装（输入 mysqld -install）；
如果出现错误1067，那就是配置文件修改错误，确认一下配置文件是否正确。
###注意：出现MySQL服务无法启动，服务没有报告任何错误的解决方法###
####1.首先查看数据库存储目录下的err文件，查看出错原因
![](http://i.imgur.com/OCGiVWn.png)
####2.err文件报错TCP端口被占用
之前碰到3306端口被其他服务占用导致MySQL无法启动，关闭其他使用3306的端口，删除MySQL后重新安装，重新启动服务。
####3.err文件报错详细信息如下
```
...
MySQL: Table 'mysql.plugin' doesn't exist
2015-11-02T01:40:40.370946Z 0 [ERROR] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
2015-11-02T01:40:40.370946Z 0 [Note] InnoDB: Buffer pool(s) load completed at 151102  9:40:40
2015-11-02T01:40:40.371946Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2015-11-02T01:40:40.372946Z 0 [Warning] Failed to set up SSL because of the following SSL library error: SSL context is not usable without certificate and private key
2015-11-02T01:40:40.373946Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2015-11-02T01:40:40.374946Z 0 [Note] IPv6 is available.
2015-11-02T01:40:40.375946Z 0 [Note]   - '::' resolves to '::';
2015-11-02T01:40:40.375946Z 0 [Note] Server socket created on IP: '::'.
2015-11-02T01:40:40.377946Z 0 [Warning] Failed to open optimizer cost constant tables

2015-11-02T01:40:40.377946Z 0 [ERROR] Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't exist
2015-11-02T01:40:40.379946Z 0 [ERROR] Aborting
```
总之就是找不到一些数据表
检查data文件目录中是否有 mysql目录，其中是否有 plugin、user 表。如果有则检查权限。
![](http://i.imgur.com/VH63qJL.png)
如果没有此目录，则表示没有初始化MySQL，需要在安装完MySQL后执行初始化命令。因此卸载MySQL重新安装，之后进行初始化data目录，命令如下
```
mysqld  --initialize
```
再启动mysql服务，如果命令报错说明data文件夹不为空，需要先删除在执行。
####如果依然报如上错误需要先删除data目录(或移动到其他地方)，再执行
```
mysqld --initialize
```
####官方文档说了mysqld --initialize-insecure自动生成无密码的root用户，mysqld --initialize自动生成带随机密码的root用户。data文件夹不为空是不能执行这个命令的。可以先删除data目录下的所有文件或者移走，随机密码可在err文件中找到。
   





