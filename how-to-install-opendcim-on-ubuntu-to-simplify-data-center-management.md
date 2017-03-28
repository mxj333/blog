# 如何在Ubuntu上安装openDCIM来简化数据中心管理

如果您正在寻找开源数据中心基础架构管理工具，请查看[openDCIM](http://opendcim.org/)。考虑到您获得的软件成本（免费），这是一个您一定要尝试的基于网络的系统。

使用openDCIM，您可以：

* 提供数据中心的资产跟踪
* 支持多间客房
* 管理空间，电源和冷却
* 管理联系人的商业目录
* 跟踪容错
* 每个机柜的重力计算中心
* 管理设备的模板
* 跟踪每个机柜和每个开关设备内的电缆连接
* 存档设备送到救助/处置
* 与智能电源板和UPS设备集成

如果您现有的Ubuntu服务器方便（也可以安装在桌面上），您可以通过一些努力获得openDCIM并运行。安装不是你会做的最简单的;然而，我已经解决了一些挑战，并介绍了在Ubuntu上安装这个强大的系统的简单步骤。

## 安装openDCIM

如果您还没有在Ubuntu机器上安装LAMP堆栈，请执行以下简单步骤。

1. 打开一个终端窗口。
2. 发出命令
   _sudo apt-get install lamp-server^_

3. 键入您的_sudo_密码，然后按Enter键。
4. 允许安装完成。

在安装过程中，系统会提示您设置一个mysql管理员密码。请确保照顾并记住该密码。

一旦您准备好LAMP堆栈，还有一些必须安装的其他依赖项。返回到您的终端窗口并发出以下命令：

_sudo apt-get install php-snmp snmp-mibs-downloader php-curl php-gettext graphviz_

允许该命令完成，您可以继续。

## 下载软件

下一步是下载最新版本的openDCIM - 在这篇文章中，该版本是4.3。回到你的终端窗口并发出命令_wget_[_http://www.opendcim.org/packages/openDCIM-4.3.tar.gz_](http://www.opendcim.org/packages/openDCIM-4.3.tar.gz)。这将把文件下载到您当前的工作目录中。使用命令 _tar xvzf openDCIM-4.3.tar.gz_解压缩文件。接下来，使用命令_sudo mv openDCIM-4.3 dcim_重命名新创建的文件夹。最后，使用命令_sudo mv dcim / var / www /_移动该文件夹。

您还需要使用以下命令更改权限：

_sudo chgrp -R www-data /var/www/dcim/pictures /var/www/dcim/drawings_

## 创建数据库

接下来我们创建数据库。使用命令_mysql -u root -p_打开MySQL提示符，然后在提示时输入在LAMP安装过程中创建的密码。发出以下命令：

* _create database dcim;_
* _grant all on dcim.\* to 'dcim'@'localhost' identified by 'dcim';_
* _flush privileges;_
* _exit;_

## 配置数据库

由于我们创建了数据库dcim并使用密码dcim，内置的数据库配置文件将无需编辑即可使用;我们所要做的就是使用命令重命名模板：

_sudo cp /var/www/dcim/db.inc.php-dist /var/www/dcim/db.inc.php_

## 配置Apache

必须为Apache配置虚拟主机。我们_将为_openDCIM使用_default-ssl.conf_配置。转到您的终端窗口并切换到_/etc/apache/sites-available_目录并打开_default-ssl.conf_文件。对于该文件，我们将首先将_DocumentRoot_变量更改为 _/var/www/dcim_，然后在该行下面添加以下内容：

&lt;Directory "/var/www/dcim"&gt; 

Options All 

AllowOverride All

AuthType Basic

AuthName dcim

AuthUserFile /var/www/dcim/.htpassword 

Require all granted

&lt;/Directory&gt;

保存并关闭该文件。

## 设置用户访问

我们还必须确保openDCIM将其限制为用户访问。我们将在htaccess的帮助下这样做。创建文件/var/www/dcim/.htaccess，内容如下：

_AuthType Basic_

_AuthName "openDCIM"_

_AuthUserFile /var/www/opendcim.password_

_Require valid-user_



保存该文件并发出命令：

_sudo htpasswd -cb /var/www/opendcim.password dcim dcim_

## 启用Apache模块和站点

最后一件事（在将浏览器指向安装之前）是启用必需的Apache模块并启用到_default-ssl_站点。您可能会发现其中一些已经启用。发出以下命令：

* _sudo a2enmod ssl_
* _sudo a2enmod rewrite_
* _sudo a2ensite default-ssl_
* _sudo service apache2 restart_



您已准备好安装openDCIM。

## 安装openDCIM

您应该将浏览器指向https://localhost/install.php（您可以使用openDCIM服务器的IP地址替换localhost）。系统将提示您输入目录凭据，这与htaccess一样。为此，用户名将为dcim，密码将为dcim。此时，应通过飞行前检查清单，直接转到部门创建页面（**图A\)**。

图A：![](/assets/opendcima.jpg)

###### 创建部门和数据中心

最后一步是删除_/var/www/dcim/install.php_文件。然后将您的浏览器指向https：// localhost（或服务器的IP地址），您将被带到主要的openDCIM站点（**图B**）。

**图B：**

![](/assets/opendcimb.jpg)

###### openDCIM主页

## 准备服务

此时，openDCIM可以为您服务。你最有可能从一个免费的软件中找到比你期望的更多的东西。花费时间加快各种功能的速度，您将可以随时更好地跟踪各种数据中心，项目，基础设施等等，这些都来自于一个集中的位置。





