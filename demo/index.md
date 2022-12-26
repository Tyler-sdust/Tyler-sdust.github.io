# 家用Linux服务器通用服务搭建教程


#### &emsp;&emsp;一个家用服务器常用服务的详细搭建教程，包括但不限于<font color=red>samba文件共享、FTP服务器、FRP内网穿透、AdGuard Home广告过滤、qBittorrent Web下载器</font>等服务的搭建教程；文中所用文本编辑器为vim，不熟悉相关用法可用其他编辑器替代或自行百度相关命令及用法，本文不再赘述；

#### &emsp;&emsp;以下内容在<font color=red>Ubuntu 20.04 LTS Server</font>系统中完全适用，其他Linux系统由于包管理工具与软件安装路径的差异，相关配置可能略有出入。

* * *

- **<font color=green>SAMBA文件共享</font>**
1. 安装samba
   
   ```bash
   sudo apt install samba
   ```

2. 修改配置文件
   
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

3. 检查配置文件  
   
    使用 `testparm` 命令检查添加的配置是否正确，出现以下提示则证明配置可用
   
   > ➜  ~ testparm  
   >  Load smb config files from /etc/samba/smb.conf  
   >  Loaded services file OK.  
   >  Weak crypto is allowed  
   > 
   >  </br>
   >  Server role: ROLE_STANDALONE  
   >	
   >  </br>
   >  Press enter to see a dump of your service definitions

4. 启动samba服务并设置开机自启
   
   ```bash
   #启动服务
   sudo systemctl start smbd   
   #设置开机自启
   sudo systemctl enable smbd
   #查看服务状态
   sudo systemctl status smbd
   ```

5. 为登录用户配置登陆密码并重启服务
   
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

6. 测试samba能否正常使用 
   
   ![](/images/SMB1.png)   

   输入服务器主机名或IP地址+目录名称  
   
   ![](/images/SMB2.png)  

   ![](/images/SMB3.png)  

