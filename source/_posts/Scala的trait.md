---
title: Scala的trait
date: 2018-02-07 16:27:12
tags: scala
---



今天看一下scala的**trait**，用法不讲，就看一下编译再反编译的源码

定义一个trait
```scala
trait TraitDemo {
  def say(): Unit
}
```
使用`javac`命令编译这个文件
```
# scalac TraitDemo.scala
```
得到一个`TraitDemo.clss`，我们再使用`javap`命令反编译这个class文件看一下
```
# javap TraitDemo.class

Compiled from "TraitDemo.scala"
public interface com.study.TraitDemo {
  public abstract void say();
}
```
一目了然，trait类型经过编译后变成了接口,而say()被编译成为了abstract方法。
如果我们把`def say(): Unit`方法实现，改为`say(): Unit = {}`，看反编译之后的代码
```
Compiled from "TraitDemo.scala"
public abstract class com.study.TraitDemo$class {
  public static void say(com.study.TraitDemo);
  public static void $init$(com.study.TraitDemo);
}
```
这时候trait就变成了一个抽象类，而实现的方法变成了`static`级别的。

看到上面的编译结果，我们就可以想到scala中，定义不同的trait，实现不同的功能，当我们需要某一个功能的时候可以直接with进来，比如成为便捷的工具类。