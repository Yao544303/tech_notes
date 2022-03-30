# VSCode\_Tips

## 下载慢

1. 挂VPN
2. 改链接

```
找到对应的文件，点击下载，会出现一个类似下面的链接：
https://az764295.vo.msecnd.net/stable/f06011ac164ae4dc8e753a3fe7f9549844d15e35/code_1.37.1-1565886362_amd64.deb 

将az764295.vo.msecnd.net 更换为如下内容：vscode.cdn.azure.cn

更换后的链接为：
https://vscode.cdn.azure.cn/stable/f06011ac164ae4dc8e753a3fe7f9549844d15e35/code_1.37.1-1565886362_amd64.deb

访问这个链接，你会发现没下载速度提升了几十倍。
其实就是用国内的镜像服务器加速。
```

[国内下载vscode速度慢解决](https://www.cnblogs.com/sctb/p/11919639.html)

## TO PDF

安装markdown to pdf，直接搜安装即可,可能出现安装依赖时报错。

```
set the http.proxy option to settings.json and restart Visual Studio Code.
```

F1打开设置，如下设置 重启即可

```
"settings": {
"http.proxy": "http://127.0.0.1:1080"
}
```

### 安装Markdown pdf插件，可以直接导出pdf格式，但是在非中文环境下，会出现中文有粗有细的现象

解决方案：修改 Markdown pdf的配置，将输出改为html使用chrome等打开，使用ctrl+p（打印），保存成pdf，格式保留的非常好
