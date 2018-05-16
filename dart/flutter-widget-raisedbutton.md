---
title: "Flutter Widget Raisedbutton"
date: 2018-05-16T09:37:39+08:00
draft: true
tags: ["flutter","widget"]
---

在`RaisedButton`组件中,有两种方式认为其是不可用的

- 通过设置`enabled`属性为`false`(在beta3.0中已经去掉该属性)

- 通过设置`onPressed`属性值为`null`,来禁用组件


```dart
import 'package:flutter/material.dart';

void main() => runApp(new HomePage());

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => new _HomePageState();
}

class _HomePageState extends State<HomePage> {
  ShapeBorder myShape;

  _changeShape() {
    setState(() {
      myShape = new StadiumBorder();
    });
  }

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        title: 'Flutter Demo',
        home: new Scaffold(
            appBar: new AppBar(
              title: new Text('Flutter Container'),
            ),
            body: new Center(
              child: new RaisedButton(
                onPressed: _changeShape,
                color: Colors.blue, // 颜色
                colorBrightness: Brightness.dark, //亮度
                // disabledColor: Colors.deepOrangeAccent, //不可用时的颜色
                // disabledTextColor: Colors.lightGreen, // 不可用时文字颜色
                // elevation: 1.0, // 按钮阴影大小
                // highlightColor: Colors.indigo, //点击时的高亮色
                // highlightElevation: 1.0,  // 点击时的阴影大小
                shape: myShape, //变更按钮形状
                // splashColor: Colors.red, //点击时波纹扩散色
                // textColor: Colors.yellow, // 文本色
                // textTheme: ButtonTextTheme.normal, //定制化按钮色彩
                animationDuration: new Duration(
                    hours: 0, minutes: 0, seconds: 3), //定义图标完成变换时所需要的时间
                child: new Text('Hello Button'), //子组件
              ),
            )));
  }
}
```