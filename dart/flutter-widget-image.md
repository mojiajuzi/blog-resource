---
title: "Flutter 基础组件-图像"
date: 2018-05-14T23:30:51+08:00
draft: true
tags: ["Flutter", "widget"]
---



> 本文章主要翻译于[Widgets: Image](https://flutterdoc.com/widgets-image-5a2296493329)

对于图像组件而言,其提供了以下几种方式来获取图片
- Image,从一个图片提供者(ImageProvider)处获取图片

- Image.assert 从(AssertBundle)处使用key来获取图片

- Image.network 通过URL从网络获取图片

- Image.file 从文件读取图像

- Image.memory 从Uint8List中获取图像


#### 属性

- **alignment** 图像对齐

- **fit** 图像的填充方式

- **height** 设置图片的高度

- **width** 设置图像的宽度

- **gaplessPlayback** 当使用的图像源变化时,图片如何切换

- **colorBlendMode** 两张重叠的图像如何显示