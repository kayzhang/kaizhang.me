---
title: Java - Iterator and Iterable interface
subtitle: Iterable 与 Iterable 的区别与合作
date: '2018-05-08'
slug: java-iterator-and-iterable
categories:
  - Java
tags:
  - Java
  - Interface
---

注：本文中的源码版本为 `openjdk-8`。

Java 自带了 2 个 interface：`java.util.Iterator` 和 `java.lang.Iterable`，这两个 interface 用于包含多个对象的序列的迭代操作。

### 1. Iterator

`java.util.Iterator` 的源码如下：

```java
package java.util;

import java.util.function.Consumer;

public interface Iterator<E> {
    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

关于源码的几点说明：

1. 在 API 说明中写到，`remove()` 方法是 `optional operation`，关于此作以下说明，参考 [Optional Methods in Java Interface](https://stackoverflow.com/questions/10572643/optional-methods-in-java-interface)：
    * 首先，Java 要求：The Java language requires that every method in an interface is implemented by every implementation of that interface. **There are no exceptions to this rule.**
    * `optional operation` 并不意味着在第一个 concrete subclass 里可以不 implement 该方法，也不意味着可以用空的方法体，其真正含义是可以采用一种只抛出 `UnsupportedOperationException` 的选择来实现该方法。

1. 关于 interface 定义中方法前边的 `default` 关键字：
    * 接口默认方法是 Java8 引入的一个新特性，该特性允许我们在接口中添加一个非抽象的方法实现，只需使用关键字 `default` 修饰该默认实现方法即可。该特性又叫扩展方法。
    * Java 采用 interface 来取代多继承以避免其潜在危害，interface 要求其方法必须全部是抽象方法，接口中定义的方法必须在接口的非抽象子类中实现，这是 Java 的一个优势。
    * 但是，这会产生一个问题，即如果我们定义的接口已经被外部程序广泛使用，这个时候如果想给该接口增加方法，那么就需要对所有实现该接口的非抽象子类进行修改。也就是说，接口的这个语法限制，导致直接改变/扩展接口内的方法变得非常困难。尤其是在对 Java8 的 Collections API 进行扩充时，很难实现 `Lambda` 表达式等。
    * 因此，Java8 引入了一个新的概念，叫做 `default` 方法，即在接口内部包含了一些默认的方法实现（也就是接口中可以包含方法体，这打破了 Java 之前版本对接口的语法限制），从而使得接口在进行扩展的时候，不会破坏与接口相关的实现类代码。
    * 不过，此时又不可避免地会引入多继承的一个潜在危害，即 *如果一个类实现了两个接口（可以看做是“多继承”），这两个接口又同时都包含了一个名字相同的 `default` 方法，那么编译器会报错。* 此时需要在该子类中重写该方法或者指定调用的是哪一个接口的方法。

虽然 Iterator 接口定义了迭代器的操作方法，但是单单通过实现该接口无法满足我们对迭代器的要求，因此需要引入接口 Iterable。

### 2. Iterable
`java.lang.Iterable` 的源码如下：

```java
package java.lang;

import java.util.Iterator;
import java.util.Objects;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.function.Consumer;

public interface Iterable<T> {
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

可以看到 Iterable 中的方法 `iterator()` 的作用为返回一个 Iterator 对象，并且 Iterable 中定义了一个新的 `default` 遍历方法：

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

参见 [Java8 编程规范入门之【forEach方法遍历集合】](https://www.jianshu.com/p/6babf082f4a3)

* `forEach` 方法接受一个在 JAVA8 中新增的 `java.util.function.Consumer` 的消费行为，或者称之为动作 (Consumer action )类型；
* 然后将集合中的每个元素作为消费行为的 accept 方法的参数执行；
* 直到每个元素都处理完毕或者抛出异常即终止行为；
* 除非指定了消费行为 action 的实现，否则默认情况下是按迭代里面的元素顺序依次处理。

参见 [Java8 新特性之集合： forEach(Consumer<? super T> action)](https://blog.csdn.net/hellomrzheng/article/details/70135884)

* 该 `forEach` 方法其实是一个 Lambda 表达式，而 Lambda 表达式本质上是一个匿名方法，所以这里是一个值传递，即拷贝了一份原来的值，所以外界操作不会再影响它的值，所以“线程安全”。

参见 [Java 8 Lambda 表达式](http://www.runoob.com/java/java8-lambda-expressions.html)

* Lambda 表达式，也可称为闭包，它是推动 Java8 发布的最重要新特性。Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。
* Lambda 表达式的语法格式如下：`(parameters) -> expression` 或 `(parameters) ->{ statements; }`。

### 3. Iterator 与 Iterable 对比
以下用 `DB` 代表我们定义的特定数据结构的类名。若采用 `DB` 实现 Iterator 的方式作为迭代器，存在以下缺点：Iterator 接口中的核心方法 `next()`，`hasNext()`，`remove()`，都是依赖当前位置。如果 `DB` 直接实现 Iterator 接口，则势必导致 `DB` 对象中包含当前迭代位置的数据（指针）。当该对象在不同方法间进行传递的时候，由于当前迭代位置不可知，所以 `next()` 的结果也不可知。除非再为 Iterator 接口添加一个 `reset()` 方法，用来重置当前迭代位置。

为了解决这一问题，可以通过综合应用 Iterator 和 Iterable 的方式来实现迭代器。

### 4. Iterator 与 Iterable 的综合应用
在实践中，通常采用以下方式实现迭代器：

* 首先建立一个 `DBIterator` 类，其 implements Iterator，实现 Iterator 中的方法。
* 然后 `DB` implements Iterable，其重写方法 `Iterator<T> iterator()`，该方法构造并返回一个 `DBIterator`，并且已经初始化，随时可以返回第一个元素。

这种方式解决了只采用 Iterator 方式的问题，此时的 iterator 相当于书签，对同一个 `DB` 对象，可以构造多个不同的 iterator，并且相互之间相互独立，互不影响（不考虑 `remove()` 方法改变 `DB` 结构的情况）。

此外，implements Iterable 之后的 `DB` 可以调用 `forEach()` 方法，并且由于该方法是 `default` 方法，因此在 `DB` 中可以不对其进行重写，非常方便使用。

### 5. ArrayList 实例说明
下边以 ArrayList 类源码中与 Iterator 和 Iterable 有关的部分进行说明：

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
    * Returns an iterator over the elements in this list in proper sequence.
    *
    * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
    *
    * @return an iterator over the elements in this list in proper sequence
    */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

首先，观察以下三个声明：
```java
public interface Collection<E> extends Iterable<E>

public interface List<E> extends Collection<E>

public class ArrayList<E> extends AbstractList<E>
implements List<E>, RandomAccess, Cloneable, Serializable
```

可以看出，ArrayList implements Iterable，并且其方法 `iterator()` 构造并返回一个 `Itr` 类型的对象，而 `Itr` 类 implements Iterator，其中重写了 `hasNext()`，`next()`，`remove()`等方法。

### 6. `forEach` 方法 与 `For-Each Loop` 与 传统 `for loop`

**`forEach()` 方法**

```java
import java.util.ArrayList;

public class iteratorTest {
    public static void main(String[] args) {
        ArrayList<String> al = new ArrayList<String>();
        al.add("1");
        al.add("3");
        al.add("4");
        al.add("6");

        al.forEach(str -> System.out.println(str));
    }
}
```

输出结果为：

    ➜  test javac iteratorTest.java
    ➜  test java iteratorTest
    1
    3
    4
    6
    ➜  test

**传统 `for loop`**

对于定义了自己的 `DBIterator` 类的 `DB` 类的对象 `db` 而言，其可以通过以下方式遍历元素：

```java
for (Iterator i = db.iterator(); i.hasNext(); ) {
    Object o = i.next();
    System.out.println(o);
}
```

**`For-Each Loop`**

定义属于自己的 `DBIterator` 类的另一好处就是可以用下边的 `For-Each Loop` 代替上边传统的 'for loop'：

```java
for (Object o : db) {
    System.out.println(o);
}
```

### 7. 自定义实例
以下以 SListIterator 类的实现代码和 Slist 类的部分实现代码进一步展示：

SListNode 类：
```java
/* list/SListIterator.java */

package list;
import java.util.*;

public class SListIterator implements Iterator {
    SListNode n;

    public SListIterator(SList l) {
        n = l.head;
    }

    public boolean hasNext() {
        return n != null;
    }

    public Object next() {
        if (n == null) {
            /* We’ll learn about throwing exceptions in the next lecture. */
            throw new NoSuchElementException();
        }
        Object i = n.item;
        n = n.next;
        return i;
    }

    public void remove() {
        /* Doing it the lazy way. Remove this, motherf! */
        throw new UnsupportedOperationException("Nice try, bozo."); // In java.lang
    }
}
```

SList 类：
```java
/* list/SList.java */

package list;
import java.util.*;

public class SList implements Iterable {
    SListNode head;
    int size;

    public Iterator iterator() {
        return new SListIterator(this);
    }

    [other methods here]
}
```
