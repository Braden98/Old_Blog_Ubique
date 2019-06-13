---
layout:     post
title:      "用 Optional 取代 null 的思考"
subtitle:   "一个避免空指针异常的好办法"
date:       2019-06-13 22:33:14
author:     "Ubik"
header-img: "img/post-bg-universe.jpg"
catalog: true
comments: true
tags:
    - Java
    - 异常
    - 重构
---

# 空指针异常
空指针异常 NullPointerExpection 是在 Java 开发过程中常见的异常，当我们访问数组中超过其长度-1的 Index 时，或者对 null 进行`.`运算时都会引发这个异常。一般来说，可以通过防御式检查`if(Object != null`来减少之。但很明显，这种方式牺牲了可读性，而且因为逐层潜嵌套，降低了可扩展性。而为了避免逐层嵌套使用多个返回值，那么可维护性就降低了。
实际上，空指针及对它的预防都有很多问题。
- 它代表的是在静态类型语言中以一种错误的方式对缺失变量值的建模，本身没有任何意义。
- 它会使代码膨胀

- Java一直试图避免让程序员意识到指针的存在，唯一的例外是：null指针
- null并不属于任何类型， 这意味着它可以被赋值给任意引用类型的变量。这会导致问题，原因是当这个变量被传递到系统中的另一个部分后，你将无法获知这个null变量最初的赋值到底是什么类型。


在 Java8 中提供了 Option 类来解决这个问题。————当你知道类中某个属性可能为 null 时，采用  `Optional<class>` 来代替 `class`.这样，从静态代码的角度，我们至少可以知道哪些值可为 null，哪些不可，方便我们排查错误。

官方文档描述如下
> A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.


那么，Optional 和 null 到底有什么区别呢？实际上，变量存在时， Optional类只是对类简单封装。变量不存在时， 缺失的值会被建模成一个“空”的Optional对象，由方法 `Optional.empty()` 返回。`Optional.empty()` 方法是一个静态工厂方法，它返回 Optional 类的特定单一实例。你可能还有疑问，null 引用和 `Optional.empty()` 有什么本?的区别吗？从语义上， 你可以把它们当作一回事儿， 但是实际中它们之间的差别非常大 ： 如果你尝试解引用一个 null ， 一 定 会 触发 NullPointerException ，不过使用 `Optional.empty()` 就完全没事儿，它是 Optional 类的一个有效对象，多种场景都能调用，非常有用。

## 创建 Optional 对象
1. 声明一个空的 Optional
`Optional<Car> optCar = Optional.empty();`

2. 依据一个非空值创建 Optional
`Optional<Car> optCar = Optional.of(car);`

3. 创建可接受null的Optional

    `Optional<Car> optCar = Optional.ofNullable(car);`

## 使用 map 从 Optional 对象中提取和转换值
```Java
Optional<String> name =
optInsurance.map(Insurance::getName;
```
## 使用 flatMap 链接 Optional 对象
链式调用 map() 根本无法通过编译，原因是 Optional 发生了嵌套，可以使用 flatMap 完成.
```Java
person.flatMap(Person::getCar)  
.flatMap(Car::getInsurance)  
.map(Insurance::getName) 
.orElse("Unknown");
```
## 无法序列化的 Optional
Java??的架构师 Brian Goetz曾 经非常明确地??过，Optional的设计初衷仅仅是要支持能返回Optional对象的语法，所以它也并未实现Serializable接口，如果你一定要实现序列化的模型，可以这样做
```Java
public class Person {    
    private Car car;  
    public Optional<Car> getCarAsOptional() {        
        return Optional.ofNullable(car);  
  } 
}
```