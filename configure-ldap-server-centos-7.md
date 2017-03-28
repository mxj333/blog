# 在CENTOS 7上配置LDAP服务器

在本文中，我们将学习如何使用CentOS 7安装和配置打开的LDAP服务器。LDAP提供了服务器和客户端用于彼此通信的标准语言。它是国际组织为X.500标准化使用的目录访问协议的较轻版本。它可以支持百万个具有标准硬件配置的条目。

### 配置LDAP的先决条件

服务器平台 - CentOS 7机器。

我们服务器的IP地址为192.168.33.100。

服务器的主机名将为ldap.jt.com。

### LDAP服务器的安装步骤

使用以下步骤配置LDAP服务器

#### 更新系统并定义主机名

更新系统

```
# yum update
```

#### 2.使用yum安装所需的openldap包

```
＃yum install *openldap*
```

定义服务器的主机名

```
# nmtui
```

![](/assets/1-1.png)

运行命令启动ldap服务

```
＃systemctl start slapd
```

启用服务以在启动时运行

```
＃systemctl enable slapd
```

确保允许端口389

```
＃netstat -ntl
```

输出![](/assets/9-1.png)

生成ldap密码

```
＃slappasswd
```

生成密码并复制

```
New password:
Re-enter new password:
{SSHA}WMRcQ+/NZ+T0Lml78e4bnU/88UGybpFP
```

#### 3.编辑打开LDAP配置文件

打开目录

```
＃cd /etc/openldap/slapd.d/cn=config


我们需要在此目录中编辑以下两个文件。
```

```
olcDatabase={1}monitor.ldif
olcDatabase={2}hdb.ldif
```

首先编辑olc数据库文件

```
＃vim olcDatabase={2}hdb.ldif
```

在下面提供您的域凭据，替换您的设置。寻找 **olcRootPW：**  语法和粘贴ldap密码值。

```
##自动生成的文件 - 不要编辑！使用ldapmodify
##========================================

# AUTO-GENERATED FILE - DO NOT EDIT!! Use ldapmodify.
# CRC32 57ead3ba1111
dn: olcDatabase={2}hdb
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: {2}hdb
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=jt,dc=com  ###Define doamin controller 
olcRootDN: cn=Manager,dc=jt,dc=com ###Define Distingush name for domain controller
olcDbIndex: objectClass eq,pres
olcDbIndex: ou,cn,mail,surname,givenname eq,pres,sub
structuralObjectClass: olcHdbConfig
entryUUID: 548fbb00-e8fb-1035-8d76-b38427b6950e
creatorsName: cn=config
createTimestamp: 20160728103939Z
entryCSN: 20160728103939.380569Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20160728103939Z
olcRootPW: {SSHA}r+op2tBDH5joBt14e0lVCnuUR0KnEWRt ###Paste generated passwod generated in above step
```

编辑 olcDatabase\={1}monitor.ldif 文件。

```
＃vim olcDatabase\=\{1\}monitor.ldif
```

定义您的域值

```
dn: olcDatabase={1}monitor
objectClass: olcDatabaseConfig
olcDatabase: {1}monitor
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=extern
al,cn=auth" read by dn.base="cn=Manager,dc=jt,dc=com" read by * none
structuralObjectClass: olcDatabaseConfig
entryUUID: 548facc8-e8fb-1035-8d75-b38427b6950e
creatorsName: cn=config
createTimestamp: 20160728103939Z
entryCSN: 20160728103939.380201Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20160728103939Z
```

测试您的设置并忽略生成的任何校验和错误，将生成成功测试的消息。

![](/assets/2-3.png)4.配置打开的LDAP数据库

将样本数据库文件复制到**/var/lib/ldap**

```
＃cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
```

更改文件的所有权

```
＃chown -R ldap:ldap /var/lib/ldap/
```

#### 5.添加LDAP方案

打开目录 /etc/openldap/slapd.d/cn\=config/

```
＃cd /etc/openldap/slapd.d/cn\=config/
```

添加方案

```
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

#### 6.使用migrationtools创建基础对象

要创建基础对象，我们需要迁移工具，让我们先安装它们

```
＃yum install migrationtools
```

将目录更改为 /usr/share/migrationtools/

```
＃cd  /usr/share/migrationtools/
```

看一看

```
＃ls
```

![](/assets/3-1.png)编辑migrate\_common.ph

```
＃vim migrate_common.ph
```

定义域凭据，编辑行号**71**和**74**。

```
70 # Default DNS domain
71 $DEFAULT_MAIL_DOMAIN = "jt.com";
72
73 # Default base
74 $DEFAULT_BASE = "dc=jt,dc=com";
```

保存并退出，创建base.ldif文件，将该文件保存到 /root/ 目录下，使用** migration\_base.pl** 生成.ldif文件。

```
＃./migrate_base.pl /root/base.ldif
```

grep /etc/passwd 到 /root 目录，在/root 中将文件另存为 'passwd'

```
＃grep ":10[0-9][0-9]" /etc/passwd > /root/passwd
```

grep /etc/group as /root/user

```
＃grep ":10[0-9][0-9]" /etc/group > /root/group
```

将/root/passwd文件迁移到 users.ldif

```
＃./migrate_passwd.pl /root/passwd /root/users.ldif
```

**users.ldif**文件将看起来像

![](/assets/6-2.png)迁移/root/group文件到 /root/groups.ldif

＃ ./migrate\_group.pl /root/group /root/groups.ldif

**Groups.ldif**将看起来像

![](/assets/7-2.png)

在这个阶段，我们在/ root /文件夹中有三个.ldif文件，即base.ldif，users.ldif和groups.ldif，看看。

＃ls / root![](/assets/4-2.png)7.将系统用户导入LDAP数据库

使用以下命令将所有三个.ldif文件导入LDAP，并输入密码，然后按Enter键

```
# ldapadd -x -W -D "cn=Manager,dc=oids,dc=cn" -f /root/base.ldif
# ldapadd -x -W -D "cn=Manager,dc=oids,dc=cn" -f /root/users.ldif
# ldapadd -x -W -D "cn=Manager,dc=oids,dc=cn" -f /root/groups.ldif
```

输出![](/assets/8-1.png)8.运行配置测试

用户广告组已迁移到LDAP，让我们检查事情是否正确完成，

```
＃ldapsearch -x cn=user2 -b dc=oids,dc=cn
```

输出

```
# extended LDIF
#
# LDAPv3
# base <dc=jt,dc=com> with scope subtree
# filter: cn=user2
# requesting: ALL
#
# user2, Group, jt.com
dn: cn=user2,ou=Group,dc=oids,dc=cn
objectClass: posixGroup
objectClass: top
cn: user2
userPassword:: e2NyeXB0fXg=
gidNumber: 1002
# search result
search: 2
result: 0 Success
# numResponses: 2
# numEntries: 1
```

Success消息表示服务器配置成功。

### 结论

我们创建了一个功能齐全的LDAP服务，可以在各种目录服务中使用。

原文链接：[http://www.jointux.com/configure-ldap-server-centos-7/](http://www.jointux.com/configure-ldap-server-centos-7/)

