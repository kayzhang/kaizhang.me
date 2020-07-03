---
title: Covariance and Contravariance in Java
date: '2020-06-22'
slug: covariance-and-contravariance-in-java
categories:
  - Java
tags:
  - arrays
  - subtyping
  - inhiritance
---

## 1 Subtyping

### 1.1 Types

A type is a set of values. Or a type is a theorem.

### 1.2 Type constructors

> [Type constructor](https://en.wikipedia.org/wiki/Type_constructor)
> 
> In the area of mathematical logic and computer science known as type theory, a type constructor is a feature of a typed formal language that builds new types from old ones.

Suppose `C` is a type constructor, given a type `T`, it can construct a new type, denoted as `C(T)`.

The following are some common type constructors, where `T` and `S` are known types.

* Arrays: `T[]` is a new type constructed from type `T` using array type constructor `[]`.
* Function types: `S->T` is a new type constructed from type `S` and `T` using function type constructor `->`.
* Generic types: `ArrayList<T>` is a new type constructed from type `T` using generic type constructor `ArrayList<>`.

### 1.3 Subtyping vs inheritance

In the object-oriented framework, inheritance is usually presented as a feature that goes hand in hand with subtyping when one organizes abstract datatypes in a hierarchy of classes. However, the two are orthogonal ideas.

> [Subtyping vs. subclassing](https://courses.cs.washington.edu/courses/cse331/17wi/lec11/lec11-subtyping-4up.pdf)
> 
> Substitution (*subtype*) — a *specification* notion
> 
> * `B` is a subtype of `A` iff an object of `B` can masquerade as an
object of `A` in any context
> * About satisfiability (behavior of a `B` is a subset of `A`’s spec)
> 
> Inheritance (`subclass`) — an `implementation` notion
> 
> * Factor out repeated code
> * To create a new class, write only the differences
> 
> Java purposely merges these notions for classes:
> 
> * Every subclass is a Java subtype
>   * But not necessarily a true subtype
> 
> We say that `B` is a ***true subtype*** of `A` if `B` has a stronger
specification than `A`
> 
> * This is ***not*** the same as a ***Java subtype***
> * Java subtypes that are not true subtypes are ***confusing*** and
***dangerous***
>   * But unfortunately common poor-design...

### 1.4 Subtype Polymorphism

> [Subtyping](https://en.wikipedia.org/wiki/Subtyping)
> 
> If `S` is a subtype of `T`, the subtyping relation is often written **S <: T**, to mean that any term of type `S` can be safely used in a context where a term of type `T` is expected.

Some features of subtyping:

* Reflexive: **A <: A**
* Transitive: If **A <: B** and **B <: C**, then **A <: C**.
* Anti-symmetric: If **A <: B** and **B <: A**, then **A == B**.
* **<:** is a partial ordering. For example, `dog` is not a subtype of `cat`, and `cat` is not a subtype of `dog` neither.

## 2 Covariance, contravariance and invariance

Covariance and contravariance is an attribute of type constructors.

* Covariance: **C(S) <: C(T)** if **S <: T**.
* Contravariance: **C(T) <: C(S)** if **S <: T**.
* Invariance: Neither covariance nor contravariance, which means if **C(S) <: C(T)**, it must be **S == T**.

### 2.1 Arrays

Suppose **Cat <: Animal**, then

* For read-only arrays, contravariance is unsafe, so it should be covariance: **(read-only)Cat[] <: (read-only)Animal[]**.
* For write-only arrays, covariance is unsafe, so it should be contravariance: **(write-only)Animal[] <: (write-only)Cat[]**.
* For read-and-write arrays, both covariance and contravariance are unsafe, so it should be invariance.

Note, here read and write refers to the array object itself, instead of the objects pointed by the references stored in the array object. For example, we are considering reading or writing `a[2]`, instead of reading or writing `a[2].age`.

### 2.2 Function types

Function type constructors are contravariant in the input type and covariant in the output type.

**T1->T2 <: S1->S2** if **S1 <: T1** and **T2 <: S2**

## 3 Arrays in Java are covariant

As we mentioned above, read-or-write arrays are neither covariantly safe nor cotravariantly safe. However, for some historical reasons, Java treats arrays as covariant.

### 3.1 Why Java treats arrays as covariant?

> [Covariant arrays in Java and C#](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Covariant_arrays_in_Java_and_C)
> 
> Early versions of Java and C# did not include generics, also termed [parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism). In such a setting, making arrays invariant rules out useful polymorphic programs.
> 
> For example, consider writing a function to shuffle an array, or a function that tests two arrays for equality using the `Object.equals` method on the elements. The implementation does not depend on the exact type of element stored in the array, so it should be possible to write a single function that works on all types of arrays. It is easy to implement functions of type:
> 
> ```java
> boolean equalArrays(Object[] a1, Object[] a2);
> void shuffleArray(Object[] a);
> ```
> 
> However, if array types were treated as invariant, it would only be possible to call these functions on an array of exactly the type Object[]. One could not, for example, shuffle an array of strings.
> 
> Therefore, both Java and C# treat array types covariantly. For instance, in Java `String[]` is a subtype of `Object[]`, and in C# `string[]` is a subtype of `object[]`.

### 3.2 How does Java handle unsafe writing into arrays?

> [Covariant arrays in Java and C#](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Covariant_arrays_in_Java_and_C)
> 
> As discussed above, covariant arrays lead to problems with writes into the array. **Java and C# deal with this by marking each array object with a type when it is created. Each time a value is stored into an array, the execution environment will check that the run-time type of the value is equal to the run-time type of the array. If there is a mismatch, an `ArrayStoreException` (Java) or `ArrayTypeMismatchException` (C#) is thrown**:
> 
> ```java
> // a is a single-element array of String
String[] a = new String[1];
> 
> // b is an array of Object
> Object[] b = a;
> 
> // Assign an Integer to b. This would be possible if b really were
> // an array of Object, but since it really is an array of String,
> // we will get a java.lang.ArrayStoreException.
> b[0] = 1;
> ```
> 
> In the above example, one can *read from* the array (`b`) safely. It is only trying to *write to* the array that can lead to trouble.

More specifically, See [10.8. Class Objects for Arrays](https://docs.oracle.com/javase/specs/jls/se14/html/jls-10.html#jls-10.8)

> Every array has an associated `Class` object, shared with all other arrays with the same component type.
> 
> Although an array type is not a class, the `Class` object of every array acts as if:
> 
> * The direct superclass of every array type is `Object`.
> * Every array type implements the interfaces `Cloneable` and `java.io.Serializable`.

### 3.3 What's the drawback of Java's covariant arrays?

> [Covariant arrays in Java and C#](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Covariant_arrays_in_Java_and_C)
> 
> One drawback to this approach is that it leaves the possibility of a run-time error that a stricter type system could have caught at compile-time. Also, it hurts performance because each write into an array requires an additional run-time check.

### 3.4 Any better design?

> [Covariant arrays in Java and C#](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Covariant_arrays_in_Java_and_C)
> 
> With the addition of generics, Java and C# now offer ways to write this kind of polymorphic function without relying on covariance. The array comparison and shuffling functions can be given the parameterized types
> 
> ```java
> <T> boolean equalArrays(T[] a1, T[] a2);
> <T> void shuffleArray(T[] a);
> ```

One possible implementation for `equalArrays` method:

```java
public class GenericMethodTest {

    public static void main(String []args){
        String[] a = {"a", "b", "c"};
        String[] b = {"a", "b", "c"};
        String[] c = {"a", "b", "d"};
        System.out.println(equalArrays(a, b)); // true
        System.out.println(equalArrays(a, c)); // false
    }
    
    public static <T> boolean equalArrays(T[] a1, T[] a2) {
        if (a1 == a2) {
            return true;
        }
        if (a1 == null || a2 == null) {
            return false;
        }
        
        if (a1.length != a2.length) {
            return false;
        }
        
        for (int i = 0; i < a1.length; i++) {
            if (!a1[i].equals(a2[i])) {
                return false;
            }
        }
        return true;
    }
}
```

Note, if Java arrays are invariant, then `a1` and `a2` must have the same component type. But if Java arrays are covariant, then `T1` and `T2` (component type of `a1` and `a2`) can be either **T1 <: T2** or **T1 >: T2**.s