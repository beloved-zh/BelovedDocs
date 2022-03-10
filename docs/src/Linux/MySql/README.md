#  

# 卸载mysql

## 1、检查是否安装mysql

```bash
rpm -qa|grep -i mysql
```

显示之前安装了：

```bash
MySQL-client-5.5.25a-1.rhel5

MySQL-server-5.5.25a-1.rhel5
```

停止mysql服务、删除之前安装的mysql

删除命令：`rpm -e –nodeps 包名`

```bash
rpm -ev MySQL-client-5.5.25a-1.rhel5
rpm -ev MySQL-server-5.5.25a-1.rhel5 
```

## 2、查找mysql目录

>  查找mysql的目录，并且删除mysql的文件和库（现在很多都是使用编译的mysql安装包进行安装的，所以查找文件是必须的

```bash
find / -name mysql
```

查找结果如下（根据个人实际情况）：

```bash
/etc/selinux/targeted/active/modules/100/mysql
/usr/share/mysql
/usr/lib64/mysql
```

删除对应的mysql目录

```bash
rm -rf /etc/selinux/targeted/active/modules/100/mysql
rm -rf /usr/share/mysql
rm -rf /usr/lib64/mysql
```

> **注意：/etc/my.cnf不会删除，需要进行手工删除**

```bash
rm -rf /etc/my.cnf
```

## 3、检查是否卸载成功

**再次查找机器是否安装mysql（注意！！！检查文件情况是必须的 注意！！！）**

检查安装情况

```bash
rpm -qa|grep -i mysql
```

检查mysql文件情况

```bash
find / -name mysql 
```

**无结果说明卸载（删除）彻底。**
