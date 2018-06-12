---
title: "Flutter 数据获取 网络"
date: 2018-06-12T16:41:22+08:00
draft: true
---

> 该文章翻译自:[Fetch data from the internet](https://flutter.io/cookbook/networking/fetch-data/)

### 概述
从网络中获取数据一般需要以下几个步骤:

1. 添加网络请求库,可以在如下的[http](https://pub.dartlang.org/packages/http)包中选择

1. 使用添加的http包发起网络请求

1. 将网络请求转换为常规的Dart对象

1. 显示数据


### 详情

#### 添加http网络请求库
[http](https://pub.dartlang.org/packages/http)库提供了简单的方式从网络获取数据
为了添加库到项目中,需要在项目根目录的`pubspec.yaml`中添加依赖说明

```dart
dependencies:
  http: <latest_version>
```

然后执行
```dart
pub get
```

#### 发起请求

在如下的实例中使用`http.get`方法从[jsonplaceholder](https://jsonplaceholder.typicode.com/)获取数据

```dart
Future<http.Response>fetchPost(){
    return http.get('https://jsonplaceholder.typicode.com/posts/1');
}
```
`http`返回的是一个`Future`,其包含一个`Response`

+ `Future`是`Dart`的一个核心类,设计用于异步操作, 它常常用来表示一个可能的值或者是错误

+ 当请求成功时,`http.Response`包含获取到的数据

#### 转换
尽管获取数据非常简单,但是获取到的数据却不能直接使用,因此我们需要将其装换成`Dart`对象

##### 创建`Post`类
首先我们需要创建一个`Post`类,用来作为通过网络请求获取到的数据的载体,该类包含一个特殊的初始化方法用来从`json`数据中创建`Post`对象实例

```dart
class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({this.userId, this.id, this.title, this.body});

  factory Post.fromJson(Map<String, dynamic> json) {
      return new Post(
        userId: json['userId'],
        id: json['id'],
        title: json['title'],
        body: json['body']);
      );
  }
}
```
##### 将`http.Response`转换为`Post`

```dart
Future<Post> fetchPost()async {
  final response =
      await http.get("https://jsonplaceholder.typicode.com/posts/1");

  final responseJson = json.decode(response.body);
  return new Post.fromJson(responseJson);
}
```

在这里我们修改`fetchPost`方法,将其返回值由`http.Response`变成`Post`

##### 显示数据
为了显示数据到屏幕上,我们可以使用[FutureBuilder](https://docs.flutter.io/flutter/widgets/FutureBuilder-class.html)组件,该组件与`Future`以及异步数据能够非常好的工作

该组件需要两个参数
1. Future参数,我们将使用上面的`fetchPost`方法

1. Builder 函数如何显示获取到的数据

```dart
  @override
  Widget build(BuildContext context) {
    return new Container(
      child: FutureBuilder(
        future: fetchPost(),
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return new Text(snapshot.data.title);
          } else if (snapshot.hasError) {
            return new Text("${snapshot.error}");
          }
          return new CircularProgressIndicator();
        },
      ),
    );
```


#### 扩展

- [http](https://pub.dartlang.org/packages/http#-readme-tab-)

- [http Library](https://pub.dartlang.org/documentation/http/0.11.3+16/http/http-library.html)

- [JSON and serialization](https://flutter.io/json/)