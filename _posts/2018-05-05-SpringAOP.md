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
<img src= "/assets/files/dynamic_class_loading.jpg" alt="加载错误" title="动态加载类"/>

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
### 创建代理对象
由Spring代理策略生成的对象
1. 创建Bean实例都是从getBean()方法开始的, 在实例创建之后,Spring容器将根据AOP的配置去匹配目标类的类名, 看目标类的类名是否满足切面规则
2. 如果满足满足切面规则, 就会调用ProxyFactory创建代理Bean并缓存到IoC容器中
3. 根据目标对象的自动选择不同的代理策略
* 目标类实现了接口, Spring会默认选择JDK Proxy
* 目标类没有实现接口, Spring会默认选择Cglib Proxy

### 拦截目标对象
1. 当用户调用目标对象的某个方法时, 将会被一个叫做AopProxy的对象拦截, Spring将所有的调用策略封装到了这个对象中, 它默认实现了InvocationHandler接口,即调用代理对象的外层拦截器
2. 在这个接口的invoke()方法中, 会触发MethodInvocation的proceed()方法. 在这个方法中会按顺序执行符合所有AOP拦截规则的拦截器链

### 调用代理对象阶段
1. Spring AOP拦截器链中的每个元素被命名为MethodInterceptor, 其实就是切面配置中的Advice通知
2. 这个回调通知可以简单地理解为是新生成的代理Bean中的方法, 即被织入的代码片段, 这些代码在这个阶段执行

### 调用目标对象阶段
MethodInterceptor接口也有一个invoke()方法,在该方法中会触发对目标对象方法的调用