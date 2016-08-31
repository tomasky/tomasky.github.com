---
layout: post
title: "scala2.10 中有哪些新东西"
description: ""
category: scala 
tags: [scala]
---
{% include JB/setup %}
## scala2.10 新特性
----

### 值类(value class)

允许用户继承顶级类AnyVal,如代码:

```scala
 class C(val u:U) extends AnyVal{
   def m1(ps1) = ...
   def m2(psn)= ...
 }
```
 
这里的C叫做值类，它必须满足以下条件：   

1. C必须只有一个public 公共可访问的val 不变参数,如C类的u参数，这里的类型U叫做`相关类型`.  
1. C没有@specialized 类型参数.  
1. C的相关类型不是值类.  
1. C不可覆盖`equals` 和`hashCode`方法.  
1. C没有其他的构函.  
1. C必须是顶级类或静态可访问对象的成员.  
1. C必须是`ephemeral`. 

什么是`ephemeral`(短暂)?

1. 除了u:U参数，没有其他的域.  
1. 没有object定义.  
1. 没有初始化语句块. 


如果C的相关类型是D类型的实例,C可以直接拆厢到D.间接拆厢是直接拆厢的过渡闭包，值类不能间接或直接拆厢到自己. 

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
-----


### 万能特质(universal trait)


   由于scala的规制，值类不能继承trait，因为所有的trait继承自AnyRef，除非是`万能特质（universal trait)`. `universal trait`必须明确继承Any类，比如:

```scala
 trait Equals[T] extends Any { … }
 trait Ordered[T] extends Equals[T] { … } 
```

   这里Ordered需要继承Any类，才是万能特质
   trait Ordered[T] extends Any with Equals[T]{}  
   万能特质也是`ephemeral` 

<!--break-->

### 值类扩充

> 值类可以被扩充，如下面的Meter类：

```scala
class Meter(val underlying: Double) extends AnyVal with Printable {
  def plus (other: Meter): Meter = 
    new Meter(this.underlying + other.underlying)
  def divide (factor: Double): Meter = new Meter(this.underlying / factor)
  def less (other: Meter): Boolean = this.underlying < other.underlying
  override def toString: String = underlying.toString + “m”
}
trait Printable extends Any { def print: Unit = Console.print(this) }
```

> 简单来说，我们假设所有的扩充步骤是针对于可擦除类型来做的.

1. 抽取方法(extracting method)   
   让值类的extractable method成为在类(不能继承)中可以自己表示的方法 ,并且在方法体中不含super调super调用用.  
  
   对于每个extractable method m，我们在这个类的半生对象(如果没有，自动新建)中建另外一个叫extension$m的方法,这个方法如下:
   
   对于 class C(val u: U) extends AnyVal{ def m(params):R=body }  
   m方法在伴生对象中被扩充为 ：   

>  def extension$m($this: C, params): R = body’ (经过实验，实际上object C中生成的是m$extension)  
   
   然后类C中的m方法会编译为：  
   
>  def m(params):R = C.m$extension(this,params)  
   
   重载的方法有其他的名字来区分(divide$extension1(this,factor))，合成的equals和hashCode方法也会加入到类中来.

3. 重选调用 
```scala
 val x, y: Meter  
	x.plus(y)  
	x.toString  
// 将被重写为:  
 Meter.plus$extension(x,y)  
  Meter.toString$extension(x)
```
 
4. 擦除 

   下面，我们为类型C介绍一个新的类型叫C$unboxed(将会在第4步被除去).这个新类型没有成员，完全独立于scala类层级之外.他是非其他类型的子类，是Nothing的父类. 

4. 清除C$unboxed
   替换C$unboxed为C的相关类型


### 动态类型

```scala
//模块化语言
import language.dynamics
// applyDynamicNamed example
object JSON extends Dynamic {
  def applyDynamicNamed(name: String)(args: (String, Any)*) {
    println(s"""Creating a $name, for:\n "${args.head._1}": "${args.head._2}" """)
  }
}
 
JSON.node(nickname = "ktoso")

//select

object Has extends Dynamic {
  def selectDynamic(name: String): String = s"I have $name!"
}
println(Has.wo)


// updateDynamic example
object MagicBox extends Dynamic {
  private var box = mutable.Map[String, Any]()
 
  def updateDynamic(name: String)(value: Any) { box(name) = value }
  def selectDynamic(name: String) = box(name)
}
MagicBox.sh ="jack" 
```

### 隐式导入类型

```scala
implicit class RichInt(n: Int) extends Ordered[Int] {
  def min(m: Int): Int = if (n <= m) n else m
}
//转化为如下：(scala只有值和类型两个命名空间,方法和类型名可以重复)
class RichInt(n: Int) extends Ordered[Int] {
  def min(m: Int): Int = if (n <= m) n else m
}
implicit final def RichInt(n: Int): RichInt = new RichInt(n)
```

### 串注入

```scala
//串类型s
val ss = s"this a num${2+2}" 

//万能类型f,串中可以出现所有匹配的number  
println(f"$name%s is $height%2.2f meters tall")

//原生类型raw,让转换符号无效  
s"a/nb"  >a
 b
raw"a/nb" > a/nb

//自定义类型 id"$name,$age"  
//1. ss"a$name" 转化=> new StringContext("a","").ss(name)
//2.自定义implicit
implicit class T(val sc:StringContext) {
 def ss(args:Any*) = _
}
```
