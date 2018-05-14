---
title: "Flutter 基础组件-行"
date: 2018-05-11T23:17:47+08:00
draft: true
tags: ["Flutter", "widget"]
---

> 本文章主要翻译于[Widgets: Row](https://flutterdoc.com/widgets-row-4ff6c5cfb9e0)

Row组件主要用来在水平方向显示其子组件， 当组件作为子组件被放置在Row中时，子组件不能滚动

只显示在视图中可见的组件


例如，可以像如下使用一个Row组件

```dart
new Row(
    children: <Widget>[
    new Text(‘The first text widget’, textAlign: TextAlign.center),
    new Text(‘The first text widget’, textAlign: TextAlign.center),
    new Text(‘The first text widget’, textAlign: TextAlign.center)
 ],
)
```
在Row中，子组件按照顺序逐个布局，再看一下下面这个例子
```dart
new Row(
    children: <Widget>[
    const FlutterLogo(),
    new Text("This is a text widget to be   
    displayed on the screen, as you can see it is a pretty long text widget, right?"),
    const FlutterLogo()
 ],
)
```
这个组件的布局顺序如下：
- 首先布局`FlutterLogo`，其占据其大小的屏幕空间

- 然后是一个文本组件，这里存在的一个问题是，文本组件的字符长度太长了，以至于只能在水平方向上显示部分文字

- 最后是第二个`FlutterLogo`,由于文本已经把剩下的空间全部占用了，因此该组件就无法显示了

因此对于水平布局组件Row而言，其一个需要注意的问题就是屏幕空间大小的问题,

为了解决这个问题，我们需要引入`Expanded`组件

```dart
new Row(
  children: <Widget>[
    const FlutterLogo(),
    const Expanded(child: const Text("This is a text widget to be   
    displayed on the screen, as you can see it is a pretty long text 
    widget, right?")),
    const FlutterLogo()
  ],
)
```

修改以后，其布局顺序也发生了改变，首先会计算两个`FlutterLogo`所占据的屏幕空间，

然后再计算被`Expanded`包裹的文本组件

#### 布局步骤

1. 优先布局子组件为`null`以及不可伸缩组件(相对的可伸缩组件为:`Expanded`),
    对于这些组件在水平方向上没有做限制,在垂直方向上限制为屏幕的大小
    如果设定`crossAxisAlignment:CrossAxisAlignment.stretch`,在水平方向上将会匹配其宽度的最大值

1. 然后将可伸缩的组件布局于剩下的空间,在这一步的时候,子组件还没有被布局

1. 当布局可伸缩组件时,采用的规则与第一步的规则相似,只不过不会将已经布局的不可伸缩组件挤出屏幕之外,而是自身超出部分不可见

1. Row的高度,子组件在水平方向的显示永远是安全的,也就是说水平方向永远可见

1. Row的宽度取决于`mainAxisSize`属性

1. 子组件的位置确定,取决于`mainAxisAlignment`和`crossAxisAlignment`两个组件


#### 属性

- **children** 组件列表，被放置于Row中
    ```dart
    children:<widget>[...]
    ```

- **textBaseline** 设置子组件对其的基准线
    ```dart
    textBaseLine: TextBaseLine.alphabetic
    ```

- **textDirection** 设置子组件的排序方式
    ```dart
    textDirection: TextDirection.ltr
    ```

- **vertically** 设置子组件垂直排列的方式以及开始和结束位置
    ```dart
    verticalDirection: VerticalDirection.up
    ```

- **mainAxisAligment** 子组件如何沿主轴放置
    ```dart
    mainAxisAligment: MaxAxisAligment.up
    ```

- **maxAxisSize** 子组件该如何分配主轴空间
    ```dart
    mainAxisSize: MainAxisSize.max
    ```




#### 参考资料

- [Widgets: Row](https://flutterdoc.com/widgets-row-4ff6c5cfb9e0)

- [Row Class](https://docs.flutter.io/flutter/widgets/Row-class.html)