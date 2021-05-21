# Cetnos安装mysql 

## 准备： 

+ centos系统 
+ [在MySQL官网中下载YUM源rpm安装包](https://dev.mysql.com/downloads/repo/yum)：
`https://dev.mysql.com/downloads/repo/yum/  `

## 清理阶段： 

+ 卸载mysql： 

    `yum remove mysql-community-server`

+ 清理mysql其他相关包： 

    `rpm -qa | grep mysql`

    `yum remove mysql-xxx-xxx-`

+ 清理mysql其他相关文件/文件夹： 

    `find / -name mysql`

    `rm -rf xxxxxxx`

+ 卸载mariadb文件：

    `rpm -qa | grep mariadb`

    `rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64`


## 开始安装： 

+ 安装mysql下载仓库： 

    `rpm -ivh mysql80-community-release-el7-11.noarch.rpm`

+ 启用对应仓库（2个）：
    
    `vi /etc/yum.repos.d/mysql-community.repo`

    `vi /etc/yum.repos.d/mysql-community-source.repo`

    <img src='../../../img/mysql.repo.png'>

+ 更新yum源

    `yum clean all && yum makecache`

+ 安装mysql

    `yum install mysql-community-server`

    > 如果报错找不到 `Error: Unable to find a match: mysql-community-server` 就， 
    先执行`yum module disable mysql` ， 在执行`yum install mysql-community-server`


+ 查看mysql状态

   `systemctl status mysqld.service` 

+ 启动/停止/重启mysql服务

    `systemctl start mysqld.service` 

    `systemctl stop mysqld.service` 

    `systemctl restart mysqld.service` 


+ 设置msyql开机自启动

    `systemctl enable mysqld.service`


+ 获取初始密码： 

    `cat /var/log/mysqld.log | grep password`

+ 登录： 

    `mysql -uroot -p`

+ 修改密码： 

    > mysql8.0版本必须符合长度（默认是8位），且必须含有数字，小写或大写字母，特殊字符。

    `mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Aa123@';`

+ 开启远程访问mysql ： 

    `mysql> grant all privileges on *.* to 'root'@'%' identified by 'Aa123@' with grant option;`

    `mysql> flush privileges;`
