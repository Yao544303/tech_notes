# Hexo\_Tips

##

## Hexo 常用命令

新建页面

```
hexo new page <title>  
```

新建文章

```
hexo new post <title>  
```

生成

```
hexo g   
```

发布

```
hexo d   
```

生成+发布

```
hexo d -g  
```

在本地4040端口预览，预览之后再进行发布

```
hexo s   
```



## 添加赞赏

在\_config.yml中配置

```
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/wechat.jpg
alipay: /images/aipay.png
```

## hexo在添加新文章的时候怎么默认在md文件中添加categories 以及 tags 标签

在博客的 scaffolds 里有个post.md 添加上需要的配置就行了， 这里是创建post的模板。

## hexo next 主题，首页文章收起，有显示全文

[Hexo之next主题设置首页不显示全文(只显示预览](https://www.jianshu.com/p/393d067dba8d)

## 文章头添加图片

```
layout: photo
title: 我的阅历
date: 2085-01-16 07:33:44
tags: [hexo]
photos:
 - http://bruce.u.qiniudn.com/2013/11/27/reading/photos-0.jpg
 - http://bruce.u.qiniudn.com/2013/11/27/reading/photos-1.jpg
```

## hexo 不想手动添加一些标题

hexo/scaffolds/photo.md

```
---
layout: {{ layout }}
title: {{ title }}
date: {{ date }}
tags: 
photos: 
---
```

然后每次可以执行带layout的new命令生成照片文章：&#x20;

```
hexo new photo "photoPostName" #新建照片文章
```

## 添加转载请注明出处

* [hexo文章添加版权声明及一些特效](http://tc9011.com/2017/02/02/hexo%E6%96%87%E7%AB%A0%E6%B7%BB%E5%8A%A0%E7%89%88%E6%9D%83%E5%A3%B0%E6%98%8E%E5%8F%8A%E4%B8%80%E4%BA%9B%E7%89%B9%E6%95%88/)
* [各个license的说明](http://creativecommons.net.cn/licenses/meet-the-licenses/)

## Hexo支持数学公式

在 hexo 中，你会发现我们不能用 Latex 语法来书写数学公式，这对于书写学术博客来说是很大的不便，因为我们会经常碰到很多的数学公式推导，但是我们可以通过安装第三方库来解决这一问题。

### 第一步： 使用Kramed代替 Marked

hexo 默认的渲染引擎是 marked，但是 marked 不支持 mathjax。 kramed 是在 marked 的基础上进行修改。我们在工程目录下执行以下命令来安装 kramed.

```
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

在node\_modules 目录下，会生成kramed的目录

然后进入node\_modules 目录下，kramed的目录 更改/node\_modules/hexo-renderer-kramed/lib/renderer.js

```
// Change inline math rule
function formatText(text) {
    // Fit kramed's rule: $$ + \1 + $$
    return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
}
```

为

```
// Change inline math rule
function formatText(text) {
    return text;
}
```

### 第二步： 停止使用 hexo-math

首先，如果你已经安装 hexo-math, 请卸载它：

```
npm uninstall hexo-math --save
```

然后安装 hexo-renderer-mathjax 包：

```
npm install hexo-renderer-mathjax --save
```

### 第三步: 更新 Mathjax 的 CDN 链接

首先，打开/node\_modules/hexo-renderer-mathjax/mathjax.html

然后，把更改为：

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

### 第四步: 更改默认转义规则

因为 hexo 默认的转义规则会将一些字符进行转义，比如 \_ 转为 _, 所以我们需要对默认的规则进行修改. 首先， 打开_

```
escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
```

更改为:

```
escape: /^\\([`*\[\]()# +\-.!_>])/,
```

把

```
em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

更改为:

```
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

### 第五步: 开启mathjax

在主题 \_config.yml 中开启 Mathjax， 找到 mathjax 字段添加如下代码：

```
mathjax:
    enable: true
```

这一步可选，在博客中开启 Mathjax，， 添加以下内容：

```
---
title: Testing Mathjax with Hexo
category: Uncategorized
date: 2017/05/03
mathjax: true
---
```



## REF

* [Next主题(Hexo)](https://www.jianshu.com/p/5d5931289c09)
* [博客二之博客美化](http://chenfengkg.cn/optimize-blog/)
* [next github](https://github.com/iissnan/hexo-theme-next/issues)
* [Hexo NexT主题中集成gitalk评论系统](https://asdfv1929.github.io/2018/01/20/gitalk/)
* [Hexo+Github+jsDelivr+Vercel建站备忘录](https://www.vincentqin.tech/2016/08/09/build-a-website-using-hexo/%E5%A2%9E%E5%8A%A0Gitter)
