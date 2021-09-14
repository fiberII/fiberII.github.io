---
layout: post
title: Class Invariants
---

In OOP, a class invariant (or type invariant) is an invariant used for constraining objects of a class. Methods of the class should preserve the invariant. The class invariant constrains the state stored in the object.

```
类不变量约束着存储在对象中的状态值，或者更准确地说，约束值的范围！

类不变量是一个不变量，用于约束某一个类的对象
类中的方法应该保护该不变量，即类的方法不能使该不变量发生改变
```

what is invariant?
In mathematics, an invariant is a property of a mathematical object (or a class of mathematical objects) which remains unchanged after operations or transformations of a certain type are applied to the objects.

### Example
A simple example of invariance is expressed in our ability to count. For a finite set of objects of any kind, there is a number to which we always arrive, regardless of the order in which we count the objects in the set. The quantity—a cardinal number—is associated with the set, and is invariant under the process of counting. 

Some more complicated examples:
+ The real part and the absolute value of a complex number are invariant under complex conjugation.
+ The degree of a polynomial is invariant under a linear change of variables.
+ The singular values of a matrix are invariant under orthogonal transformations.

```
总的来说什么是不变量？
不变量是指在某一特定操作下不变的值，比如无论从哪种顺序计算一个集合的元素数量，元素数量总是一个不变量
即不变量是有条件的，在某一特定限定下才具有意义

什么是类不变量？
类不变量即类的方法作用之下不改变的值
```

Class invariants are established during construction and constantly maintained between calls to public methods. Code within functions may break invariants as long as the invariants are restored before a public function ends. With concurrency, maintaining the invariant in methods typically requires a critical section to be established by locking the state using a mutex.

An object invariant, or representation invariant, is a computer programming construct consisting of a set of invariant properties that remain uncompromised regardless of the state of the object. This ensures that the object will always meet predefined conditions, and that methods may, therefore, always reference the object without the risk of making inaccurate presumptions. Defining class invariants can help programmers and testers to catch more bugs during software testing.

### Class invariants and inheritance
The useful effect of class invariants in object-oriented software is enhanced in the presence of inheritance. Class invariants are inherited, that is, "the invariants of all the parents of a class apply to the class itself."

Inheritance can allow descendant classes to alter implementation data of parent classes, so it would be possible for a descendant class to change the state of instances in a way that made them invalid from the viewpoint of the parent class. The concern for this type of misbehaving descendant is one reason object-oriented software designers give for favoring composition over inheritance (i.e., inheritance breaks encapsulation).

However, because class invariants are inherited, the class invariant for any particular class consists of any invariant assertions coded immediately on that class in conjunction with all the invariant clauses inherited from the class's parents. This means that even though descendant classes may have access to the implementation data of their parents, the class invariant can prevent them from manipulating those data in any way that produces an invalid instance at runtime.

### Programming language support

#### Assertions
Common programming languages like Python, JavaScript, C++ and Java support assertions by default, which can be used to define class invariants. *A common pattern to implement invariants in classes is for the constructor of the class to throw an exception if the invariant is not satisfied.* Since methods preserve the invariants, they can assume the validity of the invariant and need not explicitly check for it.

```
一种普遍的实现类不变量的方法是在构造器中抛出异常，如果类不变量条件不满足。
```

#### Native support

#### Non-native support
For C++, the Loki Library provides a framework for checking class invariants, static data invariants, and exception safety.

For Java, there is a more powerful tool called Java Modeling Language that provides a more robust way of defining class invariants.

### Examples

#### Java
This is an example of a class invariant in the Java programming language with **Java Modeling Language.** *The invariant must hold to be true after the constructor is finished and at the entry and exit of all public member functions.* Public member functions should define **precondition** and **postcondition** to help ensure the class invariant.

```
下面的例子中使用了Java Modeling language，即注释部分
```

```Java
public class Date {
    int /*@spec_public@*/ day;
    int /*@spec_public@*/ hour;

    /*@invariant day >= 1 && day <= 31; @*/ //class invariant
    /*@invariant hour >= 0 && hour <= 23; @*/ //class invariant

    /*@
    @requires d >= 1 && d <= 31;
    @requires h >= 0 && h <= 23;
    @*/
    public Date(int d, int h) { // constructor
        day = d;
        hour = h;
    }

    /*@
    @requires d >= 1 && d <= 31;
    @ensures day == d;
    @*/
    public void setDay(int d) {
        day = d;
    }

    /*@
    @requires h >= 0 && h <= 23;
    @ensures hour == h;
    @*/
    public void setHour(int h) {
        hour = h;
    }
}
```


