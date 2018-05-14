---
title: "Flutter 基础组件-列"
date: 2018-05-11T23:10:25+08:00
draft: true
tags: ["Flutter", "widget"]
---

> 本文章主要翻译于[Widgets: Column](https://flutterdoc.com/widgets-column-e129769fbcb3)

Column组件常用来显示垂直方向的子组件列表,与Row组件一样,该组件只会显示其可见范围内的子组件,并不会滚动显示其子组件

如下,可以创建一个具有三个子组件的Column组件
```dart
    body: new Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
        new Text('The first  line of text'),
        new Text('The second line of text'),
        new Text('The third  line of text'),
    ],
    ),
```

#### 布局步骤

1. 优先布局子组件为`null`以及不可伸缩组件(相对的可伸缩组件为:`Expanded`),
    对于这些组件在垂直方向上没有做限制,在水平方向上限制为屏幕的大小
    如果设定`crossAxisAlignment:CrossAxisAlignment.stretch`,在水平方向上将会匹配其宽度的最大值

1. 然后将可伸缩的组件布局于剩下的空间,在这一步的时候,子组件还没有被布局

1. 当布局可伸缩组件时,采用的规则与第一步的规则相似,只不过不会将已经布局的不可伸缩组件挤出屏幕之外,而是自身超出部分不可见

1. Column的宽度,子组件在水平方向的显示永远是安全的,也就是说水平方向永远可见

1. Column的高度取决于`mainAxisSize`属性

1. 子组件的位置确定,取决于`mainAxisAlignment`和`crossAxisAlignment`两个组件


#### 属性

- **crossAxisAlignment** 子组件在横轴方向上的定位
    ```dart
       crossAxisAlignment: CrossAxisAlignment.start,
    ```

- **textBaseline** 文本的对齐方式
    ```dart
        textBaseline: TextBaseline.ideographic,
    ```

- **textDirection** 子组件在水平方向的排列顺序
    ```dart
        textDirection: TextDirection.ltr
    ```

- **verticalDirection** 子组件在垂直方向上的排列顺序
    ```dart
        verticalDirection: VerticalDirection.down,
    ```

- **mainAxisAlignment** 子组件在主轴方向上的定位
    ```dart
    mainAxisAlignment: MainAxisAlignment.end,
    ```

#### 完整实例
```dart
import 'package:flutter/material.dart';

void main() => runApp(new HomePage());

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => new _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        title: 'Flutter Demo',
        home: new Scaffold(
          appBar: new AppBar(
            title: new Text('Flutter Container'),
          ),
          body: new Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            // textBaseline: TextBaseline.ideographic,
            // textDirection: TextDirection.ltr,
            // verticalDirection: VerticalDirection.down,
            // mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: <Widget>[
              new Text(
                'The first line of text',
                style: new TextStyle(fontSize: 40.0),
              ),
              new Text(
                'The third line of text',
                style: new TextStyle(fontSize: 40.0),
              ),
              new Expanded(
                flex: 1,
                child: new Text(
                  'The second line of text,The second line of text,The second line of text,The second line of text',
                  style: new TextStyle(fontSize: 40.0),
                ),
              ),
            ],
          ),
        ));
  }
}
```

#### 参考文件
- [Widgets: Column](https://flutterdoc.com/widgets-column-e129769fbcb3)

- [Column Class](https://docs.flutter.io/flutter/widgets/Column-class.html)