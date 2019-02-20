---
layout: post
title: "Java基础-语法"
subtitle: '介绍java基础知识 static final transient foreach原理...'
author: "xiaojiejava"
header-style: text
tags:
  - Java
  - static
  - final
  - transient
  - foreach实现原理
---
### static

1. static可以用于修饰变量和方法，修饰变量表示所有对象实例共享，修饰方法表示方法属于类方法（常用于util类方法）
2. static可以用于修饰类，表示静态内部类
3. static可以用于静态代码块，在类构造器调用前调用
4. static可以用于静态导包，避免代码臃肿

### final

1. final可以用于修饰类和方法，表示类不可被继承和方法不可被继承
2. final可以用于修饰变量，对于基本类型一旦赋值变量不能修改，对于引用类型一旦赋值变量不能重新赋值，但是引用的对象内部状态可以修改

### transient

当对java对象序列化时，不想序列化对象的某些字段，则这些字段用transient修饰可以阻止序列化，从而节省空间或屏蔽敏感信息

### foreach实现原理

java循环的几种方式:

1. 普通for循环遍历 for(int i=0; i<list.size(); i++){}
2. 迭代器 Iterator iterator = list.iterator();while(iterator.hasNext()){}
3. for each，增强的for循环 for(Integer i : list){}
4. lambda表达式

对于for each而言，是java提供的语法糖，实际在编译的时候会将其变为迭代器(Iterator)模式

### Object类中的方法以及每个方法的作用

native Class<?> getClass()
	
返回当前对象的运行时类，这个返回对象是被static synchronized锁住的对象

native int hashcode()

这个对象的int值表示，应用于Map、Set集合，再放入到集合之前先比较对象的hashCode是否相等.--equals相等hashCode也必须相等

boolean equals(Object obj)

比较两个对象是否相等，默认实现是通过比较内存地址是否相等

Object clone()

对象复制，用的较少，一般自己写util实现对象复制

String toString()

打印对象时默认调用这个方法，默认返回class和对象hashCode信息

void notify()

唤醒一个等待当前对象锁的线程

void notifyAll()

唤醒所有等待当前对象锁的线程

void wait(long timeout)

释放锁，并进入对象等待池

void wait(long timeout, int nanos)

释放锁，并进入对象等待池

void wait()

释放锁，并进入对象等待池

void finalize()

当对象被gc之前，如果覆盖了这个方法，则这个方法会被调用。jvm不保证调用，不可靠，基本不用。仅限于保底+释放非java资源

