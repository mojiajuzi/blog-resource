---
title: "Flutter 基础组件-表单"
date: 2018-05-29T17:35:50+08:00
draft: true
tags: ["flutter","widget"]
---

#### TextField, FormField

对于`TextField`和`FormField`而言,只要用来接收用户的输入,以及用户输入的事件处理,
因此,可以将其分解为三个部分来说明

##### 样式属性

```dart
child: new TextField(
autocorrect: false, // 是否自动校正
autofocus: false, //自动获取焦点
enabled: true, // 是否启用
inputFormatters: [], //对输入的文字进行限制和校验
keyboardType: TextInputType.text, //获取焦点时,启用的键盘类型
maxLines: 2, // 输入框最大的显示行数
maxLength: 3, //允许输入的字符长度/
maxLengthEnforced: false, //是否允许输入的字符长度超过限定的字符长度
obscureText: true, // 是否隐藏输入的内容
onChanged: (newValue) {
    // print(newValue); // 当输入内容变更时,如何处理
},
onSubmitted: (value) {
    // print("whar"); // 当用户确定已经完成编辑时触发
},
style: new TextStyle(
    color: new Color(Colors.amberAccent.green)), // 设置字体样式
textAlign: TextAlign.center, //输入的内容在水平方向如何显示
decoration: new InputDecoration(
    labelText: "城市",
    icon: new Icon(Icons.location_city),
    border: new OutlineInputBorder(), // 边框样式
    helperText: 'required',
    hintText: '请选择你要投保的城市',
    prefixIcon: new Icon(Icons.android),
    prefixText: 'Hello'),
),
```

##### 输入处理
在`TextField`中,有两个事件是我们应该关注的`onChanged`,`onSubmitted`

```dart
child: new TextField(
controller: _controller,
decoration: new InputDecoration(labelText: 'Your Name'),
onChanged: (val) {
    print(val);
},
onSubmitted: (String v) {
    print(v);
},
),
```

顾名思义:
- `onChanged`事件,在输入内容发生变化的时候触发
- `onSubmitted`事件,则是在输入结束,点击完成的时候触发


然而在`TextFormField`中没有这两个事件,取而代之的是`validator`, `onSaved`,`onFieldSubmitted` 他们都接受三个函数,并且将其值作为参数传递到函数里面

- validator,如果开启`autovalidate: true,`那么将会自动检验输入的值,如果没有则会在表单提交的时候检验
该函数只允许返回验证失败的错误信息以及验证通过时返回`null`

- onSaved, 当调用`FormState.save`方法的时候调用

- onFieldSubmitted, 与`onSubmitted`一样,则是在输入结束,点击完成的时候触发

##### 编辑控制
无论是在`TextField`还是`TextFormField`中,都有一个重要的属性`controller`,该属性可用来对输入框内容进行控制

先创建一个控制对象

```dart
  TextEditingController _controller = new TextEditingController();
  TextEditingController _formFieldController = new TextEditingController();
```

为输入框初始化值以及注册一个监听事件
```dart
  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    _controller.value = new TextEditingValue(text: 'Hello');
    _formFieldController.addListener(() {
      print('listener');
    });
  }
```

触发一个监听事件

```dart
  void _textFieldAction() {
    // print(_formFieldController.selection);
    // print(_formFieldController.text); //获取输入内容
    print(_formFieldController.hasListeners); //判断是否注册监听事件
    _formFieldController.notifyListeners(); //触发监听事件
  }
```

#### Form
对于表单而言,其就像是一个容器,里面包含了`TextFormField`的一个列表
下面通过一个例子说明表单的一些特性

1. 布局

```dart
 @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new MaterialApp(
      title: 'Flutter data',
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text('Flutter Form'),
        ),
        floatingActionButton: new FloatingActionButton(
          onPressed: _forSubmitted,
          child: new Text('提交'),
        ),
        body: new Container(
          padding: const EdgeInsets.all(16.0),
          child: new Form(
            key: _formKey,
            child: new Column(
              children: <Widget>[
                new TextFormField(
                  decoration: new InputDecoration(
                    labelText: 'Your Name',
                  ),
                  onSaved: (val) {
                    _name = val;
                  },
                ),
                new TextFormField(
                  decoration: new InputDecoration(
                    labelText: 'Password',
                  ),
                  obscureText: true,
                  validator: (val) {
                    return val.length < 4 ? "密码长度错误" : null;
                  },
                  onSaved: (val) {
                    _password = val;
                  },
                ),
              ],
            ),
          ),
        ),
      ),
    );
```

以上,我们使用一个`Form`包裹着两个`TextFormField`组件,在这里为了简便,我们只设置了一些必要的元素,先暂时忽略`TextFormField`中的事件

为了获取表单的实例,我们需要设置一个全局类型的key,通过这个key的属性,来获取表单对象
```dart
  GlobalKey<FormState> _formKey = new GlobalKey<FormState>();

  String _name;

  String _password;
```
同时也设置了`_name`,`_password`两个变量来存储用户的输入值,在`TextFormField`组件的`onSaved`方法中,将输入框的值赋值到设定的变量中


我们通过`FloatingActionButton`来触发表单提交事件

```dart
floatingActionButton: new FloatingActionButton(
    onPressed: _forSubmitted,
    child: new Text('提交'),
),
```

在`_forSubmitted`中我们使用key的`currentState`属性来获取表单的实例对象
```
  void _forSubmitted() {
    var _form = _formKey.currentState;

    if (_form.validate()) {
      _form.save();
      print(_name);
      print(_password);
    }
  }
```

对于表单对象来说,其有一些非常实用的方法比如: 
- `reset` 重置表单内容
- `validate`, 调用`TextFormField`的`validator`方法
- `save`, 表单保存

完整示例

```dart
import 'package:flutter/material.dart';

void main() => runApp(new HomePage());

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => new _HomePageState();
}

class _HomePageState extends State<HomePage> {
  GlobalKey<FormState> _formKey = new GlobalKey<FormState>();

  String _name;

  String _password;

  void _forSubmitted() {
    var _form = _formKey.currentState;

    if (_form.validate()) {
      _form.save();
      print(_name);
      print(_password);
    }
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return new MaterialApp(
      title: 'Flutter data',
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text('Flutter Form'),
        ),
        floatingActionButton: new FloatingActionButton(
          onPressed: _forSubmitted,
          child: new Text('提交'),
        ),
        body: new Container(
          padding: const EdgeInsets.all(16.0),
          child: new Form(
            key: _formKey,
            child: new Column(
              children: <Widget>[
                new TextFormField(
                  decoration: new InputDecoration(
                    labelText: 'Your Name',
                  ),
                  onSaved: (val) {
                    _name = val;
                  },
                ),
                new TextFormField(
                  decoration: new InputDecoration(
                    labelText: 'Password',
                  ),
                  obscureText: true,
                  validator: (val) {
                    return val.length < 4 ? "密码长度错误" : null;
                  },
                  onSaved: (val) {
                    _password = val;
                  },
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

#### 参考资料

- [FormState class](https://docs.flutter.io/flutter/widgets/FormState-class.html)

- [Form Class](https://docs.flutter.io/flutter/widgets/Form-class.html)

- [TextField class](https://docs.flutter.io/flutter/material/TextField-class.html)

- [TextFormField](https://docs.flutter.io/flutter/material/TextFormField-class.html)