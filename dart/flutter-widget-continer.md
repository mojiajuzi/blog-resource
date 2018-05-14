---
title: "Flutter 基础组件-容器"
date: 2018-05-10T22:58:56+08:00
draft: true
tags: ["Flutter", "widget"]
---

### Container Widget

> 本文章主要翻译于[Widgets: Container](https://flutterdoc.com/widgets-container-d8eee21ad2f4)

#### 特点
- 该组件常常用来包含子组件,同时其本身也包含一些样式属性,
- 如果没有包含子组件的时候,将会自动填充屏幕上给定的区域
- 如果设定了长度和宽度,将会作用域其子组件

假定我们创建了如下一个示例
```dart
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        title: 'Flutter Demo',
        home: new Scaffold(
          appBar: new AppBar(
            title: new Text('Flutter Container'),
          ),
          body: new Container(
              width: 200.0,
              height: 200.0,
              color: Colors.amber.shade400,
              child: Container(
                width: 100.0,
                height: 100.0,
                color: Colors.greenAccent.shade200,
              )),
        ));
  }
```

以上我们先创建一个容器,并设定其宽度和高度分别为200.0和200.0, 指定其填充色为橘黄色

然后创建该容器的子组件为容器,同样设定其长度和高度为100.0和100.0, 指定其颜色,

实际上,我们只能看见容器子容器的颜色,这是因为如果没有对容器中,如果没有对子组件进行限定的话

子组件将会完全沾满container容器,因此我们可以使用容器的属性对子组件进行限定

#### 属性
- **padding** 设定子组件与容器的内边距

    ```dart
    padding: const EdgeInsets.only(top: 10.0, bottom: 0.0, left: 0.0),
    ```
- **alignment** 设定子组件相对于容器的相对位置
    ```dart
    alignment: FractionalOffset.topCenter,
    ```

- **margin** 设置容器的外边距
    ```dart
    margin: const EdgeInsets.all(20.0),
    ```

- **color** 设置容器的填充色
    ```dart
    color: Colors.amber.shade400,
    ```

- **foregroundDecoration** 设置容器的前景色,将会覆盖其子元素
    ```dart
        foregroundDecoration:
            new BoxDecoration(color: Colors.amber.shade400),
    ```

- **decoration** 设置容器的颜色,图片,边框等
    ```dart
        decoration: new BoxDecoration(
            color: const Color(0xff7c94b6),
            image: new DecorationImage(
                image: new ExactAssetImage('images/hello.png'),
                fit: BoxFit.cover),
            border: new Border.all(color: Colors.black, width: 8.0)),
    ```
    需要注意的是,`color`,`foregroundDecoration`,`decoration`三者不能同时使用

- **trans** 设置旋转,可以选择`X,Y,Z`三个方向
    ```dart
    transform: new Matrix4.rotationZ(0.1),
    ```

- **decoration** 约束容器
    ```dart
    constraints: new BoxConstraints.expand(width: 100.0),
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
          body: new Container(
              alignment: FractionalOffset.topCenter,
              padding: const EdgeInsets.only(top: 10.0, bottom: 0.0, left: 0.0),
              decoration: new BoxDecoration(
                  color: const Color(0xff7c94b6),
                  image: new DecorationImage(
                      image: new ExactAssetImage('images/hello.png'),
                      fit: BoxFit.cover),
                  border: new Border.all(color: Colors.black, width: 8.0)),
              // foregroundDecoration:
              //     new BoxDecoration(color: Colors.amber.shade400),
              // color: Colors.amber.shade400,
              // transform: new Matrix4.rotationZ(0.1),
              constraints: new BoxConstraints.expand(width: 100.0),
              width: 200.0,
              height: 200.0,
              child: Container(
                margin: const EdgeInsets.all(20.0),
                width: 100.0,
                height: 100.0,
                color: Colors.greenAccent.shade200,
                alignment: FractionalOffset.topRight,
                // decoration: new BoxDecoration(color: Colors.blue.shade300),
                child: new Text('H'),
              )),
        ));
  }
}
```

####  参考资料

- [Widgets: Container](https://flutterdoc.com/widgets-container-d8eee21ad2f4)

- [Container Class](https://docs.flutter.io/flutter/widgets/Container-class.html)