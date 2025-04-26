---
layout:     post
title:      Effective-谨慎使用重载
subtitle:   java 
date:       2025-4-20
author:     Lvyonghao
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - Java 
---
# 简介
重载有别于重写，重载是指相同的方法名，但参数不一样。Effective Java中提到重载方法之间的选择是静态的，而重写方法之间的选择是动态的。动态的意味着当自类包含与父类中相同签名的方法声明时，子类实例方法会调用子类的重写方法。
## 重载可能带来的问题
下面的程序是一个善意的尝试，根据集合类型进行分类,希望打印set，list，unknown collection：
```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
}
}
```
但实际上并没有而是打印了三次Unknow Collection。why？因为参数的编译时类型为` Collection<?>`，唯一适用的方法是第三个`classify(Collection<?> c)`。这样的程序行为是违反直觉的，很容易出错。
再看一个重写的例子：
```java
class Wine {
    String name() { return "wine"; }
}
class SparklingWine extends Wine {
    @Override String name() { return "sparkling wine"; }
}
class Champagne extends SparklingWine {
    @Override String name() { return "champagne"; }
}
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
        new Wine(), new SparklingWine(), new Champagne());
        for (Wine wine : wineList)
        System.out.println(wine.name());
    }
}
```
虽然每次实例在编译时类型都是Wine但是调用重写方法时，也就是运行时jvm会查找实际对象类型，总会执行最具体的重写方法。Effectivejava给出一个结论重写是规范重载是例外，避免混淆使用重载。
## 建议
Effectivejava中指出，一个安全的和保守的策略是永远不要导出两个具有相同参数数量的重载。总是可以为方法赋予不同的名称而不是重载它们。否则会发生一些意想不到的麻烦。举一个例子,程雪将-3到2的整数添加到set和list中再删除0到2的整数，最后输出-3，-2，-1：
```java
public class SetList {
    public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();
    for (int i = -3; i < 3; i++) {
        set.add(i);
        list.add(i);
    }
    for (int i = 0; i < 3; i++) {
        set.remove(i);
        list.remove(i);
    }
    System.out.println(set + " " + list);
    }
}
```
但结果是打印[-3, -2, -1] [-2, 0, 2]，原因是因为list.remove(i) 的调用选择重载 remove(int i) 方法，它将删除列表中指定位置的元素。
另一个麻烦事在lambda表达式和方法引用中，例如考虑如下的代码片段：
```java
new Thread(System.out::println).start();
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```
Thread构造方法和submit看起来相似，但前者可以编译后者却不能，两个方法都是有一个带有Runnable的重载。原因是submit还有一个带有Callable <T> 参数的重载，Callable<T> 要求：无参，有返回值的方法（比如 T call()），Runnable 要求：无参，无返回值的方法（void run()），但是 System.out::println 本身是一个有参数的方法引用！也就是说，System.out::println 不是 Runnable，也不是 Callable！但为什么Thread可以？因为 Thread 只有一个重载 Thread(Runnable)，编译器知道目标是 Runnable，而你传的 System.out::println 也会被检测到不匹配 Runnable，所以在这里是能直接报出错误。

## 总结
如果你认为可以重载的方法并不意味着你应该这样做，通常要避免重载具有相同数量参数的多个签名的方法。如果已经有了这样的重载方法可以添加强制转换将相同参数集传递给不同的重载。