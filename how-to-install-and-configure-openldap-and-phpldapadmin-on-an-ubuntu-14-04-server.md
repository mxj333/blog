# 如何在Ubuntu 14.04服务器上安装和配置OpenLDAP和phpLDAPadmin

### 介绍 {#introduction}

**LDAP**或轻量级目录访问协议是用于以集中的分层文件和目录结构来管理和访问相关信息的协议。

在某些方面，它的操作与关系数据库类似，但这并不适用于所有内容。层次结构是数据相关的主要区别。它可以用于存储任何种类的信息，它通常用作集中式认证系统的一个组件。

在本指南中，我们将讨论如何在Ubuntu 14.04服务器上安装和配置OpenLDAP服务器。然后，我们将安装并保护一个phpLDAPadmin界面，以提供一个简单的Web界面。

## 安装LDAP和Helper实用程序 {#install-ldap-and-helper-utilities}

在我们开始之前，我们必须安装必要的软件。幸运的是，这些软件包都可以在Ubuntu的默认存储库中使用。

这是我们第一次`apt`在本课程中使用，所以我们将刷新本地包索引。之后我们可以安装我们想要的包：

```
sudo apt-get update

sudo apt-get install slapd ldap-utils
```

在安装过程中，将要求您选择并确认LDAP的管理员密码。你实际上可以把任何东西放在这里，因为你有机会在短时间内改变它。

### 重新配置slapd以选择更好的设置 {#reconfigure-slapd-to-select-better-settings}

即使该软件包刚刚安装，我们将要进行重新配置Ubuntu安装的默认配置。

这样做的原因是，虽然包有问了很多重要的配置问题的能力，这些都是在安装过程中跳过。我们可以访问所有的提示，尽管告诉我们的系统重新配置包：

```
sudo dpkg-reconfigure slapd

```

在完成这一过程时，会有不少问题被提出来。现在我们来看看这些：

* 省略OpenLDAP服务器配置？
  **没有**
* * 此选项将确定目录路径的基本结构。
    阅读消息以了解这将如何实现。
  * 这实际上是一个相当开放的选择。
    即使您没有拥有实际的域名，您也可以选择所需的任何“域名”值。
    但是，如果您有服务器的域名，那么使用它可能是明智的。
  * 对于本指南，我们
    将为我们的配置
    选择
    **test.com**
    。
* 机构名称？
  * 再次，这完全取决于你的喜好。
  * 对于本指南，我们将使用
    **示例**
    作为我们组织的名称。
* 管理员密码？
  * 正如我在安装部分提到的，这是您真正的选择管理员密码的机会。
    您在此处选择的任何内容都将覆盖您以前使用的密码。
* 数据库后端？
  **HDB**
* 清除slapd时删除数据库？
  **没有**
* 移动旧数据库？
  **是**
* 允许LDAPv2协议？
  **没有**

此时，您的LDAP应以相当合理的方式进行配置。

## 安装phpLDAPadmin以使用Web界面管理LDAP {#install-phpldapadmin-to-manage-ldap-with-a-web-interface}

尽管很可能通过命令行管理LDAP，但大多数用户会发现使用Web界面更为容易。我们将安装phpLDAPadmin，它提供了这个功能，以帮助消除学习LDAP工具的一些摩擦。

Ubuntu存储库包含phpLDAPadmin软件包。您可以通过键入以下内容进行安装：

```
sudo apt-get install phpldapadmin

```

这应该安装管理界面，启用所需的Apache虚拟主机文件，并重新加载Apache。

Web服务器现在配置为为您的应用程序提供服务，但我们将进行一些其他更改。我们需要配置phpLDAPadmin以使用我们为LDAP配置的域架构，我们也将进行一些调整，以保护我们的配置一点。

### 配置phpLDAPadmin {#configure-phpldapadmin}

现在安装了软件包，我们需要配置一些东西，以便它可以连接到在OpenLDAP配置阶段创建的LDAP目录结构。

首先在文本编辑器中打开具有root权限的主配置文件：

```
sudo nano /etc/phpldapadmin/config.php

```

在此文件中，我们需要添加我们为LDAP服务器设置的配置详细信息。首先查找主机参数并将其设置为服务器的域名或公共IP地址。此参数应反映您计划访问Web界面的方式：

```
$ servers-
>
 setValue（'server'，'host'，' 
server_domain_name_or_IP
 '）;

```

接下来，您需要配置为LDAP服务器选择的域名。记住，在我们的例子中我们选择了`test.com`。我们需要将每个域组件（所有不是点）替换为规范的值，将其转换为LDAP语法`dc`。

所有这一切意味着`test.com`，我们不会写作，而是写一些类似的东西`dc=test,dc=com`。我们应该找到设置服务器基础参数的参数，并使用我们刚才讨论的格式来引用我们决定的域：

```
$ servers-
>
 setValue（'server'，'base'，array（'dc = 
test
，dc = 
com
 '））;

```

我们需要在我们的登录bind\_id参数中调整同样的事情。该`cn`参数已设置为“admin”。这是对的。我们只需要重新调整`dc`部分，就像我们上面所说的那样：

```
$ servers-
>
 setValue（'login'，'bind_id'，'cn = admin，dc = 
test
，dc = 
com
 '）;

```

我们需要调整的最后一件事是控制警告消息的可见性的设置。默认情况下，phpLDAPadmin将在其Web界面中引入不少恼人的警告消息，关于不影响功能的模板文件。

我们可以通过搜索参数来隐藏这些，取消注释`hide_template_warning`包含它的行，并将其设置为“true”：

```
$ config-
>
 custom-
>
 appearance ['hide_template_warning'] = 
true
 ;

```

这是我们需要调整的最后一件事。完成后可以保存并关闭文件。

## 创建SSL证书 {#create-an-ssl-certificate}

我们希望通过SSL确保我们与LDAP服务器的连接，以便外部各方不能拦截我们的通信。

由于管理界面与本地网络上的LDAP服务器本身进行通信，因此我们不需要为该连接使用SSL。当我们连接时，我们只需要保护我们浏览器的外部连接。

为此，我们只需要设置我们的服务器可以使用的自签名SSL证书。这不会帮助我们验证服务器的身份，但它将允许我们加密我们的消息。

默认情况下，应在系统上安装OpenSSL软件包。首先，我们应该创建一个目录来保存我们的证书和密钥：

```
sudo mkdir /etc/apache2/ssl

```

接下来，我们可以通过键入以下操作在一个动作中创建密钥和证书：

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt

```

您将不得不回答一些问题，以使该实用程序正确填写证书中的字段。唯一真正重要的是提示`Common Name (e.g. server FQDN or YOUR name)`。输入您的服务器的域名或IP地址。

完成后，您的证书和密钥将被写入`/etc/apache2/ssl`目录。

## 创建密码验证文件 {#create-a-password-authentication-file}

我们也想密码保护我们的phpLDAPadmin位置。即使phpLDAPadmin有密码认证，这将提供额外的保护。

我们需要的实用程序包含在Apache实用程序包中。输入以下内容：

```
sudo apt-get install apache2-utils

```

现在您可以使用该实用程序，您可以创建一个密码文件，其中包含您选择的用户名和关联的散列密码。

我们将把它保存在`/etc/apache2`目录中。创建文件并指定您要使用的用户名：

```
sudo htpasswd -c / etc / apache2 / htpasswd 
demo_user
```

现在，我们已经准备好修改Apache，以利用我们的安全升级。

## 安全的Apache {#secure-apache}

我们应该做的第一件事是启用Apache中的SSL模块。我们可以输入以下内容：

```
sudo a2enmod ssl

```

这将使模块能够使用它。我们仍然需要配置Apache来利用这一点。

目前，Apache正在读取一个名为`000-default.conf`普通的，未加密的HTTP连接的文件。我们需要告诉它将我们的phpLDAPadmin接口的请求重定向到HTTPS接口，以便连接被加密。

当我们重定向流量以使用我们的SSL证书时，我们还将实施密码文件来验证用户。在修改事情的同时，我们还将更改phpLDAPadmin界面本身的位置，以最大限度地减少目标攻击。

### 修改phpLDAPadmin Apache配置 {#modify-the-phpldapadmin-apache-configuration}

我们首先要做的是修改设置为服务我们的phpLDAPadmin文件的别名。

在文本编辑器中使用root权限打开文件：

```
sudo nano /etc/phpldapadmin/apache.conf

```

这是我们需要决定要访问我们的界面的URL位置的地方。默认是`/phpldapadmin`，但是我们想改变它，以减少机器人和恶意方随机登录尝试。

对于本指南，我们将使用该位置`/superldap`，但您应该选择自己的价值。

我们需要修改指定的行`Alias`。这应该在一个`IfModule mod_alias.c`块。当你完成它应该看起来像这样：

```
<
IfModule mod_alias.c
>

    别名
/ superldap
 / usr / share / phpldapadmin / htdocs
<
/ IfModule
>
```

当你完成，安全并关闭文件。

### 配置HTTP虚拟主机 {#configure-the-http-virtual-host}

接下来，我们需要修改我们当前的虚拟主机文件。在编辑器中使用root权限打开它：

```
sudo nano /etc/apache2/sites-enabled/000-default.conf

```

在里面，你会看到一个如此简单的配置文件：

```
<
VirtualHost *：80
>

    ServerAdmin webmaster @ localhost

    DocumentRoot / var / www / html

    ErrorLog $ {APACHE_LOG_DIR} /error.log

    CustomLog $ {APACHE_LOG_DIR} /access.log组合
<
/ VirtualHost
>
```

我们要添加有关我们的域名或IP地址的信息来定义我们的服务器名称，并且我们要设置重定向，将所有HTTP请求指向HTTPS接口。这将匹配我们在上一节中配置的别名。

我们讨论的变化最终会像这样。用你自己的值修改红色的项目：

```
<
VirtualHost *：80
>

    ServerAdmin webmaster @ 
server_domain_or_IP

    DocumentRoot / var / www / html
ServerName server_domain_or_IP 
重定向永久/超级页面https：// server_domain_or_IP / superldap

    ErrorLog $ {APACHE_LOG_DIR} /error.log

    CustomLog $ {APACHE_LOG_DIR} /access.log组合
<
/ VirtualHost
>
```

完成后保存并关闭文件。

### 配置HTTPS虚拟主机文件 {#configure-the-https-virtual-host-file}

Apache包括一个默认的SSL虚拟主机文件。但是，默认情况下不启用。

我们可以通过键入来启用它：

```
sudo a2ensite default-ssl.conf

```

这将把文件从`sites-available`目录链接到`sites-enabled`目录中。我们可以通过键入以下内容来编辑此文件：

```
sudo nano /etc/apache2/sites-enabled/default-ssl.conf

```

这个文件比最后一个有点多，所以我们只讨论我们必须做的更改。下面的所有更改都应该在文件中的虚拟主机块中。

首先，将`ServerName`值重新设置为您的服务器的域名或IP地址，并更改该`ServerAdmin`指令：

```
ServerAdmin webmaster @ 
server_domain_or_IP 
ServerName server_domain_or_IP
```

接下来，我们需要设置SSL证书指令来指向我们创建的密钥和证书。这些指令应该已经存在于您的文件中，因此只需修改它们指向的文件：

```
SSLCertificateFile 
/etc/apache2/ssl/apache.crt
 
SSLCertificateKeyFile 
/etc/apache2/ssl/apache.key
```

我们需要做的最后一件事就是设置一个位置块，为整个phpLDAPadmin安装实现我们的密码保护。

我们通过引用我们提供phpLDAPadmin的位置并使用我们生成的文件来设置身份验证来做到这一点。我们将要求任何人尝试访问此内容以作为有效用户进行身份验证：

```
<
地点/ superldap
>
进行AuthType基本
AuthName指令“受限制的文件” 
的AuthUserFile的/ etc / apache2的/ htpasswd的
需要有效的用户
<
/位置
>
```

完成后保存并关闭文件。

重新启动Apache以实现我们所做的所有更改：

```
sudo service apache2 restart

```

我们现在可以转到实际的界面。

## 登录到phpLDAPadmin Web界面 {#log-into-the-phpldapadmin-web-interface}

我们已经对phpLDAPadmin软件进行了我们需要的配置更改。我们现在可以开始使用它了。

我们可以通过访问我们服务器的域名或公共IP地址，然后访问我们配置的别名访问Web界面。在我们的情况下，这是`/superldap`：

```
http：// 
server_domain_name_or_IP
 / 
superldap
```

您第一次访问时，您可能会看到有关网站的SSL证书的警告：

![](https://assets.digitalocean.com/articles/ldap_install_1404/ssl_warning.png "phpLDAPadmin SSL警告")

警告只是在这里让您知道浏览器无法识别签署您的证书的证书颁发机构。既然我们签署了_自己的_证书，这是预期的，而不是一个问题。

点击“继续进行”按钮或浏览器提供的任何类似选项。

接下来，您将看到为Apache配置的密码提示：

![](https://assets.digitalocean.com/articles/ldap_install_1404/password_prompt.png "phpLDAPadmin密码提示")

填写使用该`htpasswd`命令创建的帐户凭据。您将看到主要的phpLDAPadmin着陆页：

![](https://assets.digitalocean.com/articles/ldap_install_1404/main_landing.png "phpLDAPadmin着陆页")

点击您可以在页面左侧看到的“登录”链接。

![](https://assets.digitalocean.com/articles/ldap_install_1404/login_page.png "phpLDAPadmin登录页面")

您将被带到登录提示。登录名“DN”就像您将要使用的用户名。它包含“cn”下的帐户名称，您为服务器选择的域名分为“dc”部分，如上所述。

如果正确配置phpLDAPadmin，应该为管理员帐户预先填充正确的值。在我们的例子中，这看起来像这样：

```
cn=admin,dc=test,dc=com

```

对于密码，请输入在LDAP配置过程中配置的管理员密码。

您将被带到主界面：

![](https://assets.digitalocean.com/articles/ldap_install_1404/main_page.png "phpLDAPadmin主页")

## 添加组织单位，组和用户 {#add-organizational-units-groups-and-users}

此时，您将登录到phpLDAPadmin界面。您可以添加用户，组织单位，组和关系。

LDAP对于如何构建数据和目录层次结构是灵活的。您基本上可以创建任何类型的结构，并为它们的交互创建规则。

由于Ubuntu 14.04上的这个过程与Ubuntu 12.04相同，您可以按照Ubuntu 12.04的[LDAP安装文章](https://digitalocean.com/community/articles/how-to-install-and-configure-a-basic-ldap-server-on-an-ubuntu-12-04-vps#AddOrganizationalUnits,Groups,andUsers)的“添加组织单位，组和用户”一节中的[步骤进行操作](https://digitalocean.com/community/articles/how-to-install-and-configure-a-basic-ldap-server-on-an-ubuntu-12-04-vps#AddOrganizationalUnits,Groups,andUsers)。

在这个安装中的步骤将是完全一样的，所以请遵循一些实践，使用界面，并了解如何构建您的单元。

## 结论 {#conclusion}

您现在应该在Ubuntu 14.04服务器上安装并配置OpenLDAP。您还安装并配置了一个Web界面，以通过phpLDAPadmin程序来管理您的结构。您通过强制SSL和密码保护整个应用程序，为应用程序配置了一些基本的安全性。

我们设置的系统非常灵活，您应该可以根据需要设计自己的组织架构并管理资源组。在下一个指南中，我们将讨论如何配置联网计算机以将此LDAP服务器用于系统身份验证。

