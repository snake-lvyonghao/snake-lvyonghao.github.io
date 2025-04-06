---
layout:     post
title:      2025-4-6-EffectiveJava-使用builder构造多参数构造函数
subtitle:   静态工厂 枚举 Java
date:       2024-4-6
author:     lvyonghao
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Java
---
# 考虑使用builder构造多参数构造函数

## 简介
你是否遇到过一个构造函数有很多个参数，每次创建新的实例都要仔细查看每一个参数是否对应，虽然这个问题现在很多的集成开发环境已经有了跟随你的参数给出提示的功能，但当你要一个可变的的参数列表应该怎么做会更好呢？

## Builder Pattern
下面我用一个例子表示如何实现一个建造者模式：

```java
public class Pizza{
    private final int size;
    private final int servings;
    private final boolean cheese;      
    private final boolean pepperoni; 
    public static class Builder{
        private  int size;
        private  int servings;
        private  boolean cheese;      
        private  boolean pepperoni;

        public Buider size(int value){
            this.size = value; return this;
        }
        public Buider servings(int value){
            this.servings = value; return this;
        }
        public Buider cheese(boolean  value){
            this.cheese = value; return this;
        }
        public Buider pepperoni(boolean value){
            this.pepperoni = value; return this;
        }   
        public PizzaFacts builder() {
            return Pizza(this)
        }
    }
    public Pizza(Builder builder){
        size = builder.size
        servings = builder.servings
        cheese = builder.cheese
        pepperoni = builder.pepperoni               
    }     
}
```
链式调用，非常流畅的创建对象，客户端代码应该是这样的：
```java
Pizza p = new Pizza.Builder().size(5).servings(3).cheese(ture).pepperoni(false).builder();
```

## 在有继承结构的类中使用Builder模式
场景背景是，你有一组pizza类，他们之间是继承的关系，但你用传统的Builder模式每个子类都要写一边会很麻烦，这个时候就要通过父类提供一个抽象的Builder类，里面放通用的构建逻辑，子类继承这个builder构建方法返回具体的子类对象，但是共用父类的逻辑。
```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(topping);
            return self();  //trick：返回子类类型
        }

        abstract Pizza build();

        //子类必须在这个方法返回"this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        this.toppings = builder.toppings.clone();
    }
}
```
这里有一个很关键的点在于子类继承了父类的builder，未来子类会调用父类的Builder的方法addTopping方法，但这个方法返回的类型将会是一个父类的Builder的子类。T代表“将来子类的类型”，一个类型自引用（self-referencing） 的泛型，我写一个子类的例子就很清楚了：
```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        public Builder(Size size) {
            this.size = size;
        }
        @Override public NyPizza build() {
            return new NyPizza(this);
        }
        @Override protected Builder self() { return this; }
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```
NyPizza添加了新的属性size，NyPizza的Builder继承的是Pizza.Builder，addTopping方法应该返回的不是Pizza.Builder而是子类Nypizza.Builder。
子类实例化就可以正常链式调用了：
```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
```