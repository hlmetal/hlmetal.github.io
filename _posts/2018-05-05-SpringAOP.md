---
layout: post
title:  "反射机制及SpingAOP"
date:   2018-05-05 08:00:30 +0200
categories: java
---

反射机制是Java语言中的一个特性，它允许程序在运行时进行自我检查，同时也允许其内部的成员进行操作。

## 反射机制
反射机制是java底层执行机制，几乎所有框架底层都使用反射技术。具体而言，是指在程序运行期间对任意一个类，都能知道其所有属性和方法，并能够对属性和方法进行操作。通过反射机制，可以在编译期间不需要直到类的具体名称，而在运行期间动态获取类信息，创建对象，调用方法等。
### 反射机制功能
* 动态执行功能
    - 动态加载类
    - 动态创建对象
    - 动态访问属性，执行方法
    - 动态解析注解等\

### 基本概念
* 静态执行
编译以后就确定了执行顺序：先创建对象，然后执行方法，在运行期间按照编译结果顺序执行。
* 动态执行
在运行期间才确定执行次序，包括运行期间决定创建哪个对象，运行期间决定调用哪个方法。
* 动态加载类
在运行期间动态确定加载那个类，只加载一次Class cls=class.froName()
<img src= "/assets/files/dynamic_class_loading.png" alt="加载错误" title="动态加载类"/>

* 动态创建对象
`Object obj=cls.newInstance();`
调用cls引用的类信息中的无参构造器，创建一个对象。cls对应的类中包含无参构造器，若没有则抛出异常(反射API可以调用有参构造器)
* 动态获取类中的方法信息
    * 利用反射API，获取一个类中声明的方法信息
    * 利用反射调用该方法
    执行method对应的方法，对象是包含方法的对象，参数是执行方法时传递的参数，返回值(value)是方法执行以后的返回值
```java
Object value=method.invoke(对象，方法参数)
invoke 调用
method 方法
```
* 利用反射执行私有方法
```java
//打开方法访问限制
method.setAccessible(true);
```
* 利用反射解析注解
```java
Object obj=method.getAnnotation(注解类型);
//检查method上是否包含指定的注解，若返回null则不包含
Object obj=cls.getAnnotation(注解类型);
//检查属性上是否包含指定的注解，若返回null则不包含
Object obj=field.getAnnotation(注解类型);
```
## SpringAOP