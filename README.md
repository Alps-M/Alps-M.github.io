# Alps-M.github.io
# 环境准备
根据自己服务器情况准备，以下是我的配置情况
注：系统全部采用的最小化安装

| 服务 | 系统 | 配置 | 私网IP |
| --- | --- | --- | --- |
| server/agent | Redhat7/Centos7 | 4U+8G+200G | 172.25.100.206 |
| mysql | Redhat7/Centos7 | 4U+8G+200G | 172.25.100.207 |
| proxy | Redhat7/Centos7 | 4U+8G+200G | 172.25.100.208 |

此方案拓扑如下
![](https://cdn.nlark.com/yuque/0/2023/jpeg/21399732/1689759365163-2f01a775-ec3d-46b3-a80a-dacab417b397.jpeg)
虚拟机的安装不再展示，若需要会单独出教程，下边直接看部署过程

# server端安装配置
## 关闭selinux
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```
## 配置阿里源
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```
## 配置zabbix源
```bash
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm  
sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo 
```
## 清除yum缓存
```bash
yum clean all
```
## 配置release仓库
```bash
yum install centos-release-scl -y
```

## 安装zabbix server端的相关组件
```basic
yum install zabbix-server-mysql zabbix-agent -y
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689748620778-07b29dd0-6cb0-4224-8ef7-4d22635e3979.png#averageHue=%23232221&clientId=u5b78559f-32a3-4&from=paste&height=654&id=u8cbc38d8&originHeight=852&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=126455&status=done&style=none&taskId=u06dbfcb2-5239-46d9-a473-af60915157d&title=&width=1236.4799622656262)
## 修改启用zabbix仓库
```bash
vi /etc/yum.repos.d/zabbix.repo
修改之后，wq保存退出
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689748700652-e5a54a52-d2a2-43e7-8d1c-07bc40ffb059.png#averageHue=%23222120&clientId=u5b78559f-32a3-4&from=paste&height=654&id=ub915f44e&originHeight=852&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=71203&status=done&style=none&taskId=u7e3d71ad-8ae5-469c-bdbb-ccf9cf4d124&title=&width=1236.4799622656262)

## 安装web服务相关组件
```bash
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689748873171-de136be1-9719-4ba0-a0fd-5bbe94cd8d76.png#averageHue=%23262423&clientId=u5b78559f-32a3-4&from=paste&height=654&id=ud8458398&originHeight=852&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=127860&status=done&style=none&taskId=ufb7a6725-c8d8-4844-a3e0-537890b505b&title=&width=1236.4799622656262)
## 修改config相关配置
**第一次安装主要修改数据库的信息，生产环境优化，也是修改此配置文件**
```bash
vi /etc/zabbix/zabbix_server.conf
egrep -v "^#|^$" /etc/zabbix/zabbix_server.conf
```
主要修改以下几个位置，数据库的IP和密码
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689749165356-60835130-0f40-4df2-b62d-2d173b458e3b.png#averageHue=%23282221&clientId=u5b78559f-32a3-4&from=paste&height=233&id=uef59c821&originHeight=304&originWidth=756&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=30937&status=done&style=none&taskId=u196afa2b-6e0c-48cb-878f-9db5906941b&title=&width=580.6079822812505)
## 修改PHP时区
```bash
vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
```
> ; php_value[date.timezone] = Europe/Riga

修改为
> php_value[date.timezone] = Asia/Shanghai

## 重启服务端的所有相关服务
```bash
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
```
此时重启会遇到以下报错
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689749533888-eba1857d-9ab0-4b86-8d2b-f5e2a992503d.png#averageHue=%232d2a26&clientId=u5b78559f-32a3-4&from=paste&height=124&id=ufc3b7551&originHeight=162&originWidth=1582&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=31530&status=done&style=none&taskId=u77b38664-4b06-456e-84ae-57771a9e937&title=&width=1214.975962921876)
查看日志，会发现由于server端口未在防火墙放通，导致服务起不来，由于安全考虑防火墙并不关闭，新手练习可以将防火墙关闭
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689749571695-cf0b25ae-c975-4684-b7d9-80ad774df9e4.png#averageHue=%232b2422&clientId=u5b78559f-32a3-4&from=paste&height=248&id=uac5a979c&originHeight=323&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=53750&status=done&style=none&taskId=u64aaf1f1-7ca7-47c2-ac2e-2624a67496e&title=&width=1236.4799622656262)
## 配置防火墙放行对应端口，并重启
```bash
firewall-cmd --add-port=10051/tcp --permanent && firewall-cmd --reload
firewall-cmd --add-port=80/tcp --permanent && firewall-cmd --reload
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689750315478-dbb58785-2eff-4db5-836b-80e714221ab2.png#averageHue=%23242120&clientId=u5b78559f-32a3-4&from=paste&height=187&id=u2f2a78b8&originHeight=243&originWidth=1220&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=39492&status=done&style=none&taskId=u36501db1-6c51-4be8-be34-ed0125c84d0&title=&width=936.9599714062509)
## 将服务加入开机自启
```bash
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```
# MySQL安装配置
## 配置阿里源
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```
## 安装相关工具
```bash
yum install wget -y
```
## 安装MySQL的rpm包
```bash
wget http://repo.mysql.com/yum/mysql-8.0-community/el/7/x86_64/mysql80-community-release-el7-2.noarch.rpm
yum install mysql80-community-release-el7-2.noarch.rpm -y
```
## 启用MySQL源
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689751016940-9d703ba4-4618-4226-8ef0-bee00f0d7c9f.png#averageHue=%23232221&clientId=u5b78559f-32a3-4&from=paste&height=654&id=u594794c9&originHeight=852&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=81921&status=done&style=none&taskId=uac73881c-5a71-4584-8971-e27ef7a0d76&title=&width=1236.4799622656262)
## 安装MySQL
```bash
yum install mysql-community-server -y 
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689751313845-cd1bc610-64d5-48a3-94b7-7321c6d8074c.png#averageHue=%23252422&clientId=u5b78559f-32a3-4&from=paste&height=621&id=u38db4549&originHeight=808&originWidth=1610&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=118428&status=done&style=none&taskId=u6731bedf-9cb6-4ffb-be31-3d7a2f874a5&title=&width=1236.4799622656262)
## 数据库的初始化及配置
```bash
systemctl start mysqld &&systemctl enable mysqld
```
### 查看MySQL初始密码
```bash
grep 'temporary password' /var/log/mysqld.log 
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689751439935-f57cbaca-8576-4d64-8956-0adbd116b085.png#averageHue=%232c2523&clientId=u5b78559f-32a3-4&from=paste&height=54&id=u4e70fdaf&originHeight=70&originWidth=1169&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=13791&status=done&style=none&taskId=u9961969b-fc1c-4225-b345-c77ac5a11db&title=&width=897.7919726015633)
### 修改密码
```bash
mysql -u root -p
SET PASSWORD = PASSWORD('Zabbix@123'); 
grant all privileges on *.* to 'root'@'%' identified by 'Zabbix@123';
flush privileges; 
```
### 创建数据库
```bash
create database zabbix character set utf8 collate utf8_bin; 
grant all privileges on zabbix.* to zabbix@'%' identified by 'Zabbix@123';
flush privileges;  
```
### 导入zabbix表
在server上查找create.sql.gz的位置，并传到MySQL上
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689752280071-2c77eca2-10c3-4145-9b08-954c3295f6a4.png#averageHue=%2322211f&clientId=u5b78559f-32a3-4&from=paste&height=210&id=u3e42cf2d&originHeight=273&originWidth=1567&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=49199&status=done&style=none&taskId=u2e24e1aa-9524-438e-861f-797c5fdc7cb&title=&width=1203.4559632734386)
```bash
zcat create.sql.gz | mysql -uzabbix -p zabbix
```
```bash
systemctl daemon-reload 
```
### 配置防火墙
放行MySQL的端口，此处不做修改，默认3306
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689751654905-5587de71-2122-4cc5-94b2-2bf1d091d2a5.png#averageHue=%23201f1e&clientId=u5b78559f-32a3-4&from=paste&height=83&id=u350cc5c3&originHeight=108&originWidth=727&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=10653&status=done&style=none&taskId=ua5358855-b38e-4415-9cf3-43007ff6843&title=&width=558.335982960938)

# web配置
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689753119637-321513e2-accd-4fa3-a27e-764e97a8c161.png#averageHue=%23f1f5f5&clientId=u5b78559f-32a3-4&from=paste&height=738&id=ucb1cc67f&originHeight=961&originWidth=1898&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=55523&status=done&style=none&taskId=ua71c004b-1ef9-4bb9-864f-52b4599532f&title=&width=1457.6639555156264)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689753124953-663c8e8c-23a7-4f5c-8070-caababf21503.png#averageHue=%23e7f1ef&clientId=u5b78559f-32a3-4&from=paste&height=701&id=ud281c2de&originHeight=913&originWidth=1898&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=55431&status=done&style=none&taskId=ub3d62a7c-ec31-4b79-996c-1d6a120bfc6&title=&width=1457.6639555156264)
此处填写数据库的相关信息
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689753149416-8737b4d0-1e8d-4652-8def-a4e09756f85a.png#averageHue=%23fbf7d6&clientId=u5b78559f-32a3-4&from=paste&height=594&id=u33dcd713&originHeight=774&originWidth=1261&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=45795&status=done&style=none&taskId=ue051ff03-7558-4f62-b3dc-c71220d7896&title=&width=968.4479704453134)
配置完成之后就如下界面，初始账号Admin密码zabbix
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689757704985-c33bc56b-ecac-439a-af5c-81e88f1ac015.png#averageHue=%23e8ecef&clientId=u5b78559f-32a3-4&from=paste&height=654&id=u723e18f9&originHeight=851&originWidth=1898&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=33980&status=done&style=none&taskId=u2b7fd605-4e70-4698-9642-06ce6dcd70e&title=&width=1457.6639555156264)
### 修改前端字体，解决中文乱码问题
将字体文件上传至 /usr/share/zabbix/accets/fonts/，可直接覆盖原字体文件，然后重新刷新web界面即可
# proxy部署
proxy和server安装步骤几乎一致，不做具体解释
## 关闭selinux
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```
## 配置阿里源
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```
## 配置zabbix源
```bash
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm  
sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo 
```
## 清除yum缓存
```bash
yum clean all
```
## 配置release仓库
```bash
yum install centos-release-scl -y
```

## 安装zabbix proxy端的相关组件
```bash
yum -y install zabbix-proxy-mysql zabbix-get 
```
## 安装数据库
```bash
wget http://repo.mysql.com/yum/mysql-8.0-community/el/7/x86_64/mysql80-community-release-el7-2.noarch.rpm
vi /etc/yum.repos.d/mysql-community.repo
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689761494806-f6dfcce3-0311-43f1-9e42-a01307750cce.png#averageHue=%23242322&clientId=ubf051f38-3b30-4&from=paste&height=538&id=ud8890da3&originHeight=701&originWidth=1256&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=64664&status=done&style=none&taskId=u3a925585-2bb7-49be-88e4-d6ff308677f&title=&width=964.6079705625009)
```bash
yum install mysql-community-server -y
```
## 配置MySQL
```bash
systemctl start mysqld
grep 'temporary password' /var/log/mysqld.log
SET PASSWORD = PASSWORD('Autewifi@123'); 
```

```bash
grant all privileges on *.* to 'root'@'%' identified by 'Zabbix@123';   
create database zabbix_proxy character set utf8 collate utf8_bin;
grant all privileges on zabbix_proxy.* to zabbix@"localhost" identified by "Zabbix@123";
flush privileges;
exit
```
```bash
rpm -ql zabbix-proxy-mysql //查找sql文件
zcat /usr/share/doc/zabbix-proxy-mysql-5.0.36/schema.sql.gz | mysql -uzabbix -p  //导入sql
```
## 修改proxy的配置
```bash
vi /etc/zabbix/zabbix_proxy.conf
grep "^[a-Z]" /etc/zabbix/zabbix_proxy.conf
```
主要修改server的IP和端口，proxy名字，还有数据库的信息
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689762338057-0564378f-8cf4-4d27-96ce-dea4f53d10b9.png#averageHue=%23292221&clientId=ubf051f38-3b30-4&from=paste&height=276&id=u4e1706d6&originHeight=359&originWidth=638&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=34657&status=done&style=none&taskId=uae218955-f514-44e1-b857-8d0eb5921a7&title=&width=489.98398504687543)
重启proxy服务之后，查看服务状态
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689762555159-72f9ef33-48cf-4691-9589-0d52f9b2e428.png#averageHue=%232d2925&clientId=ubf051f38-3b30-4&from=paste&height=406&id=u8982992f&originHeight=529&originWidth=1214&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=105519&status=done&style=none&taskId=u35bc558a-97fc-4de1-b7f0-4075d1b1f60&title=&width=932.3519715468759)
# 添加proxy
填写proxy的信息
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689762699252-a49a5c9a-ee47-41d6-b75a-7f86fde5383a.png#averageHue=%23fefefe&clientId=u16dec013-555b-4&from=paste&height=368&id=ub6ae7ac9&originHeight=479&originWidth=950&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=21690&status=done&style=none&taskId=u1a819bed-d181-4817-8807-5dca81be63c&title=&width=729.5999777343757)
添加成功的状态
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21399732/1689762744453-c2b5623c-ae0f-4354-8740-de0c92a563f9.png#averageHue=%23e8f4ef&clientId=u16dec013-555b-4&from=paste&height=234&id=u6bab6dba&originHeight=719&originWidth=1894&originalType=binary&ratio=1.3020833730697632&rotation=0&showTitle=false&size=60299&status=done&style=none&taskId=u652afda0-f234-471c-b7aa-333a3a0606a&title=&width=616)
