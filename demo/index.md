# 家用Linux服务器通用服务搭建教程


 **&emsp;&emsp;一个家用服务器常用服务的详细搭建教程，包括但不限于<font color=red>samba文件共享、FTP服务器、FRP内网穿透、AdGuard Home广告过滤、qBittorrent Web下载器</font>等服务的搭建教程；文中所用文本编辑器为vim，不熟悉相关用法可用其他编辑器替代或自行百度相关命令及用法，本文不再赘述；以下内容在<font color=red>Ubuntu 20.04 LTS Server</font>系统中完全适用，其他Linux系统由于包管理工具与软件安装路径的差异，相关配置可能略有出入。**

* * *
本文是视频[<svg role="img" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" data-svg-src="/lib/simple-icons/icons/bilibili.min.svg" class="icon"><path d="M17.813 4.653h.854c1.51.054 2.769.578 3.773 1.574 1.004.995 1.524 2.249 1.56 3.76v7.36c-.036 1.51-.556 2.769-1.56 3.773s-2.262 1.524-3.773 1.56H5.333c-1.51-.036-2.769-.556-3.773-1.56S.036 18.858.0 17.347v-7.36c.036-1.511.556-2.765 1.56-3.76 1.004-.996 2.262-1.52 3.773-1.574h.774l-1.174-1.12a1.234 1.234.0 01-.373-.906c0-.356.124-.658.373-.907l.027-.027c.267-.249.573-.373.92-.373s.653.124.92.373L9.653 4.44c.071.071.134.142.187.213h4.267a.836.836.0 01.16-.213l2.853-2.747c.267-.249.573-.373.92-.373s.662.151.929.4.391.551.391.907c0 .355-.124.657-.373.906zM5.333 7.24c-.746.018-1.373.276-1.88.773-.506.498-.769 1.13-.786 1.894v7.52c.017.764.28 1.395.786 1.893.507.498 1.134.756 1.88.773h13.334c.746-.017 1.373-.275 1.88-.773.506-.498.769-1.129.786-1.893v-7.52c-.017-.765-.28-1.396-.786-1.894-.507-.497-1.134-.755-1.88-.773zM8 11.107c.373.0.684.124.933.373.25.249.383.569.4.96v1.173c-.017.391-.15.711-.4.96-.249.25-.56.374-.933.374s-.684-.125-.933-.374c-.25-.249-.383-.569-.4-.96V12.44c0-.373.129-.689.386-.947.258-.257.574-.386.947-.386zm8 0c.373.0.684.124.933.373.25.249.383.569.4.96v1.173c-.017.391-.15.711-.4.96-.249.25-.56.374-.933.374s-.684-.125-.933-.374c-.25-.249-.383-.569-.4-.96V12.44c.017-.391.15-.711.4-.96.249-.249.56-.373.933-.373z"></path></svg>Linux服务器应用搭建教程](https://www.bilibili.com/video/BV1HS4y1S7P9)的文字版教程，目前只更新了一部分视频中的配置，后续会逐步完善并增加更多内容🤡
* * *

## **<font color=green>1. SAMBA文件共享</font>**

Samba可以实现window访问Linux的资源，实现两个系统间的数据交互。samba服务程序已经成为在Linux系统和Windows系统之间共享文件的最佳选择，但是基于SMB2.0协议的客户端数据传输速度可能不及FTP。

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

FTP (File Transfer Protocol) 可说是最古老的协议之一了，主要是用来进行文件的传输，尤其是大型文件的传输使用 FTP 更是方便。

### 2.1 安装vsftpd

```bash
   sudo apt install vsftpd
```

### 2.2 修改配置文件

以下是vsftpd的配置，基本配置条目已存在，只要取消注释或更改设置即可(配置文件中包含英文说明，某些主要的配置我会加上中文注释)：

点击展开代码块查看完整的示例配置👇

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

## **<font color=green>3. FRP内网穿透</font>**

目前国内家用宽带基本全是大内网，如果要从公网访问内网的设备，内网穿透技术就显得尤为重要；作者使用的内网穿透服务是[Sakura FRP](https://www.natfrp.com/)，普通用户可以免费创建两条隧道，每月有5G流量，并且每日签到会送1-4G的流量，10Mb带宽基本满足个人使用了。

|   |普通用户|青铜 VIP|白银 VIP |
|-----|-------|--------|---------|
|正常限速|10 Mibps|24 Mibps|36 Mibps|
|隧道数量|2 条|2 条 + 8 条额外隧道|2 条 + 8 条额外隧道|
|月基础流量|5 GiB|5 GiB|5 GiB|
|每月获赠流量包|❌|66 GiB|88 GiB|
|解锁 VIP 节点|❌|✔️|✔️|
|购买额外隧道|❌|✔️|✔️|
|解锁高级节点|❌|❌|✔️|

### 3.1 注册并下载客户端软件

访问[Sakura FRP](https://www.natfrp.com/)官网注册账号并登录,花一块钱完成实名认证后再找到服务中的软件下载
![软件下载](/images/FRP1.png)

按照系统版本选择对应的软件版本下载或复制下载链接
![下载](/images/FRP2.png)

使用`wget`命令下载到服务器并保存为/usr/local/bin/frpc备用

```bash
➜  ~ wget https://getfrp.sh/d/frpc_linux_amd64  
--2022-12-29 23:20:58--  https://getfrp.sh/d/frpc_linux_amd64  
正在解析主机 getfrp.sh (getfrp.sh)... 2606:4700:3037::ac43:95bc, 2606:4700:3037::6815:1dc6, 172.67.149.188, ...  
正在连接 getfrp.sh (getfrp.sh)|2606:4700:3037::ac43:95bc|:443... 已连接。  
已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily  
位置：https://nyat-static.globalslb.net/natfrp/client/0.45.0-sakura-1/frpc_linux_amd64 [跟随至新的 URL]
--2022-12-29 23:20:59--  https://nyat-static.globalslb.net/natfrp/client/0.45.0-sakura-1/frpc_linux_amd64  
正在解析主机 nyat-static.globalslb.net (nyat-static.globalslb.net)... 240e:914:6:d:37b3:771e:6811:fffe, 153.35.50.206
正在连接 nyat-static.globalslb.net (nyat-static.globalslb.net)|240e:914:6:d:37b3:771e:6811:fffe|:443... 已连接。  
已发出 HTTP 请求，正在等待回应... 200 OK  
长度： 4556856 (4.3M) [application/octet-stream]  
正在保存至: ‘frpc_linux_amd64’  

frpc_linux_amd64              100%[=================================================>]   4.35M  12.3MB/s    用时 0.4s  

2022-12-29 23:21:00 (12.3 MB/s) - 已保存 ‘frpc_linux_amd64’ [4556856/4556856])

➜  ~ sudo mv ./frpc_linux_amd64 /usr/local/bin/frpc  
➜  ~
```

### 3.2 创建隧道

创建隧道前需要在 **用户-实名认证** 中完成实名认证

在 **服务-隧道列表** 中创建隧道  
![创建隧道](/images/FRP3.png)

根据需求类型选择合适的节点，以枣庄多线2为例（使用SSH）  
![配置隧道](/images/FRP4.png)

隧道类型选择TCP隧道,本地IP使用默认值，端口选择22后创建隧道  
![配置隧道](/images/FRP5.png)

隧道创建完成后会在隧道列表中显示隧道的<font color=red>ID</font>和外部访问<font color=red>端口</font>,**这两个参数后面会用到！**  
![隧道信息](/images/FRP6.png)

### 3.3 配置本地服务

在 **用户-用户信息** 页面中查看<font color=red>访问密钥</font>备用，<font color=red>**注意：访问密钥是使用隧道的重要参数不要泄漏给任何人！！！**</font>  
![访问密钥](/images/FRP7.png)

使用命令`sudo vim /etc/systemd/system/frpc.service`在`/etc/systemd/system/`目录下编辑一个`frpc.service`服务文件

文件内容如下：

```bash
[Unit]
Description=SakuraFrp Service
After=network.target

[Service]
Type=idle
User=nobody
Restart=on-failure
RestartSec=60s
#/usr/local/bin/frpc为下载的FRP文件绝对路径
#同一节点支持启动多个隧道 参数为 -f <访问密钥>:<隧道ID>,<隧道ID>，...
ExecStart=/usr/local/bin/frpc -f 8**************d:1****6,1****0,1*****3,1*****2


[Install]
WantedBy=multi-user.target
```

然后依次执行以下命令

```bash
#更新服务配置
sudo systemctl daemon-reload
#启动FRP服务
sudo systemctl start frpc
#设置服务开机启动
sudo systemctl enable frpc
#查看服务状态
sudo systemctl status frpc
```

查看服务状态出现类似以下提示(部分信息已作脱敏处理)则为隧道启动成功

```bash
● frpc.service - SakuraFrp Service  
     Loaded: loaded (/lib/systemd/system/frpc.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-12-25 22:07:20 CST; 4 days ago
   Main PID: 710 (frpc)
      Tasks: 10 (limit: 4561)
     Memory: 18.2M
        CPU: 2min 32.520s
     CGroup: /system.slice/frpc.service
             └─710 /usr/local/bin/frpc -f 8*************b:1*****6,1*****0,1*****3

12月 29 18:02:05 ubuntu frpc[710]: TCP 类型隧道启动成功
12月 29 18:02:05 ubuntu frpc[710]: 使用 [********.natfrp.cloud:2***5] 来连接到你的隧道
12月 29 18:02:05 ubuntu frpc[710]: 或使用 IP 地址连接（不推荐）：[***.***.***.***:2***5]
12月 29 18:02:05 ubuntu frpc[710]: 2022/12/29 18:02:05 [I] [44/*****/****] [9******5.***] start proxy success
12月 29 18:02:05 ubuntu frpc[710]: TCP 类型隧道启动成功
12月 29 18:02:05 ubuntu frpc[710]: 使用 [********.natfrp.cloud:2***5] 来连接到你的隧道
12月 29 18:02:05 ubuntu frpc[710]: 或使用 IP 地址连接（不推荐）：[***.***.***.***:2***5]
12月 29 18:02:05 ubuntu frpc[710]: 2022/12/29 18:02:05 [I] [44/*****/****] [9******5.***] start proxy success
12月 29 18:02:05 ubuntu frpc[710]: 2022/12/29 18:02:05 [I] [44/*****/****] 限速已更新: 10 Mibit/s
12月 29 23:58:30 ubuntu systemd[1]: /lib/systemd/system/frpc.service:7: Special user nobody configured, this is not saf>
lines 1-20/20 (END)
```

提示信息中会有类似以下使用隧道的提示字样，内容为隧道的外网访问地址和端口
>使用 [**********.natfrp.cloud:3*****6] 来连接到你的隧道

### 3.4 测试隧道连接

使用上面提示的链接与端口尝试通过SSH连接Ubuntu服务器，出现输入密钥密码提示，证明隧道启动成功！  
![SSH](/images/FRP8.png)

## **<font color=green>4. AdGuard Home</font>**

家用网络通常是光猫拨号，默认使用ISP(互联网服务提供商)提供的DNS(域名服务器)服务器，因为DNS污染严重导致某些网站(如GitHub)经常被解析到错误的IP地址，导致网站无法正常访问。

AdGuard Home 是 AdGuard 开源的一个私人 DNS 服务端，只需在网关部署，即可实现全局域网的广告拦截与隐私反追踪。在 DNS 解析的过程中，匹配规则库内的 Url 进行拦截，同时在客户端中，还可以通过自定义过滤规则实现网页 DOM 的拦截。

>AdGuard Home 是一款全网广告拦截与反跟踪软件。在您将其安装完毕后，它将保护您所有家用设备，同时您不再需要安装任何客户端软件。随着物联网与连接设备的兴起，掌控您自己的整个网络环境变得越来越重要。  
</br>
—— AdGuard Home

### 4.1 安装AdGuard Home

点击前往[AdGuard Homed的GitHub主页](https://github.com/AdguardTeam/AdGuardHome)，README.md中提供了全自动安装脚本
![Automated install](/images/ADGH1.png)

直接执行其提供的安装命令即可(需要root权限)

```bash
sudo curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

若执行时提示服务器拒绝连接可编辑配置文件`/etc/systemd/resolved.conf`将DNS改为以下任意DNS地址后重新执行

>几个可用的DNS:  
cloudflare DNS: 1.0.0.1 , 1.1.1.1  
Google Public DNS: 8.8.8.8 , 8.8.4.4  
百度公共 DNS: 180.76.76.76  
DNSPod(腾讯) DNS: 119.29.29.29  
阿里云 DNS: 223.5.5.5 , 223.6.6.6  
OneDNS: 117.50.10.10  
DNS派 电信DNS: 101.226.4.6 , 移动DNS：218.30.118.6  
114 DNS: 114.114.114.114 , 114.114.115.115  

脚本会自动安装配置并启动服务，执行`sudo systemctl status AdGuardHome.service`命令可查看服务运行状态

```bash
➜  ~ sudo systemctl status AdGuardHome.service
● AdGuardHome.service - AdGuard Home: Network-level blocker
     Loaded: loaded (/etc/systemd/system/AdGuardHome.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-12-31 06:01:04 CST; 15h ago
   Main PID: 694 (AdGuardHome)
      Tasks: 12 (limit: 4561)
     Memory: 63.8M
        CPU: 4min 40.054s
     CGroup: /system.slice/AdGuardHome.service
             └─694 /home/tyler/AdGuardHome/AdGuardHome -s run

12月 31 06:01:04 ubuntu systemd[1]: Started AdGuard Home: Network-level blocker.
➜  ~
```

### 4.2 设置AdGuard Home并添加屏蔽规则

AdGuard Home Web界面默认在3000端口可通过Web页面修改端口，访问服务器IP:3000即可查看Web页面；如果Web页面无法访问，请检查防火墙是否已打开3000端口,Ubuntu防火墙开放3000端口命令为`sudo ufw allow 3000`。首次访问会引导用户配置登陆密码  
![首次访问](/images/ADGH2.png)

80是HTTP网站的默认端口，如果你的服务器有建站计划，建议这里不要把网页管理界面端口设置成80；
客户端访问DNS默认使用53端口，为了避免不必要的麻烦不要修改DNS服务器的监听端口！  
另外，作为DNS服务器必须使用静态IP，服务器IP变化将会导致域名解析失败。  
![首次访问](/images/ADGH3.png)

配置好用户名密码后即可访问管理页面，在 **设置-DNS设置** 中可以配置上游DNS  
![首次访问](/images/ADGH5.png)

在 **过滤器-DNS 拦截列表** 中可以配置 adblock 规则和 Hosts 语法  
![首次访问](/images/ADGH6.png)

以下为作者自用的几个规则：

>乘风广告：https://gitee.com/xinggsf/Adblock-Rule/raw/master/rule.txt  
乘风视频：https://gitee.com/xinggsf/Adblock-Rule/raw/master/mv.txt  
广告终结者：http://sub.adtchrome.com/adt-chinalist-easylist360.txt  
EasyList China：https://easylist-downloads.adblockplus.org/easylistchina.txt  
澳门皇家赌场专用屏蔽规则：https://raw.githubusercontent.com/Goooler/1024_hosts/master/hosts  
CHN: AdRules DNS List：https://adguardteam.github.io/HostlistsRegistry/assets/filter_29.txt  
CHN: anti-AD：https://adguardteam.github.io/HostlistsRegistry/assets/filter_21.txt  
知乎广告：https://cdn.jsdelivr.net/gh/zsakvo/AdGuard-Custom-Rule@master/rule/zhihu-strict.txt  
ChinaList+EasyList(淇)：http://sub.adtchrome.com/adt-chinalist-easylist.txt  
EasyPrivacy：https://easylist-downloads.adblockplus.org/easyprivacy.txt  

### 4.3 设置默认DNS服务器

直接设置路由器的DHCP服务器(以小米路由器为例，因为我用的就是小米😼)，登录路由器后台，在 **常用设置-局域网设置-DHCP服务** 中将DNS1设置为服务器的地址，若服务器7*24小时运行可不设置DNS2，保存设置后连接到此路由器的设备都会使用AdGuard Home作为DNS服务器。  
![设置DNS](/images/ADGH7.png)

### 4.4 测试DNS服务器

不使用AdGuard Home服务器的设备：  

```bash
#使用nslookup命令查询github素材服务器raw.githubusercontent.com的IP地址，结果为0.0.0.0
➜  ~ nslookup raw.githubusercontent.com
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   raw.githubusercontent.com
Address: 0.0.0.0
Name:   raw.githubusercontent.com
Address: ::

#cm.ipinyou.com是某视频广告的域名，使用nslookup命令查询可以得到IP
➜  ~ nslookup cm.ipinyou.com
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   cm.ipinyou.com
Address: 152.136.199.109
Name:   cm.ipinyou.com
Address: 152.136.179.124
Name:   cm.ipinyou.com
Address: 152.136.187.208
```

使用AdGuard Home服务器的设备：  

```bash
➜  ~ nslookup raw.githubusercontent.com
服务器:  UnKnown
Address:  192.168.31.52

非权威应答:
名称:    raw.githubusercontent.com
Addresses:  2606:50c0:8003::154
          2606:50c0:8000::154
          2606:50c0:8002::154
          185.199.111.133
          185.199.110.133
          185.199.108.133
          185.199.109.133

➜  ~ nslookup cm.ipinyou.com
服务器:  UnKnown
Address:  192.168.31.52

*** UnKnown 找不到 cm.ipinyou.com: Query refused
```

经过对比不难发现AdGuard Home是可以规避DNS污染并实现广告过滤的。
