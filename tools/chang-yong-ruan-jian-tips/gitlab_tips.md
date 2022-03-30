# Gitlab\_Tips

## 在ubuntu 上搭建gitlab

```
sudo gitlab-ctl stop 停止
sudo gitlab-ctl start 开启
sudo gitlab-ctl restart 重启
sudo gitlab-ctl status 查看状态
sudo gitlab-ctl reconfigure 确认配置（修改配置后，必须执行）
sudo gitlab-ctl tail  查看日志
配置GitLab域名 ，否则项目git clone的地址时错
vi /etc/gitlab/gitlab.rb
编辑：external_url '你的网址'
例如：external_url 'http://192.168.1.100'
编辑完成后，再sudo gitlab-ctl reconfigure一下，使配置生效
```

### REF

* [在ubuntu 16.04下安装gitlab（摘抄中文官方网站）](https://www.cnblogs.com/gabin/p/6385908.html)&#x20;
* [在ubuntu16上搭建gitlab（实测可用）](https://blog.csdn.net/qq\_36467463/article/details/78283874)
