---
layout:     post
title:      Effective-泛型
subtitle:   java 泛型
date:       2025-4-20
author:     Lvyonghao
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - Java 
---
# 简介
使用泛型，你告诉编译器在每个集合中允许那些类型的对象。编译器会自动插入强制转换，并在编译时告诉你是否尝试插入错误类型的对象。我们在此讨论如何正确的使用泛型，以及相关的参数化，通配符。
术语：参数化类型：`List<String>`,实际参数类型：`String`,泛型类型：`List<E>`,形式类型参数：`E`,无限制通配符类型：`List<?>`,原始类型：`List`，递归类型参数：`<T extends Comparable<T>>`,限制通配符类型：`List<? extends Number>`，泛型方法：`static <E> List<E> asList(E[] a)`,类型令牌：‘String.class’。

## 不要使用原始类型
一个类或接口，它的声明有一个或多个类型参数（type parameters），被称之为泛型类或泛型类接口。泛型类和接口统称为泛型类型。每个泛型定义了一个原始类型，它是没有任何类型参数的泛型类型的名称。例如对应List<E>的原始类型是List。但是不要在代码中使用这种类型，保留这种类型只是因为Java要兼容旧版本！举一个例子说明：
```java
```
```java
List<String> list = new List();
list.add("Hello");
list.add(123);  //传入integer 编译器不会报错
String s = (String) list.get(1); //ClassCastExceptio
```
这种方式没有类型检查，可能在运行时才会发现ClassCastException，不是类型安全的，如果你使用原始类型，则会丧失泛型的所有安全性的表达上的优势。
---
如果你想将任何元素放入具有原始类型的集合中，会轻易的破坏集合的类型不变性。所以永远优先使用 Set<?> 而不是原始类型 Set，举一个例子：
```java
public static void unsafeRawSet(Set set) {
    set.add("Hello");  // 编译器不会警告你
    set.add(123);      // 你可以加任何类型
}
```
调用时；
```java
Set<Integer> intSet = new HashSet<>();
unsafeRawSet(intSet);           // 原始类型方法接受一切

for (Integer i : intSet) {
    System.out.println(i);      // 运行时抛出异常：ClassCastException
}
```
使用通配符：
```java
public static void safeWildcardSet(Set<?> set) {
    // set.add("Hello");  // 编译错误：不允许添加任何值（除了 null）
    for (Object o : set) {
        System.out.println(o);  // 只能读，类型安全
    }
}
```
## 消除非检查警告
使用泛型编程时，会看到许多编译器警告：未经检查的强制转换警告，未经检查的方法调用警告，未经检查的参数化可变长度类型警告以及未经检查的转换警告。尽可能地消除每一个未经检查的警告。
---
堆污染指的是：编译器认为某个泛型集合中存的是 T 类型，结果你通过某些方式放进去了“不是 T 的对象”，
这就违背了泛型的类型安全，运行时可能发生 ClassCastException。
最经典的堆污染例子（泛型 + 可变参数）：
```java
public class HeapPollutionDemo {

    public static void main(String[] args) {
        unsafeMethod(new String[]{"hello"}, new Integer[]{123});
    }

    @SafeVarargs  // 加这个可以抑制警告，但不能避免错误
    public static void unsafeMethod(List<String>... lists) {
        Object[] array = lists;           // List<String>[] 被当作 Object[]
        array[0] = List.of(123);          // 用 List<Integer> 替换了第一个 List<String>
        String s = lists[0].get(0);       // 期望是 String，实际是 Integer → ClassCastException
        System.out.println(s);
    }
}
```
使用 @SafeVarargs 的条件（必须满足）：
- 方法必须是 static、final 或 private
- 因为子类重写可能打破你的“安全承诺”
- 你不能往 varargs 参数里添加或修改元素（特别是使用索引修改）
- 你只读、不写，或者只读引用，编译器才认为你安全

---
常用@SuppressWarnings("unchecked")抑制泛型未检查转换unchecked的警告，举个例子：
```java
import java.util.*;

public class WarningDemo {
    @SuppressWarnings("unchecked") //告诉编译器“我知道这个是泛型强转”
    public static void main(String[] args) {
        List<String> list = (List<String>) new ArrayList();  // 会报警告
        list.add("Hello");
        System.out.println(list.get(0));
    }
}
```
## 列表优于数组
引用effective：
> 数组在两个重要方面与泛型不同。 首先，数组是协变的
（covariant）。 这个吓人的单词意味着如果 Sub 是 Super 的子类
型，则数组类型 Sub[] 是数组类型 Super[] 的子类型。 相比之
下，泛型是不变的（invariant）：对于任何两种不同的类型 Type1 和
Type2 ， List<Type1> 既不是 List<Type2> 的子类型也不是父类
型。[JLS，4.10; Naftalin07,2.5]。

使用泛型时不能吧一个String类型放到一个Integer类型容器中，但是用一个数组，你会发现运行时产生了一个错误，对于列表，可以在编译时就能发现错误。分别举个例子说明下：
```java
// 数组，运行时报错
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in";

//列表，编译时报错
List<Object> ol = new ArrayList<Long>(); // 不兼容的类型
ol.add("I don't fit in");
```
---
由于数组和泛型不能很好的混用，所以创建泛型类型的数组，参数化类型的数组，以及类型参数的数组都是非法的。 因此，这些数组创建表达式都不合法： new List<E>[] ， new List<String>[] ， new E[] 。 所有将在编译时导致泛型数组创建错误。那为什么创建一个泛型数组是非法的？举个例子说明下；
```java
//假设允许泛型数组
List<String>[] stringLists = new List<String>[1]; 
//创建一个Integer类型列表
List<Integer> intList = List.of(42);
//创建数组把泛型数组传入，并把Integer类型列表，List[] 是 Object[] 的子类
Object[] objects = stringLists;
objects[0] = intList;
//报错ClassCastException
String s = stringLists[0].get(0); 
```
泛型数组非法的原因是因为泛型是“擦除型泛型”（Type Erasure），
而数组是协变（covariant）且类型信息保留的，泛型数组 + 类型擦除 = 类型安全无法保证 → 编译器禁止你创建。
## 使用限定通配符来增加API的灵活性
正如上文所提到的，参数化类型是不变的。有时候你需要更多的灵活性。考虑一个Stack类：
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
假设我们想要添加一个方法获取一系列元素，并把他们都push到栈中，泛型新手可能会这样写：
```java
public void pushAll(Iterable<E> src) {
    for (E e : src)
    push(e);
}
```
但是假设有一个`Stack<Number>`调用pushAll(intVal),其中intval的类型是Integer。因为Integer是Number的子类形，从逻辑上看应该没问题，但你如果尝试就会得到错误消息`Iterable<Integer>
cannot be converted to Iterable<Number>`，因为参数化类型是不变的。
对应的解决方法是调用一个限定通配符类型来处理这种情况，pushall输入类型不应该是E的Iterable接口，而是E的某个子类型的Iterable接口。修改后的代码：
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
    push(e);
}
```
现在假设你想写一个popAll方法，与pushAll方法对应，泛型新手可能会这样写：
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
    dst.add(pop());
}
```
假设你有如下的调用,会因为Collection<Object> 不是 Collection<Number> 的子类型报错：
```java
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
```
对应的解决方法是调用一个限定通配符类型来处理这种情况，PopAll输入类型应该是E的某个父类型的集合。修改后的代码：
```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
    dst.add(pop());
}
```
---
总结下限定通配符的使用场景，和原因：
- 从集合中读取元素（只读）`<? extends T>` 元素可能是 T 的子类，读出来当 T 用是安全的, 但是不能写入，因为不知道是哪个子类。
- 向集合中添加元素（只写）`<? super T>`	可以向上转型为父类容器，写入 T 或其子类是安全的，但是不能读，只能读出Object，因为读取时，编译器无法确定实际类型是哪一个父类，所以只能当作 Object 处理.

