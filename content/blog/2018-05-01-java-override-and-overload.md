---
title: Java Override and Overload
subtitle:
date: '2018-05-01'
slug: java-override-and-overload
categories:
  - Java
tags:
  - Java
---

Java 中子类方法 override（重写 / 覆盖）父类方法必须遵循“两同两小一大”的原则：

* 两同：方法名相同，形参列表完全相同；
* 两小：子类返回值应与父类返回值类型相同或更小（即子类返回值是父类返回值的子类），子类方法抛出的异常类型应与父类相同或更小；
* 一大：子类方法的访问权限应与父类相同或更大（即限制更低）。

注意：在这里边有一点需要特别注意，即**子类重写方法的形参列表必须与父类完全相同**。

* 即使只将子类方法参数类型改为父类方法参数类型的子类，也将不再是重写，针对重写的动态方法查找将失效。若对一个指向子类实例的父类变量调用该函数，则会调用父类中的同名函数。若父类方法中有多个重载的同名函数，则基于实参类型根据向上转型最近匹配的原则确定具体调用父类的哪一个函数。
* 重载是相对于同一个类中的同名函数而言的，上述的子类函数不是父类函数的重写，同时也不是父类函数的重载，对父类引用变量调用该函数只会在父类函数域内的重载函数之间进行最近匹配；同时可以理解为该子类函数在子类函数域内是子类继承得到的父类函数的重载，在对子类引用变量调用该函数时，会同时对该子类函数和继承得到的父类函数同时考虑就近匹配。

比较拗口，以下以具体的例子来进行说明：

父类定义如下：

```java
public class Animal {
    public void run(Object obj) {
        System.out.println("Animal.run(Object obj)");
    }

    public void run(String str) {
        System.out.println("Animal.run(String str)");
    }
}
```

子类定义如下：

```java
public class Dog extends Animal {
    public void run(Dog dog) {
        System.out.println("Dog.run(Dog dog)");
    }
}
```

主函数如下：

```java
public class testOverride {
    public static void main(String[] args) {
        Object obj = new Object();
        Animal ani = new Dog();
        Dog dog = new Dog();

        // Animal.run(Object obj), not Dog.run(Dog dog)
        // Animal.run(Object obj) 并未被 Dog.run(Dog dog) 重写，动态方法查找失效，直接调用父类方法，且遵循向上转型就近匹配原则
        ani.run(dog);

        // Animal.run(String str), not Animal.run(Object obj)
        // 向上转型就近匹配原则
        ani.run("string");

        // Animal.run(Object obj)
        dog.run(obj);

        // Animal.run(String str)
        dog.run("string");

        // Dog.run(Dog dog)
        dog.run(dog);
    }
}
```

1. ani.run(dog): ani 是父类引用变量，而子类未对父类的 run 函数进行重写（参数类型虽是子类但并不完全相同），因此动态方法查找失效（即编译器认为子类没有对这一方法进行更改，那么直接调用父类方法），调用父类方法，而父类中同名方法有 2 个，此时根据向上转型最近匹配原则，执行父类函数 Animal.run(Object obj)。
2. ani.run("string"): 同上，调用父类方法，根据向上转型最近匹配原则，执行父类函数 Animal.run(String str)。
3. dog.run(obj): dog 是子类引用变量，直接调用子类函数域内的函数，此时同名方法有3 个，即从父类那里继承的 2 个，和子类中增加的 1 个，这三个函数在子类函数域内是重载函数，根据向上转型最近匹配原则进行选择，执行 Animal.run(Object obj)。
4. dog.run("string"): 同上，执行 Animal.run(String str)。
5. dog.run(dog): 同上，执行 Dog.run(Dog dog)。

运行结果如下：

    Animal.run(Object obj)
    Animal.run(String str)
    Animal.run(Object obj)
    Animal.run(String str)
    Dog.run(Dog dog)
