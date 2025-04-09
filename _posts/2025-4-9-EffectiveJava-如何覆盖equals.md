---
layout:     post
title:      如何覆盖equals准则
subtitle:   effective ive Java
date:       2025-4-7
author:     Lvyonghao
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - Java
---
# 如何覆盖equals准则
equals方法是Object默认的方法用来比较两个类的实例是否相等，如果类满足以下两个条件，就可以完全依赖Object默认的equals：
1. 每个对象本身就是唯一的比如Tread，socket，process,FileInputStream，这些天生就是独一无二的。
2. 你不需要逻辑相等的概念，指的是两个对象的属性，或者状态相同，不是地址相同。（equals默认就是比较两个类的地址是否相同）

## 通用约定
当你想通过equals来比价两个实例是否在逻辑上相等而不是地址相同时，重写equals方法要遵循通用约定：
1. 反身性 x.equals(x) 返回true。
2. 对称性 x.equals(y) true y.equals(x) 也应该为 true。
3. 一致性 多次调用x.equals(y) 应该永远返回 true or false。
4. 对于任何非空的引用值 x.equals(null) 都必须返回 false

## equals实践
在前文提到的通用约定中，继承的特性会破坏约定，举个例子,我们有一个类Point代表一个二维平面的点，他有一个子类ColorPoint代表有颜色的点：
```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;   
    Point p = (Point)o;
    return p.x == x && p.y == y;
    }
}

public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
... // Remainder omitted
}
```
ColorPoint的equals方法应该是怎样的呢，如果Point的实例和ColorPoint比较忽略颜色虽然不违反约定但是显然不能接受的。如果通过让ColorPoint.equals进行混合比较时忽略颜色解决问题：

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    // 普通点
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    // 颜色点
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

虽然满足了对称性，但是违反了传递性，现在陷入了两难的境地。effective java中指出无法解决，但可以通过使用组合而非继承来解决这个问题：
```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    /**
    * Returns the point-view of this color point.
    */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // Remainder omitted
}
```
## 高质量的equals
effective java中也给出了高质量equals的秘诀：
1. 使用 == 来检查参数是否为此对象的引用。当你实现equals方法，第一步就该写`if (this == obj) return true;
`。
2. 使用instanceof运算符而不是getclass方法进行比较。
3. 将参数转为正确的类型（强制转化）前要确认有类型检查。
4. 不要在equals声明中用其他的类代替Object。
5. 覆盖equals时一定要覆盖hashCode。

一个最佳实践：
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;                    //  自比较优化
    if (!(o instanceof Point)) return false;       //  类型检查，防止转换出错
    Point p = (Point) o;                           //  类型安全转换
    return x == p.x && y == p.y;                   //  逻辑相等比较
}

```

## 覆盖equals时一定要覆盖hashCode
Java中的集合类使用hashCode()定位元素然后使用equals()判断两个对象是否相等。如果你不这样做，在使用HashMap，HashSet这类集合的时候会有一个bug，我举一个例子：
```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 重写了 equals()，表示两个人的名字和年龄一样就认为相等
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person p = (Person) o;
        return age == p.age && name.equals(p.name);
    }

    //没有重写 hashCode()
}

Set<Person> set = new HashSet<>();

Person p1 = new Person("Alice", 20);
Person p2 = new Person("Alice", 20); // 与 p1 逻辑相等
set.add(p1);
System.out.println("p1.equals(p2): " + p1.equals(p2)); // true
System.out.println("set.contains(p2): " + set.contains(p2)); //false !!
System.out.println("set.size(): " + set.size()); // 1
set.add(p2); //不该添加成功，但会添加进去！
System.out.println("After add p2, set.size(): " + set.size()); //2
```
没有重写hashcode()导致p1,p2的hashcode是基于内存地址的，导致逻辑相等的对象，在集合中表示为不存在，可以重复添加的现象。
正确的修复方式，重写hashcode：
```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```