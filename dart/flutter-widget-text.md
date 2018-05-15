---
title: "Flutter 基础组件-文本"
date: 2018-05-15T09:08:35+08:00
draft: true
tags: ["flutter","widget"]
---


文本组件,常常用来显示显示文本文字,当文字长度超过父组件时是截断还是显示多行,这个取决于属性的限制

#### 样式规则
1. 对于文本的样式是可选的,如果忽略将会使用与它最接近组件的默认样式

1. 如果设置了样式的`TextStyle.inherit`属性值,那么父组件的样式以及组件本身的样式将会同时作用

1. 如果需要使得同一行的文本的不同文字显示不同的样式,则需要使用`TextSpan.rich`作为构造函数


#### 属性
- **textAlign** 控制文本的对齐方式
    ```dart
    textAlign: TextAlign.justify,
    ```

- **textDirection** 文本的显示方向
    ```dart
    textDirection: TextDirection.rtl
    ```
Note: 需要注意的是.`textAlign`和`textDirection`两个属性存在相互影响

- **overflow** 如何处理溢出的文本
    ```dart
    overflow: TextOverflow.clip
    ```

- **softWrap** 溢出的文字是否在下一行显示
    ```dart
    softWrap: true,
    ```

Note: `overflow`和`softWrap`存在相互影响

- **style** 文本的样式
    ```dart
    style: new TextStyle(fontSize: 40.0),
    ```

- **textScaleFactor** 缩放比例
    ```dart
    textScaleFactor: 1.5
    ```

#### Simple
```dart
child: new Text(
    'Helloaaaaaaaaaaaaaaaaaaaaaaaaaaa',
    textAlign: TextAlign.justify,
    // overflow: TextOverflow.clip,
    textDirection: TextDirection.rtl,
    // softWrap: true,
    style: new TextStyle(fontSize: 40.0),
    // textScaleFactor: 1.5,
),
```


#### 参考资料

- [Text Class](https://docs.flutter.io/flutter/widgets/Text-class.html)

