---
title: "Flutter 路由"
date: 2018-05-31T00:31:23+08:00
draft: true
---

### 基本概念

- 路由(Route),路由在Flutter中是屏幕或者页面的抽象

- 导航(Navigator),导航是一个用来管理路由的组件

- Navigator通过栈的形式来管理路由对象，
    由于栈有出栈和入栈两个操作，相对应的Navigator也通过`pop`和`push`来与之对应


### 路由的定义

1. 作为`MaterialApp`的属性
    ```dart
    return new MaterialApp(
      title: 'Flutter Demo',
      routes:<String, WidgetBuilder>{
        '/home':(BuildContext context) => new HomePage(),
        '/detail':(BuildContext context) => new DetailPage(),
      },
    );
    ```

1. TransitionRoute


1. ModalRoute


1. MaterialPageRoute(动画基于特定的平台，也就是说不同的平台动画效果可能不一样)


1. PopupRoute

1. PageRoute