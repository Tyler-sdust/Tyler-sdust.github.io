# 家用Linux服务器通用服务搭建教程


#### &emsp;&emsp;一个家用服务器常用服务的详细搭建教程，包括但不限于<font color=red>samba文件共享、FTP服务器、FRP内网穿透、AdGuard Home广告过滤、qBittorrent Web下载器</font>等服务的搭建教程；文中所用文本编辑器为vim，不熟悉相关用法可用其他编辑器替代或自行百度相关命令及用法，本文不再赘述；以下内容在<font color=red>Ubuntu 20.04 LTS Server</font>系统中完全适用，其他Linux系统由于包管理工具与软件安装路径的差异，相关配置可能略有出入。

* * *
目前只更新了samba的配置，后续会逐步完善内容(主要是Markdown语法我还没研究明白🤡)
* * *

## **<font color=green>1. SAMBA文件共享</font>**

### 1.1 安装samba

   ```bash
   sudo apt install samba
   ```

### 1.2 修改配置文件

   ```bash
   sudo vim /etc/samba/smb.conf
   ```

在文件末尾添加以下配置：

   ```bash
   [share]  
   #名称随便起  
   comment = samba home directory  
   #共享目录的绝对路径  
   path = /home/share/smb/disk  
   public = yes  
   browseable = yes  
   read only = no  
   guest ok = no  
   #选择一个本地用户作为登录用户名，注意此处以share为例，后面会用到  
   valid users = share  
   #文件权限控制  
   create mask = 0755  
   #目录权限控制  
   directory mask = 0755  
   available = yes  
   #关闭异步读取  
   aio read size = 0  
   ```

### 1.3 检查配置文件  

使用 `testparm` 命令检查添加的配置是否正确，出现以下提示则证明配置可用

> ➜  ~ testparm  
  Load smb config files from /etc/samba/smb.conf  
  Loaded services file OK.  
  Weak crypto is allowed  
  </br>
  Server role: ROLE_STANDALONE  
  </br>
  Press enter to see a dump of your service definitions

### 1.4 启动samba服务并设置开机自启

   ```bash
   #启动服务
   sudo systemctl start smbd   
   #设置开机自启
   sudo systemctl enable smbd
   #查看服务状态
   sudo systemctl status smbd
   ```

### 1.5 为登录用户配置登陆密码并重启服务

   ```bash
   #配置访问密码
   sudo smbpasswd -a <user> #<user>为前文配置文件中配置的用户名share，命令执行后会要求设置密码
   
   #注意配置密码时默认不显示输入：
   ➜  ~ sudo smbpasswd -a share  
   New SMB password:  
   Retype new SMB password:
   
   #密码配置完成后重启服务
   sudo systemctl restart smbd
   ```

### 1.6 测试samba能否正常使用

   ![SMB1](/images/SMB1.png)  

   输入服务器主机名或IP地址+目录名称，然后输入用户名密码  

   ![SMB2](/images/SMB2.png)  

   ![SMB3](/images/SMB3.png)  

## **<font color=green>2. FTP服务器</font>**

### 2.1 安装vsftpd

```bash
   sudo apt install vsftpd
```

### 2.2 修改配置文件

以下是vsftpd的配置，基本配置条目已存在，只要取消注释或更改设置即可(配置文件中包含英文说明，某些主要的配置我会加上中文说明)：

```bash
# Example config file /etc/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
#
# Run standalone?  vsftpd can run either from an inetd or as a standalone
# daemon started from an initscript.
#以独立模式运行vsftpd
listen=YES
#
# This directive enables listening on IPv6 sockets. By default, listening
# on the IPv6 "any" address (::) will accept connections from both IPv6
# and IPv4 clients. It is not necessary to listen on *both* IPv4 and IPv6
# sockets. If you want that (perhaps because you want to listen on specific
# addresses) then you must run two copies of vsftpd with two configuration
# files.
#listen_ipv6=NO
#
# Allow anonymous FTP? (Disabled by default).
#禁止匿名登录
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
#允许本地用户登录
local_enable=YES
#
# Uncomment this to enable any form of FTP write command.
#允许写入，如果只想从FTP服务器读取或下载则可以改为NO
write_enable=YES
#
# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
#local_umask=022
#
# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
#anon_upload_enable=YES
#
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
#anon_mkdir_write_enable=YES
#
# Activate directory messages - messages given to remote users when they
# go into a certain directory.
#欢迎语设置，会检查共享目录下是否有.message这个档案，如果有，则会回显此档案的内容
dirmessage_enable=YES
#
# If enabled, vsftpd will display directory listings with the time
# in  your  local  time  zone.  The default is to display GMT. The
# times returned by the MDTM FTP command are also affected by this
# option.
#使用本地时间显示文件日期
use_localtime=YES
#
# Activate logging of uploads/downloads.
#启用上传/下载日志记录
xferlog_enable=YES
#
# Make sure PORT transfer connections originate from port 20 (ftp-data).
#通过20端口链接
connect_from_port_20=YES
#
# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
#chown_uploads=YES
#chown_username=whoever
#
# You may override where the log file goes if you like. The default is shown
# below.
#xferlog_file=/var/log/vsftpd.log
#
# If you want, you can have your log file in standard ftpd xferlog format.
# Note that the default log file location is /var/log/xferlog in this case.
#xferlog_std_format=YES
#
# You may change the default value for timing out an idle session.
#idle_session_timeout=600
#
# You may change the default value for timing out a data connection.
#data_connection_timeout=120
#
# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
#nopriv_user=ftpsecure
#
# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
#async_abor_enable=YES
#
# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
#ascii_upload_enable=YES
#ascii_download_enable=YES
#
# You may fully customise the login banner string:
#ftpd_banner=Welcome to blah FTP service.
#
# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd.banned_emails
#
# You may restrict local users to their home directories.  See the FAQ for
# the possible risks in this before using chroot_local_user or
# chroot_list_enable below.
#允许用户切换到上级目录
chroot_local_user=YES
#
# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
# (Warning! chroot'ing can be very dangerous. If using chroot, make sure that
# the user does not have write access to the top level directory within the
# chroot)
#启用指定的用户列表文件
chroot_list_enable=YES
# (default follows)
#用户列表文件，搞清楚规则之前请不要向该文件中添加用户！！！
chroot_list_file=/etc/vsftpd.chroot_list
#
#用户登录后的默认目录，不设置则为登录用户的home目录
local_root=/home/share/smb/disk
#
# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
#ls_recurse_enable=YES
#
# Customization
#
# Some of vsftpd's settings don't fit the filesystem layout by
# default.
#
# This option should be the name of a directory which is empty.  Also, the
# directory should not be writable by the ftp user. This directory is used
# as a secure chroot() jail at times vsftpd does not require filesystem
# access.
secure_chroot_dir=/var/run/vsftpd/empty
#
# This string is the name of the PAM service vsftpd will use.
pam_service_name=vsftpd
#
# This option specifies the location of the RSA certificate to use for SSL
# encrypted connections.
#SFTP相关配置，FTP不适用
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
allow_writeable_chroot=YES
#
# Uncomment this to indicate that vsftpd use a utf8 filesystem.
#使用utf-8编码
utf8_filesystem=YES
```

{{< admonition >}}  
**<font color=red>注意：如果write_enable已经设置为YES，为了保证服务器的安全性，请仔细阅读以下说明！！！</font>**  

chroot_local_user和chroot_list_enable两个配置项可以搭配使用，规则如下：  
</br>
1.当chroot_list_enable=YES，chroot_local_user=YES时，vsftpd.chroot_list文件中列出的用户，可以切换到其他目录；未在文件中列出的用户，不能切换到其他目录。  
</br>
2.当chroot_list_enable=YES，chroot_local_user=NO时，vsftpd.chroot_list文件中列出的用户，不能切换到其他目录；未在文件中列出的用户，可以切换到其他目录。  
</br>
3.当chroot_list_enable=NO，chroot_local_user=YES时，所有的用户均不能切换到其他目录。  
</br>
4.当chroot_list_enable=NO，chroot_local_user=NO时，所有的用户均可以切换到其他目录。
{{< /admonition >}}

### 2.3 重启vsftpd服务

```bash
sudo service vsftpd restart
```

### 2.4 测试配置是否正常

若配置正常但仍无法登录FTP并且已配置SSH登录，请检查`/etc/ssh/sshd_config`配置文件，将其中的SFTP服务配置全部注释掉

>#SFTP服务  
Subsystem sftp internal-sftp  
&emsp;Match Group share  
&emsp;&emsp;ChrootDirectory /home/share/sftp  
&emsp;&emsp;ForceCommand internal-sftp  
&emsp;&emsp;PasswordAuthentication yes  
&emsp;&emsp;PermitTunnel no  
&emsp;&emsp;X11Forwarding no  
&emsp;&emsp;AllowTcpForwarding no  
&emsp;&emsp;AllowAgentForwarding no

