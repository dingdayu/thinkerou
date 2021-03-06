---
title: 设计模式之单例模式
description: ""
tags: [
  "设计模式",
]
categories: [
  "设计模式",
]
author: "thinkerou"
date: 2017-02-26
image: 'stuck.jpg'
---


## 1. 目的

保证一个类仅有一个实例，并提供一个访问它的全局访问点。

## 2. 动机

如何才能保证一个类只有一个实例，并且这个实例易于被访问？

一个全局变量使得一个对象可以被访问，但它不能防止实例化多个对象。

> 更好的办法是：让类自身负责保存它的唯一实例。

这个类可以保证没有其它实例可以被创建，并且它可以提供一个访问该实例的方法，这就是单例（Singleton）模式。


## 3. 适用性

单例模式适用于如下情况：

- 当类只能有一个实例，且客户可以从一个众所周知的访问点访问它时；
- 当这个唯一实例应该是通过子类化扩展的，且客户应该无需更改代码就能使用一个扩展的实例时。

## 4. 参与者

定义一个 Instance 操作，允许客户访问它的唯一实例，Instance 是 C++ 中的一个**静态成员函数**；负责创建它自己的唯一实例。

## 5. 协作

客户只能通过 Singleton 的 Instance 操作访问一个 Singleton 的实例。

## 6. 效果

- 对唯一实例的受控访问
  - 因为 Singleton 类封装它的唯一实例，所以它可以严格的控制客户怎样以及何时访问它。
 
- 缩小名空间
  - 单例模式是对全局变量的改进，避免了哪些存储唯一实例的全局变量污染名空间。

- 允许对操作和表示的精化
  - Singleton 类可以有子类，且用这个扩展类的实例来配置一个应用是很容易的。

## 7. 实现

单例模式使得这个唯一实例是类的一般实例，但该类被写成只有一个实例能被创建，为了做到这的常用方法是：**将创建这个实例的操作隐藏在一个类操作（即一个静态成员函数或是一个类方法）后面， 由它保证只有一个实例被创建**。这个操作可以访问保存唯一实例的变量，而且它可以保证这个变量在返回值之前用这个唯一实例初始化。这种方法保证了单例在它首次使用前被创建和使用。

在 C++ 中可以用 Singleton 类的静态成员函数 Instance 来定义这个类操作， Singleton 还定义一个静态成员变量 _instance，它包含一个指向它的唯一实例的指针。

Singleton 类定义：

```
    class Singleton {
        public:
            static Singleton* Instance();
        protected:
            Singleton();
        private:
            static Singleton* _instance;
    };
```
  
Singleton 类实现：

```
    Singleton* Singleton::_instance = 0;
    
    Singleton* Singleton::Instance() {
        if(_instance == 0) {
            _instance = new Singleton;
        }
        return _instance;
    }
```

针对代码的几点说明：

- 客户仅通过 Instance 成员函数访问这个单件；
- 变量 _instance 初始化为 0，而静态成员函数 Instance 返回该变量值，如果其实为 0 则用唯一实例初始化它；
- Instance 使用 lazy 方式初始化，它的返回值直到被第一次访问时才被创建和保存。

**注意：**构造函数是保护型的，如果直接实例化 Singleton 的客户将会得到编译时错误信息，这也就保证了仅有一个实例可以被创建。

因为变量 _instance 是一个指向 Singleton 对象的指针， Instance 成员函数可以将一个指向 Singleton 的子类的指针赋给这个变量。

## 8. 双检测锁模式

实现线程安全的 Instance 一种方式是：**每次判断是否为 NULL 前加锁**，但加锁是很慢的。

事实上，只有第一次实例创建时才需要加锁，即双检测锁：

```
    Singleton* Singleton::Instance() {
        if(_instance == 0) {
            Lock lock;
            if(_instance == 0) {
                _instance = new Singleton;
            }
        }
        return _instance;
    }
```

在 C++11 中关于双检测锁可以阅读[《Double-Checked Locking is Fixed In C++11》](http://preshing.com/20130930/double-checked-locking-is-fixed-in-cpp11/)
