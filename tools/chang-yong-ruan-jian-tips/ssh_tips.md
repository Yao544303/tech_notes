# SSH\_Tips

## 禁止root 登陆，更换ssh端口

### 先使用root用户登陆，创建新用，为新用户创建密码

```
useradd guest
passwd guest
```

```
[root@hecs-235796 ~]# useradd guest
[root@hecs-235796 ~]# passwd guest
Changing password for user guest.
New password:
BAD PASSWORD: The password fails the dictionary check - it is based on a dictionary word
Retype new password:
passwd: all authentication tokens updated successfully.
```

### 为用户分配sudo权限

```
gpasswd -a guest wheel  //给予sudo权限, 当权限不够时，可以用sudo
 
lid -g wheel             //查询所有带sudo权限的用户
 
// 如果想删除用户
userdel -r guest        //删除用户和相应的目录
```

```
[root@hecs-235796 ~]# gpasswd -a guest wheel
Adding user guest to group wheel
[root@hecs-235796 ~]# lid -g wheel
guest(uid=1001)
```

### 用户创建完毕，切换guest用户登陆（因为要为当前用户生成密钥）

```
su guest
ssh-keygen -t rsa -b 4096 -C "guest@guest.com" //生成密钥，邮件换成自己的
 
生成之后 ，
私钥是/home/guest/.ssh/id_rsa
公钥是id_rsa.pub
可以更改私钥名称为id_rsa_guest.pem,公钥为authorized_keys
```

### 禁止密码登陆和启用密钥验证

修改/etc/ssh/sshd\_config文件，禁止密码登陆

```
PasswordAuthentication no 
```

修改/etc/ssh/sshd\_config文件，启用密钥验证

```
RSAAuthentication yes
PubkeyAuthentication yes
```

重启ssh服务

```
systemctl restart sshd.service
```

### 禁止root 登陆

修改/etc/ssh/sshd\_config文件

```
PermitRootLogin no
```

重启ssh服务

```
systemctl restart sshd.service
```

### 修改ssh 端口

查看当前ssh服务器端口号

```
netstat -tunlp | grep "ssh"

tcp        0      0 0.0.0.0:12312           0.0.0.0:*               LISTEN      1789/sshd
tcp6       0      0 :::12312                :::*                    LISTEN      1789/sshd
```

修改默认的ssh服务器端口

```
vi /etc/ssh/sshd_config

Port xxxxx  #改成你需要的端口，记得网络策略放行
```

重启ssh服务

```
systemctl restart sshd.service
```
