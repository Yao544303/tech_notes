# MacOS\_Tips

## 中英文切换

```
ctrl + space
```

## Mac OSX 修改主机名(解决bogon)

打开终端，发现主机名莫名其妙变成了bogon，强迫症表示不能忍，于是找到以下解决方案

```
sudo hostname your-desired-host-name 
sudo scutil --set LocalHostName $(hostname) 
sudo scutil --set HostName $(hostname)
```

[ref](https://blog.csdn.net/yuukoiry/article/details/52673147)

## 修改host

```
sudo vi /etc/hosts
```

## 如何强制退出应用

同时按住三个按键：Option、Command 和 Esc (Escape) 键。这类似于在 PC 上按下 Control-Alt-Delete。或者，在屏幕左上角的苹果 () 菜单中选取“强制退出”。

## 默认zsh

mac os 系统默认的终端为bash，切换默认终端为zsh，可以用以下命令

```
chsh -s /bin/zsh
```

如过要切回默认终端bash则使用以下命令

```
chsh -s /bin/bash
```

## 取词翻译

```
ctrl command D
```

[ref](https://www.jianshu.com/p/88f99600b9cd)
