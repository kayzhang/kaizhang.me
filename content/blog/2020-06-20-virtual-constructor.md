---
title: Virtual Constructor in C++ and Java
date: '2020-06-20'
slug: virtual-constructor
categories:
  - C++
  - Java
tags:
  - 
---

注：Virtual Constructor 只是一个概念或者习惯的叫法，其并非 C++ 或者 Java 的一个语言特性。比如 C++ 是不支持 Virtual Constructor 的，其需要借助其他方法实现类似 Virtual Constructor 的功能需求。

## 1. 为什么需要 Virtual Constructor 这一特性？

应用场景：在一个对象的静态类型已知、动态类型未知的情况下，创建动态类型与其相同的新对象或者对其进行深拷贝。

举例如下：

```cpp
#include <cstdlib>
#include <iostream>

class Expression
{
 public:
  virtual long evaluate() = 0;
  virtual ~Expression() {}
};

class NumExpression : public Expression
{
 private:
  long myNum;

 public:
  explicit NumExpression(long numin) : myNum(numin) {}

  virtual long evaluate() { return myNum; }

  virtual ~NumExpression() {}
};

class PlusExpression : public Expression
{
 private:
  Expression * left;
  Expression * right;

 public:
  PlusExpression(Expression * lin, Expression * rin) : left(lin), right(rin) {}

  virtual long evaluate() { return left->evaluate() + right->evaluate(); }

  virtual ~PlusExpression() {
    delete left;
    delete right;
  }
};

int main(void) {
  Expression * pexp1 = new PlusExpression(new NumExpression(5), new NumExpression(10));

  std::cout << pexp1->evaluate() << std::endl;

  delete pexp1;
  return EXIT_SUCCESS;
}
```

上述代码存在一个问题，那就是其违背了 “Rule of Three”：由于 `PlusExpression` 实现了自定义的 destructor，因此其必须同时实现 copy constructor 和 assignment operator。

在实现这二者时便遇到了一个棘手的问题：`Plusexpression` 类存在两个成员变量 `Expression * left` 与 `Expression * right`，虽然其所指对象的静态类型为 `Expression`，但我们并不知道其动态类型，因此对无法直接调用构造函数对其进行深拷贝。

## 2. 如何解决上述问题呢？

### 2.1 Make Constructor Virtual—-can we do this?

C++ 是不支持 virtual constructor 这一语言特性的，原因参见：

[Why don't we have virtual constructors?](https://www.stroustrup.com/bs_faq2.html#virtual-ctor)

> A virtual call is a mechanism to get work done given partial information. In particular, "virtual" allows us to call a function knowing only an interfaces and not the exact type of the object. To create an object you need complete information. In particular, you need to know the exact type of what you want to create. Consequently, a "call to a constructor" cannot be virtual.

### 2.2 Write a large switch/case statement to copy different subclasses—any drawbacks to this approach?

Copy constructor 可以实现如下：

```cpp
PlusExpression(const PlusExpression & rhs) {
  Expression * leftCopy;
  switch ((rhs.left)->whichType) {
    case TYPE_NUM:
      leftCopy = new NumExpr(*(rhs.left));
      break;
    case TYPE_PLUS:
      leftCopy = new PlusExpr(*(rhs.left));
      break;
      //...
  }
  //same goes for the rightCopy...
}
```

此方法虽然可行，但是其存在很多缺点：

* Requires Expressions to have an additional field whichType.
* What if we add another subclass called MinusExpression?
* We have to implement this switch statement in copy constructor of every operation expression class!!

### 2.3 理想的解决方案：Virtual Factory Methods `create()` 和 `clone()`

虽然 constructor 无法是定义为 virtual，但是我们可以实现 virtual 的成员方法：

* `virtual Expression * create() const;`：对应构造函数，可以针对每一个构造函数重载一个对应的 `create()` 方法。参见：[7 Advance C++ Concepts & Idiom Examples You Should Know](http://www.vishalchovatiya.com/7-advance-cpp-concepts-idiom-examples-you-should-know/)
* `virtual Expression * clone() const;` ：主要用于实现 copy constructor 与 assignment operator。

对于实现 copy constructor 与 assignment operator 只适用 `clone()` 即可。

实现如下：

```cpp
#include <cstdlib>
#include <iostream>

class Expression
{
 public:
  virtual long evaluate() = 0;

  // pure virtual clone() method
  virtual Expression * clone() const = 0;

  virtual ~Expression() {}
};

class NumExpression : public Expression
{
 private:
  long myNum;

 public:
  explicit NumExpression(long numin) : myNum(numin) {}

  virtual long evaluate() { return myNum; }

  // implement clone() method
  virtual Expression * clone() const { return new NumExpression(myNum); }

  virtual ~NumExpression() {}
};

class PlusExpression : public Expression
{
 private:
  Expression * left;
  Expression * right;

 public:
  PlusExpression(Expression * lin, Expression * rin) : left(lin), right(rin) {}

  // Copy Constructor

  virtual long evaluate() { return left->evaluate() + right->evaluate(); }

  /* Note, here clone() and copy constructor are mutually recursive functions.
   * The base case is the copy constructor of NumExpression class.
   */

  // override clone() method
  virtual Expression * clone() const { return new PlusExpression(*this); }

  // copy constructor
  PlusExpression(const PlusExpression & rhs) : left(rhs.left->clone()), right(rhs.right->clone()) {}

  // assignment operator
  PlusExpression & operator=(const PlusExpression & rhs) {
    if (this != &rhs) {
      PlusExpression temp(rhs);
      std::swap(left, temp.left);
      std::swap(right, temp.right);
    }
    return *this;
  }

  virtual ~PlusExpression() {
    delete left;
    delete right;
  }
};

int main(void) {
  /* The file is compiled using:
   * g++ -o test -std=gnu++98 -Wall -Werror -ggdb3 test.cpp
   * And the code is valgrind clean.
   */

  PlusExpression * pexp1 = new PlusExpression(new NumExpression(5), new NumExpression(10));
  std::cout << pexp1->evaluate() << std::endl;

  Expression * pexp2 = new PlusExpression(*pexp1);
  delete pexp1;
  std::cout << pexp2->evaluate() << std::endl;

  Expression * pexp3 = new PlusExpression(new NumExpression(5), new NumExpression(10));
  *pexp3 = *pexp2;
  delete pexp2;
  std::cout << pexp3->evaluate() << std::endl;

  delete pexp3;

  return EXIT_SUCCESS;
}
```

### 3. Java 中的实现方式

* 对应于 C++ 中的 `clone` 方法，Java 中 `Object` 类中定义了 [`clone`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#clone())。
* Java 不需要类似 C++ 中 `create` 方法的实现方式，因为其可以直接通过 Reflection 机制获得需要的 constructor。具体为：Java 中通过 `java.lang.reflect.Constructor` 中的 [`newInstance`] 方法实现创建新对象，其中 `Constructor` 类的对象可以由 `java.lang.Class` 类中的 [`getConstructor`](public Constructor<T> getConstructor​(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException) 方法获得。详见 <<Core Java Volumn I>> 中的 5.7.1 小结。

示例如下：

```java
import java.util.Random;
import java.lang.reflect.Constructor;

public class Test {

    public static void main(String []args) throws
        ClassNotFoundException,
        NoSuchMethodException,
        InstantiationException,
        IllegalAccessException,
        java.lang.reflect.InvocationTargetException {
        
        /* There are 3 common ways to get the Class instance */
        
        /* Example 1: use Class.forName static method */
        String className = "java.util.Random";
        Class<?> cl1 = Class.forName(className);
        Constructor<?> ct1 = cl1.getConstructor(); // get the default constructor
        Random rand1 = (Random) ct1.newInstance();
        // output: java.util.Random
        System.out.println(rand1.getClass().getName());
        
        /* Example 2: use getClass method inhirited from Object class */
        Random temp = new Random();
        Class<?> cl2 = temp.getClass(); // get the default constructor
        Constructor<?> ct2 = cl2.getConstructor();
        Random rand2 = (Random) ct2.newInstance();
        // output: java.util.Random
        System.out.println(rand2.getClass().getName());
        
        /* Example 3: use type.class shorthand */
        /* In this way, we don't need to use ? */
        Class<Random> cl3 = Random.class;
        Constructor<Random> ct3 = cl3.getConstructor();
        Random rand3 = ct3.newInstance();
        // output: java.util.Random
        System.out.println(rand3.getClass().getName());
    }
}
```


