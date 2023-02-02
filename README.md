## 文章推送步骤

### 1、安装

* 安装hexo
https://github.com/hexojs/hexo

* 安装依赖
```sh
npm uninstall hexo-prism-plugin
npm install hexo-generator-search --save
npm i --save hexo-wordcount
```

- 更新子模块

```sh
git submodule update --init --recursive
```

### 2、clone 工程

```
git clone https://github.com/ALightGroup/ALightGroup.github.io
```

### 3、编写文章
在`source/_posts` 目录下编写文章，文章头需要添加下面header
```
---
title: Android oneDrive 集成（一）-- SDK申请
toc: true
date: 2021-02-09 16:16:22
author: <a href="https://github.com/AriaLyy">AriaLyy</a>   
revision: 
tags: 
  - onedrive
categories: android
---
```

`author`：文章作者，如果希望进行跳转可以使用a标签，如
```
author: <a href="https://github.com/AriaLyy">AriaLyy</a>
```

`revision`: 审校团队，同样支持a标签，如果没有审校团队，留空

页脚效果
![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/20230105175124.png)

页头效果
![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/20230105175155.png)

如果需要分割文章，需要添加
```
<!--more-->
```

参考[Version Catalog.md](https://github.com/ALightGroup/ALightGroup.github.io/blob/main/source/_posts/gradle/Version%20Catalog.md)


### 4、本地预览
使用下面地址进行本地预览
```
hexo s
```


### 5、推送文章
将文章推送到仓库，Actions会自动进行部署


### 6、图床
使用 https://github.com/Molunerfinn/PicGo 将图片上传到github仓库 </br>
图传配置：https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A

![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/20221228212653.png)














