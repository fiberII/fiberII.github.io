---
layout: post
title: Java 注解
---
Java注解又称Java标注，是JDK5.0引入的一种注释机制。

Java语言中的类，方法，变量，参数和包都可以被标注。 和Javadoc不同， *Java标注可以通过反射获取标注内容*。再编译器生成类文件时，标注可以被嵌入到字节码中。Java虚拟机可以保留标注内容，在运行时可以获取到标注内容。当然它页支持自定义Java标注。

## 内置的注解
Java定义了一套注解，共有7个，3个在Java.lang中，剩下4个在Java.lang.annotation中。

作用在代码中的注解是：
+ @Override - 检查该方法是否重写方法。 如果发现其父类，或者是引用的接口并没有该方法，会报编译错误。
+ @Deprecated - 标记过时方法。 如果使用该方法，会报编译警告。
+ @SupperessWarnings - 指示编译器去忽略注释中声明的警告

作用在其他注解中的注解（或者说 元注解）是
+ @Retention - 标识这个注解怎么保存，是之在代码中，还是编入class文件，或者是运行时可以通过反射访问
+ @Documented - 标记这些注解是否包含在用户文档中
+ @Target - 标记这个注解应该是哪种Java成员
+ @Inherited - 标记这个注解是继承哪个注解类（默认 注解并没有继承于任何子类）

从Java7开始，额外添加了3个注解：
+ @SafaVarags - Java 7开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告
+ @FunctionalInterface - 从Java 8开始支持，标识一个匿名函数或函数是接口
+ @Repeatable - Java 8开始支持，标识某注解可以在同一个声明上使用多次


## 1、Annotation架构
![](/images/JavaAnnotation1.PNG)

从中我们可以看出：

**01. 1个Annotation和1个RetentionPolicy关联。**

可以理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性。

**02. 1个Annotation和1~n个ElementType关联。**

可以理解为：对于每1个Annotation对象，可以由若干个ElementType属性。

**03. Annotation有许多实现类，包括：Deprecated, Documented, Inherited, Override 等等。**

Annotation的每一个实现类，都“和1个RetentionPolicy关联”且“和1~n个ElementType关联”。
下面，我先介绍框架图的左半边(如下图)，即 Annotation, RetentionPolicy, ElementType；然后再就 Annotation 的实现类进行举例说明。

![](/images/JavaAnnotation2.PNG)

## 2、Annotation组成部分
Java Annotation的组成中，有3个非常重要的主干类。它们分别是：
+ Annotation.java

```Java
package java.lang.annotation;  
public interface Annotation {  
  
 boolean equals(Object obj);  
  
 int hashCode();  
  
 String toString();  
  
 Class<? extends Annotation> annotationType();  
}
```

+ ElementType.java

```Java
package java.lang.annotation;  
  
public enum ElementType {  
 TYPE, /* 类、接口（包括注释类型）或枚举声明  */  
  
 FIELD, /* 字段声明（包括枚举常量）  */  
  
 METHOD, /* 方法声明  */  
  
 PARAMETER, /* 参数声明  */  
  
 CONSTRUCTOR, /* 构造方法声明  */  
  
 LOCAL_VARIABLE, /* 局部变量声明  */  
  
 ANNOTATION_TYPE, /* 注释类型声明  */  
  
 PACKAGE /* 包声明  */  
}
```

+ RetentionPolicy.java

```Java
package java.lang.annotation;  
public enum RetentionPolicy {  
 SOURCE, /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */  
  
 CLASS, /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */  
  
 RUNTIME /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */  
}
```

说明：

**01. Annotation就是个接口。** 

"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联，并且与 "1～n 个 ElementType" 关联。可以通俗的理解为：每 1 个 Annotation 对象，都会有唯一的 RetentionPolicy 属性；至于 ElementType 属性，则有 1~n 个。

**02. ElementType是Enum枚举类型，它用来指定Annotation的类型。**

"每 1 个 Annotation" 都与 "1～n 个 ElementType" 关联。当 Annotation 与某个 ElementType 关联时，就意味着：Annotation有了某种用途。例如，若一个 Annotation 对象是 METHOD 类型，则该 Annotation 只能用来修饰方法。

**03. RetentionPolicy是Enum枚举类型，它用来指定Annotation的策略。通俗而言，就是不同的RetentionPolicy类型的Annotation的作用域不同。**

"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联。
+ a) 若 Annotation 的类型为 SOURCE，则意味着：Annotation 仅存在于编译器处理期间，编译器处理完之后，该 Annotation 就没用了。 例如，" @Override" 标志就是一个 Annotation。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查！编译器处理完后，"@Override" 就没有任何作用了
+ b) 若 Annotation 的类型为 CLASS，则意味着：编译器将 Annotation 存储于类对应的 .class 文件中，它是 Annotation 的默认行为。
+ c) 若 Annotation 的类型为 RUNTIME，则意味着：编译器将 Annotation 存储于 class 文件中，并且可由JVM读入。
这时，只需要记住"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联，并且与 "1～n 个 ElementType" 关联。学完后面的内容之后，再回头看这些内容，会更容易理解。

## 3、Java自带的Annotation
理解了上面的3个类的作用之后，我们接下来可以讲解Annotation实现类的语法定义了。

### 1）Annotation 通用定义
```
@Documented
@Target(ElementType.TYPE)
@RetentionPolicy(RetentionPolicy.RUNTIME)
public @interface MyAnnotation1{
}
```
说明：
上面的作用是定义一个Annotation，它的名字是MyAnnotation1。定义了MyAnnotation1之后，我们可以在代码中通过"@MyAnnotation1" 来使用它。其他的， @Documented，@Target，@Retention，@Interface都是用来修饰 MyAnnotation1的。下面分别说说它们的含义：

**01）@interface**

使用@interface定义注解时，意味着它实现了Java.lang.annotation.Annotation接口，即该注解就是一个Annotation。
定义Annotation时，@interface是必须的。
注意：它和我们通常的implemented实现接口的方法不同。Annotation接口的实现细节都有编译器完成。通过@interface定义注解后，该注解不能继承其他的注解或接口。

**02）@Documented**

类和方法的Annotation在缺省情况下是不出现在javadoc中的。如果使用@Documented修饰该Annotation，则表示它可以出现在javadoc中。
定义Annotation时，@Documented可有可无；若没有定义，则Annotation不会出现在javadoc中。

**03）@Target(ElementType.TYPE)**

前面我们说过，ElementType是Annotation的类型属性。而@Target的作用，就是来指定Annotation的类型属性。
@Target(ElementType.TYPE)的意思就是指定该Annotation的类型是ElementType.TYPE。这就意味着，MyAnnotation1是来修饰“类，接口（包括注释类型）或枚举声明”的注解。
定义Annotation时，@Target可有可无。若有@Target，则该Annotation只能用于它所指定的地方；若没有@Target，则该Annotation可以用于任何地方。

**04）@Retention(Retention.RUNTIME)**

前面我们说过，RetentionPolicy是Annotation的策略属性，而@Retention的作用，就是指定Annotation的策略属性。
@Retention(RetentionPolicy.RUNTIME) 的意思就是指定该Annotation的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将该Annotation信息保留在.class文件中，并且能被虚拟机读取。
定义Annotation时，@Retention可有可无。若没有@Retention，则默认是@RetentionPolicy.CLASS

### 2) Java自带的Annotation
通过上面的示例，我们能理解：@interface用来声明Annotation，@Document用来表示该Annotation是否会出现在javadoc中，@Target用来指定Annotation的类型，@Retention用来指定Annotation的策略。
理解这一点之后，我们就很容易理解Java中自带的Annotation的实现类，即Annotation架构图的右半边。如下图：

![](/images/JavaAnnotation3.PNG)

**Java常用的Annotation：**

```Java
@Deprecated --所标注的内容，不再被建议使用
@Override   --只能标注方法，表示该方法覆盖父类中的方法
@Documented --所标注的内容，可以出现在javadoc中
@Inherited  --只能被用来标注“Annotation类型”，它所标注的Annotation具有继承性
@Retention  --只能被用来标注“Annotation类型”，而且它被用来指定Annotation的RetentionPolicy属性
@Target     --只能被用来标注“Annotation类型”，而且它被用来指定Annotation的ElementType属性
@SupressWarnings --所标注内容产生的警告，编译器会对这些警告保持静默
```

由于 "@Deprecated 和 @Override" 类似，"@Documented, @Inherited, @Retention, @Target" 类似；下面，我们只对 @Deprecated, @Inherited, @SuppressWarnings 这 3 个 Annotation 进行说明。

**2.1）@Deprecated**

@Deprecated 的定义如下：

```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Deprecated{
}
```
说明：
+ (01) @interface -- 它被用来修饰 Deprecated，意味着 Deprecated 实现了 java.lang.annotation.Annotation 接口；即 Deprecated 就是一个注解。 
+ (02) @Documented -- 它的作用是说明该注解能出现在 javadoc 中。
+ (03) @Retention(RetentionPolicy.RUNTIME) -- 它的作用是指定 Deprecated 的策略是 RetentionPolicy.RUNTIME。这就意味着，编译器会将Deprecated 的信息保留在 .class 文件中，并且能被虚拟机读取。
+ (04) @Deprecated 所标注内容，不再被建议使用。

例如，若某个方法被 @Deprecated 标注，则该方法不再被建议使用。如果有开发人员试图使用或重写被 @Deprecated 标示的方法，编译器会给相应的提示信息。示例如下:

![](/images/JavaAnnotation4.PNG)

说明：
上面是 eclipse 中的截图，比较类中 "getString1() 和 getString2()" 以及 "testDate() 和 testCalendar()" 。
(01) getString1() 被 @Deprecated 标注，意味着建议不再使用 getString1(); 所以 getString1() 的定义和调用时，都会一横线。这一横线是eclipse() 对 @Deprecated 方法的处理。
getString2() 没有被 @Deprecated 标注，它的显示正常。
(02) testDate() 调用了 Date 的相关方法，而 java 已经建议不再使用 Date 操作日期/时间。因此，在调用 Date的API 时，会产生警告信息，途中的 warnings。
testCalendar() 调用了 Calendar 的 API 来操作日期/时间，java 建议用 Calendar 取代 Date。因此，操作 Calendar 不会产生 warning。

**2.2) @Inherited**
@Inherited 的定义如下：
```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited{
}
```
说明：
+ (01) @interface -- 它的用来修饰 Inherited，意味着 Inherited 实现了 java.lang.annotation.Annotation 接口；即 Inherited 就是一个注解。
+ (02) @Documented -- 它的作用是说明该注解能出现在 javadoc 中。
+ (03) @Retention(RetentionPolicy.RUNTIME) -- 它的作用是指定 Inherited 的策略是 RetentionPolicy.RUNTIME。这就意味着，编译器会将 Inherited 的信息保留在 .class 文件中，并且能被虚拟机读取。
+ (04) @Target(ElementType.ANNOTATION_TYPE) -- 它的作用是指定 Inherited 的类型是 ANNOTATION_TYPE。这就意味着，@Inherited 只能被用来标注 "Annotation 类型"。
+ (05) @Inherited 的含义是，它所标注的Annotation将具有继承性。

假设，我们定义了某个 Annotaion，它的名称是 MyAnnotation，并且 MyAnnotation 被标注为 @Inherited。现在，某个类 Base 使用了 MyAnnotation，则 Base "具有了注解 MyAnnotation"；现在，Sub 继承了 Base，由于 MyAnnotation 是 @Inherited的(具有继承性)，所以，Sub 也 "具有了注解 MyAnnotation"。

@Inherited的使用示例：
InheritableSon.java
```Java
import java.lang.annotation.Target;  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Inherited;  
  
/**  
 * 自定义的Annotation。  
 */  
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Inherited  
@interface Inheritable  
{  
}  
  
@Inheritable  
class InheritableFather  
{  
 public InheritableFather() {  
 // InheritableBase是否具有 Inheritable Annotation  
 System.out.println("InheritableFather:"+InheritableFather.class.isAnnotationPresent(Inheritable.class));  
 }  
}  
  
/**  
 * InheritableSon 类只是继承于 InheritableFather，  
 */  
public class InheritableSon extends InheritableFather  
{  
 public InheritableSon() {  
 super(); // 调用父类的构造函数  
 // InheritableSon类是否具有 Inheritable Annotation  
 System.out.println("InheritableSon:"+InheritableSon.class.isAnnotationPresent(Inheritable.class));  
 }  
   
 public static void main(String[] args)  
 {  
 InheritableSon is = new InheritableSon();  
 }  
}
```

运行结果：
```
InheritableFather:true 
InheritableSon:true
```

现在，我们对 InheritableSon.java 进行修改：注释掉 "Inheritable 的 @Inherited 注解"。
InheritableSon.java
```Java
import java.lang.annotation.Target;  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Inherited;  
  
/**  
 * 自定义的Annotation。  
 */  
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
//@Inherited  
@interface Inheritable  
{  
}  
  
@Inheritable  
class InheritableFather  
{  
 public InheritableFather() {  
 // InheritableBase是否具有 Inheritable Annotation  
 System.out.println("InheritableFather:"+InheritableFather.class.isAnnotationPresent(Inheritable.class));  
 }  
}  
  
/**  
 * InheritableSon 类只是继承于 InheritableFather，  
 */  
public class InheritableSon extends InheritableFather  
{  
 public InheritableSon() {  
 super(); // 调用父类的构造函数  
 // InheritableSon类是否具有 Inheritable Annotation  
 System.out.println("InheritableSon:"+InheritableSon.class.isAnnotationPresent(Inheritable.class));  
 }  
   
 public static void main(String[] args)  
 {  
 InheritableSon is = new InheritableSon();  
 }  
}
```

运行结果：
```
InheritableFather:true 
InheritableSon:false
```
对比上面的两个结果，我们发现：当注解Inheritable被@Inherited标注时，它具有继承性。否则，没有继承性。

**2.3）@SupressWarnings**
@SuppressWarnings 的定义如下：
```Java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE}) @Retention(RetentionPolicy.SOURCE) 
public @interface SuppressWarnings { String[] value(); }
```
说明：
(01) @interface -- 它的用来修饰 SuppressWarnings，意味着 SuppressWarnings 实现了 java.lang.annotation.Annotation 接口；即 SuppressWarnings 就是一个注解。
(02) @Retention(RetentionPolicy.SOURCE) -- 它的作用是指定 SuppressWarnings 的策略是 RetentionPolicy.SOURCE。这就意味着，SuppressWarnings 信息仅存在于编译器处理期间，编译器处理完之后 SuppressWarnings 就没有作用了。
(03) @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE}) -- 它的作用是指定 SuppressWarnings 的类型同时包括TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE。
(04) String[] value(); 意味着，SuppressWarnings 能指定参数
(05) SuppressWarnings 的作用是，让编译器对"它所标注的内容"的某些警告保持静默。例如，"@SuppressWarnings(value={"deprecation", "unchecked"})" 表示对"它所标注的内容"中的 "SuppressWarnings 不再建议使用警告"和"未检查的转换时的警告"保持沉默。示例如下：
......

说明：
(01) 左边的图中，没有使用 @SuppressWarnings(value={"deprecation"}) , 而 Date 属于 java 不再建议使用的类。因此，调用 Date 的 API 时，会产生警告。而右边的途中，使用了 @SuppressWarnings(value={"deprecation"})。因此，编译器对"调用 Date 的 API 产生的警告"保持沉默。
补充：SuppressWarnings 常用的关键字的表格

```
deprecation -- 使用了不赞成使用的类或方法时的警告 
unchecked -- 执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型。 
fallthrough -- 当 Switch 程序块直接通往下一种情况而没有 Break 时的警告。 
path -- 在类路径、源文件路径等中有不存在的路径时的警告。 
serial -- 当在可序列化的类上缺少 serialVersionUID 定义时的警告。 
finally -- 任何 finally 子句不能正常完成时的警告。 
all -- 关于以上所有情况的警告。
```

## 4、Annotation的作用
Annotation是一个辅助类，它在Junit、Struts、Spring等工具框架中被广泛使用。
我们在编程中经常会使用到的Annotation作用有：

### 1）编译检查
Annotation具有“让编译器进行编译检查的作用”。
例如，@SupressWarnings，@Deprecated和@Override都具有编译检查作用。

### 2）在反射中使用Annotation
在反射的Class、Method、Field等函数中，有许多与Annotation相关的接口。
这也意味着，我们可以在反射中解析并使用Annotation。

### 3）根据Annotation生成帮助文件
通过给 Annotation 注解加上 @Documented 标签，能使该 Annotation 标签出现在 javadoc 中。
### 4）能够帮忙查看代码
通过 @Override, @Deprecated 等，我们能很方便的了解程序的大致结构。
另外，我们也可以通过自定义 Annotation 来实现一些功能。









