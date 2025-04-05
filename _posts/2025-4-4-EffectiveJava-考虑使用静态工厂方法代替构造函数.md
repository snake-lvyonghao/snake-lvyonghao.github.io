---
layout:     post
title:      EffectiveJava-考虑使用静态工厂方法代替构造函数
subtitle:   静态工厂 枚举 Java
date:       2024-4-4
author:     lvyonghao
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Java
---
# 考虑使用静态工厂方法代替构造函数
本节介绍Effective-Java中，第一条：使用静态工厂方法替代构造函数。

## 简介
传统方法获取实例的方式是使用默认的构造函数，另一种技术是使用公共的静态工厂方式。这是书中提供的一个简单的demo，将原始的布尔类型转为布尔对象的装箱类。
```
public static Boolean valueOf(boolean b) {
return b ? Boolean.TRUE : Boolean.FALSE;
}
```
## 优点
1. 静态工厂它可以有自己的名字,比如这个getInstance:
```
StackWalker luke = StrackWalker.getInstance();
```
2. 静态工厂可以返回一个不可变的类，不需要每次调用的时候都返回一个新的对象，比如单例模式：
```
public class AppConfig {
    // 私有静态的唯一实例
    private static final AppConfig INSTANCE = new AppConfig();

    // 私有构造器，防止外部 new
    private AppConfig() {
        // 初始化配置
    }

    // 公有的静态工厂方法
    public static AppConfig getInstance() {
        return INSTANCE;
    }

    // 示例方法
    public String getAppName() {
        return "My Awesome App";
    }
}
```
3. 和构造函数不同，可以返回类型的任何子类型对象。我们定义一个接口Friut，然后通过静态工厂方法来返回不同的子类。
```
public interface Fruit {
    String getName();
}
```
然后实现两个子类：
```
public class Apple implements Fruit {
    public String getName() {
        return "Apple 🍎";
    }
}

public class Banana implements Fruit {
    public String getName() {
        return "Banana 🍌";
    }
}

```
使用静态工厂类决定构造什么子类
```
public class FruitFactory {
    public static Fruit createFruit(String type) {
        if ("apple".equalsIgnoreCase(type)) {
            return new Apple();
        } else if ("banana".equalsIgnoreCase(type)) {
            return new Banana();
        } else {
            throw new IllegalArgumentException("Unknown fruit: " + type);
        }
    }
}
```
4. 你可以先定义一个工厂接口或静态工厂类，但返回的实现类可以等到以后才添加（比如通过配置、反射、插件等机制注入），实现 运行时绑定 / 插件式开发 / 解耦部署。

定义一个接口
```
public interface PaymentProcessor {
    void pay(int amount);
}
```
提供一个静态工厂方法,ServiceLoader用来加载或查找所有实现了PaymentProcessor接口类型的实例。
```
public class PaymentFactory{
    public static Payment getPaymentProceessor(){
        ServiceLoader<PaymentProcessor> loader = ServiceLoader.load(PaymentProcessor.class);
        for (PaymentProcessor p : loader) {
            return p; // 返回第一个发现的实现类
        }
    }
}
```
后来，当你新建了另一个模块实现了接口：
```
public class StripePayment implements PaymentProcessor {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via Stripe.");
    }
}
```
你有以下的配置
> # META-INF/services/PaymentProcessor
> com.example.StripePayment
运行时调用
```
PaymentProcessor processor = PaymentFactory.getPaymentProcessor();
processor.pay(99);
```
这段代码并不依赖于StripePayment类的实现，它可以先定义，返回类（StripePayment）可以后面再添加。使用者不用关心你实际返回哪个类，只要用接口即可，实现了解耦和可扩展。
---
## 缺点
1. 如果一个类只提供私有构造函数 + 静态工厂方法（没有 public 或 protected 构造器），那么它就不能被继承。换而言之就是别人无法写这个类的子类。当然缺点的另一面也是他的优点，更鼓励组合而不是继承。