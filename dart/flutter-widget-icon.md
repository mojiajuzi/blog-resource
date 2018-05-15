---
title: "Flutter 基础组件-图标"
date: 2018-05-15T10:48:15+08:00
draft: true
tags: ["flutter","widget"]
---

对于图标组件来说其不存在交互性,如果需要使用交互性,需要使用`IconButton`组件

#### Icon 组件

```dart
    new Icon(
    Icons.star, // 图标数据
    size: 40.0, //大小
    color: Colors.red, //颜色
    semanticLabel: 'Hello', // 图标不显示时,显示的文本
    ),
```

#### IconButton 组件
```dart
    new IconButton(
    icon: new Icon(
        Icons.star,
        // color: Colors.teal,
    ), // 图标
    color: Colors.red, //图标显示颜色
    splashColor: Colors.blue, //图标扩散时的颜色
    highlightColor: Colors.yellow, //单击时轮廓颜色
    disabledColor: Colors.green, //图标不可用时显示的颜色
    iconSize: 80.0, //图标大小
    padding: const EdgeInsets.all(20.0), //图标内边距
    onPressed: () {}, // 图标点击事件
    tooltip: 'IconButton', // 点击时的提示信息
```
Note:这里有一点需要注意的是,`IconButton`组件是否可用,取决于其`onPresses`属性是否为null,如果为`null`那么将会只显示`disabledColor`属性设置的颜色,其他所有的颜色将全部不显示


#### 外部图标
关于外部图标的引用,可以参考一下这篇文章[Icons](http://flutter.link/2018/03/09/Icons/)