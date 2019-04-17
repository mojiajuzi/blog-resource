---
title: "Golang-Error"
description: "错误处理是编程时不可避免遇到的问题"
date: 2019-04-05T09:10:29+08:00
draft: true
---

`error`类型是一个内置的接口
```go
type error interface {
	Error() string
}
```
依据接口的原则，凡是实现了接口所定义方法的类型，都可以算作是实现了该接口，因此下面是几种在程序逻辑中常用的处理错误的方式

##### 使用`New`方法创建
在Go的标准包`errors`中提供了创建错误的方法,那就是使用`New`方法,它的实现方式为
```go
  func New(text string) error {
      return &errorString{text}
  }

  type errorString struct {
      s string
  }

  func (e *errorString) Error() string {
      return e.s
  }
```
因此就可以创建一个错误`errors.New("error info")`
```go
type Student struct {
	name string
	age  int
}

func (s Student) getFullStudentInfo() (string, error) {
	if s.age <= 0 {
		return strconv.Itoa(s.age), errors.New("the age can not lt zero")
	}

	return fmt.Sprintf("the name %s, age is:%d", s.name, s.age), nil
}
```
以上创建了一个学生结构体，该结构体包含简单的名称和年龄两个字段，然后有一个`getFullStudentInfo`方法获取学生的详细信息
在该方法中，对年龄进行了简单的判断，如果年龄小于0，则抛出一个错误

##### 使用`Errorf`创建
我们可以使用`Errorf`抛出更纤细的错误处理信息,这是因为`Errorf`方法对`New`方法进行封装
```go
func Errorf(format string, a ...interface{}) error {
	return errors.New(Sprintf(format, a...))
}
```
因此将上面的例子改写为
```go
func (s Student) getFullStudentInfo() (string, error) {
	if s.age <= 0 {
		return strconv.Itoa(s.age), fmt.Errorf("the student name:%s,the age is error, age is:%d", s.name, s.age)
	}

	return fmt.Sprintf("the name %s, age is:%d", s.name, s.age), nil
}
```

##### 使用结构体的字段或者方法，亦可以创建错误
```go
type Student struct {
	name string
	age  int
}

func (s Student) getFullStudentInfo() (string, error) {
	if s.age <= 0 {
		return strconv.Itoa(s.age), s
	}

	return fmt.Sprintf("the name %s, age is:%d", s.name, s.age), nil
}

func (s Student) Error() string {
	return fmt.Sprintf("the student name:%s,the age is error, age is:%d", s.name, s.age)
}
```
以上使用结构体实现了`error`方法，因此可以实现错误的抛出

#### 自定义结构体错误类型
```go
func (s *StudentError) Error() string {
	return s.err
}

func (s *StudentError) ageLt() bool {
	return s.age <= 0
}

func (s *StudentError) ageGt() bool {
	return s.age >= 100
}
```
以上，定义了一个结构体类型，那么就可以使用如下:
```go
if err != "" {
	return strconv.Itoa(s.age), &StudentError{age: s.age, name: s.name}
}
```


#### 其他

1. 在一个函数或者方法含有多个返回值的话，如果返回一个错误值，那么该错误值应该作为以后一个返回值的参数返回

1. 在Go常用的判断错误的方式是将返回的错误值与`nil`进行比对，如果为`nil`则表示没有错误，反之则表示有错误发生

