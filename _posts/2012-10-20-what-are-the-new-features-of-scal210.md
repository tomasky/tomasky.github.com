---
layout: post
title: "scal2.10 中有哪些新东西"
post_type: ""
description: ""
category: 
tags: []
---
{% include JB/setup %}

##值类
允许用户继承顶级类AnyVal,如代码:

 ```scala
 class C(val u:U) extends AnyVal{
   def m1(ps1) = ...
   def m2(psn)= ...
 }
 ```
 这里的C叫做值类，它必须满足一下条件：   

1. C必须只有一个public 公共可访问的val 不变参数,如C类的u参数，这里的类型U叫做`相关类型`.  
1. C可以没有@specialized 类型参数.  
1. C的相关类型可以不是值类.  
1. C可以没有`equals` 和`hashCode`方法.  
1. C可以没有其他的构涵.  
1. C必须是顶级类或静态可访问对象的成员.  
1. C必须是`ephemeral`. 

什么是`ephemeral`?

1. 可以除了u:U参数，没有其他的域.  
1. 可以没有object定义.  
1. 可以没有初始化语句快. 


如果C的相关类型是D类型的实例,C可以直接拆厢到D. 

下面的隐式转换假设适用于值类:

1. 值类是final的，所以不能被继承.
1. 有隐式方法 'equals','hashCode'(可继承自AnyVal):

```scala
def equals(other: Any) = other match {
case that: C => this.u == that.u
case _ => false
}
def hashCode = u.hashCode
```
