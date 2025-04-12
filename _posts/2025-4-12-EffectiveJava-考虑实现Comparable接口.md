---
layout:     post
title:      EffectiveJava-考虑实现Comparable接口
subtitle:   effective Java
date:       2025-4-12
author:     Lvyonghao
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - Java 
---
# 考虑实现Comparable接口
    类实现了Comparable接口，就可以和泛型算法以及依赖于接口的集合进行协作。如果你编写的一个它有明显的具有内在排序关系的类，那你就要考虑实现Comparable接口。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
    compareTo方法的通用约定与equals相似，要满足当该对象小于等于大于指定对象的时候分别返回一个负整数，零挥着正整数。无法比较要抛出ClassCastException。

## 实例
    在Java8+中我们可以使用比较构造器来实现compareTo方法，优点是简洁明了lambda 表达，以及易组合链式排序（e.g., thenComparing）使用。举个例子，再详细解释：
```java
class PhoneNumber implements Comparable<PhoneNumber> {
    int areaCode;
    int prefix;
    int lineNum;

    private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);

    @Override
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
}
```
    其中comparingInt用于比较int值，comparingInt内部是：
```java
public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator<T> & Serializable)
        (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
}
```
    返回值是一个Comparator接口的实现，传入的参数是ToIntFunction<? super T>，ToIntFunction是一个接口，内部是：
```java
@FunctionalInterface
public interface ToIntFunction<T> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    int applyAsInt(T value);
}
```
    接口的方法是applyAsInt，返回一个int。
用`Objects.requireNonNull(keyExtractor);`检查传入的实现applyAsInt方法的对象是否为空，然后返回一个实现Comparator和Serializable的对象。
`(c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));`是如何做到的呢？lambda 表达式 本质是语法糖，表示的是一个函数式接口的匿名对象，`(c1, c2) -> Integer.compare(...)`相当于：
```java
class MyComparator implements Comparator<T>, Serializable {
    public int compare(T c1, T c2) {
        return Integer.compare(...);
    }
}
```
    Integer.compare内部就是我们最熟悉的用符号比较方式：
```java
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```
    comparingInt的原理就清晰了，传入一个实现了ToIntFunction接口的对象 ，返回一个实现Comparator对象。那`PhoneNumber pn`什么时候实现的ToIntFunction接口呢？是在`(PhoneNumber pn) -> pn.areaCode`时lambda自动生成了一个实现ToIntFunction接口的对象，等价于：
```java
class AreaCodeExtractor implements ToIntFunction<PhoneNumber> {
    public int applyAsInt(PhoneNumber pn) {
        return pn.areaCode;
    }
}
```
    回到COMPARATOR的thenComparingInt，与ComparingInt原理类似，COMPARATOR最终返回一个实现了Comparator的对象，在PhoneNumber中实现Comparable接口只需要在compareTo调用`return COMPARATOR.compare(this, pn);`即可。
其中有一个细节是我们使用了`comparingInt`而不是`comparing`避免了`p -> p.areaCode`自动被装箱为 Integer，然后传入比较器中。
> JDK原文：Returns a lexicographic-order comparator with a function that extracts a primitive int sort key. This is a specialization for performance of comparing(Function).
> 返回一个字典顺序比较器，该比较器具有提取原始int排序键的函数。这是对compare （Function）性能的专门化。

## 总结
    总而言之，每当实现一个对排序敏感的类都应该实现Comparable接口。当在compareTo方法实现比较时避免使用<和>操作符（可能出现整数溢出），而是使用静态的compare方法,或者像上文在Comparator接口中使用比较器构造方法。