---
layout:     post
title:      EffectiveJava-使用私有构造函数或者枚举类型强制单例属性
subtitle:   静态工厂 单例 Java
date:       2024-4-7
author:     lvyonghao
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Java
---
# 使用私有构造函数或者枚举类型强制单例属性
## 简介
>单例是一个只实例化一次的类。单例通常表示无状态对象或本质上唯一的系统组件。

有两种常见的方法实现单例，基于将构造函数保持私有，在公共静态成员一共对唯一实例的访问。
## 公共final实现单例
书上提供了一个经典的单例实现：
```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```
static保证instance最先被初始化，final保证实例不会被更改，私有化的构造函数保证构造函数只被调用一次。主要优点是明确的指出是一个单例，其次就是实现简单。
> 警告**privileged client**可以借助AccessibleObject.setAccessible调用私有构造函数破坏单例。防御这种攻击需要修改构造函数在被要求创建第二个实例时抛出异常。

```java
public class SafeElvis {
    public static final SafeElvis INSTANCE = new SafeElvis();
    private static boolean instanceCreated = false;

    private SafeElvis() {
        if (instanceCreated) {
            throw new RuntimeException("⚠️ You cannot create another Elvis!");
        }
        instanceCreated = true;
    }
}
```
## 静态工厂实现单例
```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
```
静态工厂方法的一个优点是，工厂返回唯一的实例但是可以对其修改，例如为每个调用他的线程单独返回一个单独的实例：
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {
        return threadLocalInstance.get(); // 每个线程返回一个不同的对象
    }
    public void leaveTheBuilding() { ... }
}
```
静态工厂方法的第二个优点是，可以编写一个通用的单例工厂
```java
public class SingletonFactory {
    //以类为 key、以实例为 value
    private static final Map<Class<?>, Object> instances = new HashMap<>();

    public static synchronized <T> T getInstance(Class<T> clazz) {
        return (T) instances.computeIfAbsent(clazz, SingletonFactory::createInstance);
    }

    private static <T> T createInstance(Class<T> clazz) {
        try {
            return clazz.getDeclaredConstructor().newInstance();
        } catch (...) {
            throw new RuntimeException();
        }
    }
}
```
静态工厂方法还可以被写成方法引用用作函数式接口（本质上匿名内部类写法，函数式接口能被 Lambda “自动实现”）：
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
表示一个没有参数，返回一个对象的函数。如果你使用了静态工厂方法你就可以这样写：
```java
Supplier<Elvis> supplier = Elvis::instance;
```
可以把实例创建逻辑传给其他类作为**延迟调用的工厂**比如用在依赖注入框架、缓存、懒加载容器中。
## 序列化
如果要使用上述任意一种方法实现可序列化，仅在声明中添加implements Serializable是不够的。要在所有实例字段声明为瞬态并提供 readResolve 方法，否则每次反序列化实例都会创建一个新的实例：
```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}

    // 防止反序列化创建新对象
    private Object readResolve() {
        return INSTANCE;
    }
}
```

## 单元素枚举
一种可以免去序列化的方式了类似公共方法，枚举是天然的单例，JVM保证每个enum常量（例子中是INSTANCE）在内存中只有一个唯一的实例，枚举类的构造函数只能被调用一次，不会重复构建。
```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```
## 枚举类和普通类的区别
Java 中的枚举（`enum`）本质上是继承自 `java.lang.Enum` 的特殊类。它拥有普通类的大部分特性，但 JVM 和编译器为它加上了额外的限制和功能，使得枚举类在语义上更安全、结构上更规范。

与普通类相比，枚举具有以下关键特性：

1. **自动继承 `java.lang.Enum`**：枚举类不能继承其他类，所有枚举都默认继承自 `Enum`，这由编译器强制控制。
2. **不可被继承**：所有枚举类型默认是 `final`，无法被继承，避免设计上的不一致或破坏封装。
3. **构造方法只能是私有的**：枚举构造器只能是 `private` 或默认权限，防止外部创建新实例。
4. **反射不可实例化**：即便使用反射 API，JVM 也会禁止对枚举对象进行实例化，这是语言层的保护机制。
5. **线程安全、天然单例**：每个枚举常量在 JVM 中只有一个实例，是天生线程安全的单例模式实现方式。
6. **支持 `switch-case` 语句**：枚举类型可以直接用在 `switch` 中，这是普通类无法做到的语法优势。
7. **自动生成 `values()` 和 `valueOf()` 方法**：
   - `values()` 返回所有枚举常量数组，常用于遍历。
   - `valueOf(String)` 根据名称返回对应的枚举常量，常用于字符串转换。
8. **序列化安全**：枚举在反序列化时不会创建新对象，始终返回原始的唯一实例，不会破坏单例。
9. **每个枚举常量本质是类的实例对象**：和普通类不同，`enum` 中的每个常量其实都是枚举类的唯一实例对象。

这些特性使得 Java 的枚举不仅仅是“几个常量的集合”，而是一个拥有行为、结构和语义的强大类型，非常适合用来建模有限状态、策略集、类型标签、配置选项等。
