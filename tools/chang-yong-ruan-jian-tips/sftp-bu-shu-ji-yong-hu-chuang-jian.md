# SFTP部署及用户创建

## 创建sftp组，sftp用户密码

```
groupadd sftp
useradd -g sftp -s /sbin/nologin -M sftp //-g：加入主要组 -s指定用户登入后所使用的shell -M：不要自动建立用户的登入目录
passwd sftp
```

## 创建sftp用户的根目录和属主、属组，修改权限（755）

```
cd /usr/
mkdir sftp
chown root:sftp sftp/
chmod 755 sftp/
```

## 在sftp 的目录中创建可写入的目录

```
cd sftp/
mkdir file
chown sftp:sftp file
```

注释：可根据目录权限控制 普通用户的 上传和下载的 权限操作。 也可创建普通用户加如sftp组。

## 修改sshd\_config 的配置文件

```
vi /etc/ssh/sshd_config
```

把原来的Subsystem 行注释掉，添加如下配置：

```
#Subsystem      sftp    /usr/libexec/openssh/sftp-server      #注释掉这行，增加以下6行
Subsystem sftp internal-sftp    实际只增加此行即可工作，以下各行根据个人需求
Match Group home （用户组名，我用的home（groupadd  home））                                       
ChrootDirectory /home/%u                 #设定属于用户组sftp的用户访问的根文件夹
ForceCommand internal-sftp
AllowTcpForwarding no
X11Forwarding no                        #设置不允许SSH的X转发

#UseDNS no                              #注释掉
#AddressFamily inet                     #注释掉
#SyslogFacility AUTHPRIV                #注释掉    
```

配置完成后重启服务 systemctl restart sshd.service

## 测试

sftp sftp@127.0.0.1

默认端口22 用户：sftp 密码：xxx
