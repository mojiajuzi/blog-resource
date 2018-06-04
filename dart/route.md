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

1. 直接在组件中定义
  ```dart
      body: new Center(
        child: new RaisedButton(
          child: new Text('Launch new screen'),
          onPressed: () {
            Navigator.of(context).push(
                  new MaterialPageRoute(builder: (context) => new SecondPage()),
                );
          },
        ),
      ),
```

对于作为属性的的路由来说,其缺乏传递动态参数的能力,对于直接在组件中定义的路由而言,对于其管理又存在一些不便


#### 参数传递

##### 发送数据到其他页面

为了传递参数到其他的页面，只需要在定义其他页面的时候添加初始化的方法声明需要的参数类型即可，比如:

```dart
class SecondPage extends StatelessWidget {
  
  final int backAccount;

  SecondPage({Key key, @required this.backAccount}):super(key:key);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(title: new Text('Second'),),
      body: new Center(
        child: new RaisedButton(
          child: new Text(backAccount.toString()),
          onPressed: (){
            Navigator.of(context).pop();
          },
        ),
      ),
    );
  }
}
```

在这里初始化的时候，需要限定传递参数，并且使用`@required`(注意：这里需要引入`import 'package:flutter/foundation.dart';`)注释`backAccount`参数为必需

然后我们只需要在实例化的时候传递必需的参数即可

```dart
  int _backCount = 0;

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(title: 'First Page',home: new Scaffold(
      appBar: new AppBar(title: new Text('First Page'),),
      body: new Builder(
        builder: (BuildContext context){
          return new Center(
              child: new RaisedButton(
                child: new Text('GO To Second'),
                onPressed: (){
                  Navigator.push(
                    context,
                    new MaterialPageRoute(builder: (context) => new SecondPage(backAccount: _backCount,)),
                  );
                },
              ),
          );
        },
      ),
    ),);
  }
}
```

在这里只是传递一个返回的次数统计,然而这里的功能并不齐全，因为我们还没有返回参数，并对返回的结果进行处理

##### 接受其他页面返回的参数
为了接受其他页面返回的参数，我们需要做一个异步处理等待其它页面的返回，因此我么需要使用到`dart:async`这个包

1. 首先将路由抽离出来

    ```dart
      Future getBackAccount(BuildContext context) async {
          int times = await Navigator.push(context,
            new MaterialPageRoute<int>(builder: (context) => new SecondPage(backAccount: _backCount,)),
          );

          if(times != null){
            setState(() {
                    _backCount = times;  
            });
          }
      }

    }
    ```
    这里将路由抽离出来放到一个单独的函数里面，限定返回的数据类型为int类型
    
    如果返回的数据不是一个null类型，则将返回的值赋值给`_backCount参数`

1. 在`SecondPage`中，只需要在`Navigator.pop`方法中将值返回即可

    ```dart
          onPressed: (){
            backAccount += 1;
            Navigator.of(context).pop(backAccount);
          },
    ```

#### 参考文档

- [ Flutter Cookbook - Navigation](https://flutter.io/cookbook/)

- [BuildContext-class](https://docs.flutter.io/flutter/widgets/BuildContext-class.html)

##### 完成代码

```dart
import 'dart:async';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';


void main() => runApp(new FirstPage());



class FirstPage extends StatefulWidget {
  @override
  _FirstPageState createState() => new _FirstPageState();
}

class _FirstPageState extends State<FirstPage> {

  int _backCount = 0;

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(title: 'First Page',home: new Scaffold(
      appBar: new AppBar(title: new Text('First Page'),),
      body: new Builder(
        builder: (BuildContext context){
          return new Center(
              child: new RaisedButton(
                child: new Text('GO To Second'),
                onPressed: (){
                  getBackAccount(context);
                }
              ),
          );
        },
      ),
    ),);
  }

  Future getBackAccount(BuildContext context) async {
      int times = await Navigator.push(context,
        new MaterialPageRoute<int>(builder: (context) => new SecondPage(backAccount: _backCount,)),
      );

      if(times != null){
        setState(() {
                _backCount = times;  
        });
      }
  }

}


class SecondPage extends StatelessWidget {
  
  int backAccount;

  SecondPage({Key key, @required this.backAccount}):super(key:key);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(title: new Text('Second'),),
      body: new Center(
        child: new RaisedButton(
          child: new Text(backAccount.toString()),
          onPressed: (){
            backAccount += 1;
            Navigator.of(context).pop(backAccount);
          },
        ),
      ),
    );
  }
}

```