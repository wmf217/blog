---
layout:     post
title:      "PowerDesign使用记录"
subtitle:   " \"第一次使用PowerDesing详细记录\""
date:       2017-11-13 19:25:00
author:     "wmf"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - mysql
    - 软件
    - PowerDesign
---
### 1. 导出sql文件
 sql文件只要结构不要数据
### 2.导入PowerDesign
* >打开PowerDesign 点击`File->Reverse Enginner->Databases`

* >选择适用你SQL文件数据库类型，点击“确定”
 
* >点击实用“Using script files:”选项，然后点击，选中你SQL文件所在位置，然后点击确定，等待

![](/img/in-post/power-data.png)
### 3.画图
随便画，可以新建项目把数据表复制过来
>去掉背景图中的表格：`Tools -> Display Preferences` Show page delimiter 点掉
### 4.显示注释
[powerdesign 下ER模型中展示数据注释中文列](https://www.cnblogs.com/zh-haining/p/6291750.html)
### 5.导出文件
* 导出图片：先ctrl+A选择所有，点击`Edit->Export Image..`,最好选择emf格式分辨率比较高
* 导出word或html: 点击`Report->Generate report...`,选择rtf或html(print可以打印pdf，没研究)
![](/img/in-post/power-result.png)
