---
title: Protected Modifier in Java
date: '2020-06-15'
slug: protected-modifier-in-java
categories:
  - Java
tags:
  - Java
---

在 Java 中，有四种不同的的访问权限修饰符。

1. Accessible in the class only (`private`).
2. Accessible in the package--the default. No modifiers are needed.
3. Accessible in the package and all subclasses (`protected`).
4. Accessible by the world (`public`).

其中，不同 package 内的子类访问父类的 `protected` 成员时的规则比较复杂，下面简要说明。

## 1. 当父类中的 `protected` 成员并非 `static` 时

父类定义如下:

```java
package parent;

public class Parent {
    protected String str = "A protected instance field";

    protected void printMsg() {
        System.out.println("A protected instance method");
    }
}
```

### 1.1 不同包下，子类不能通过父类引用访问父类的 `protectd` 成员

注意此为编译时的限制，与引用的动态类型无关。

子类定义如下:

```java
package child;

import parent.Parent;

public class Child extends Parent {
    public static void main(String[] args) {
        Parent p1 = new Parent();
        String str1 = p1.str; // Compile-time error
        p1.printMsg();        // Compile-time error

        Parent p2 = new Child();
        String str2 = p2.str; // Compile-time error
        p2.printMsg();        // Compile-time error
    }
}
```

编译时错误如下:

```
~/dev/java/ProtectedTest $ javac child/Child.java 
child/Child.java:8: error: str has protected access in Parent
        String str1 = p1.str;
                        ^
child/Child.java:9: error: printMsg() has protected access in Parent
        p1.printMsg();
          ^
child/Child.java:12: error: str has protected access in Parent
        String str2 = p2.str;
                        ^
child/Child.java:13: error: printMsg() has protected access in Parent
        p2.printMsg();
          ^
4 errors
~/dev/java/ProtectedTest $ 
```

## 1.2 不同包下，子类可以通过该子类(或其子类)的引用访问父类的 `protectd` 成员

注意此为编译时的限制，与引用的动态类型无关。

子类定义如下:

```java
package child;

import parent.Parent;

public class Child extends Parent {
    public static void main(String[] args) {
        Child c1 = new Child();
        String str1 = c1.str; // OK
        c1.printMsg();        // OK

        Child c2 = new GrandChild();
        String str2 = c2.str; // OK
        c2.printMsg();        // OK

        GrandChild g1 = new GrandChild();
        String str3 = g1.str;
        g1.printMsg();
    }
}
```

## 1.3 不同包下，子类可以通过 `super` 关键字访问父类的 `protectd` 成员

子类定义如下:

```java
package child;

import parent.Parent;

public class Child extends Parent {
    @Override
    protected void printMsg() {
        System.out.println(
            "Overridden version of the protected instance method");
    }

    private void show() {
        System.out.println(super.str);
        super.printMsg();
        printMsg();
    }

    public static void main(String[] args) {
        Child c1 = new Child();
        c1.show();
    }
}
```

输出如下:

```
~/dev/java/ProtectedTest $ java child/Child
A protected instance field
A protected instance method
Overridden version of the protected instance method
~/dev/java/ProtectedTest $
```

## 1.4 不同包下，子类 I 不能通过父类的其他子类 II 的引用访问父类的 `protected` 成员

注意此为编译时的限制，与引用的动态类型无关。另外，此处的其他子类 II 不包括子类 I 的子类。

子类定义如下:

```java
package child;

import parent.Parent;

public class Child extends Parent {
    public static void main(String[] args) {
        Child2 c2 = new Child2();
        String str = c2.str; // Compile-time error
        c2.printMsg();       // Compile-time error
    }
}

class Child2 extends Parent {}
```

编译时错误如下:

```
~/dev/java/ProtectedTest $ javac child/Child.java 
child/Child.java:8: error: not a statement
        c2.str;
          ^
1 error
~/dev/java/ProtectedTest $ javac child/Child.java 
child/Child.java:8: error: str has protected access in Parent
        String str = c2.str;
                       ^
child/Child.java:9: error: printMsg() has protected access in Parent
        c2.printMsg();
          ^
2 errors
~/dev/java/ProtectedTest $
```

## 2. 当父类中的 `protected` 成员为 `static` 时

对于 `protected` 的静态变量，在所有子类中(无论子类是否与父类在同一包内)均可直接访问。