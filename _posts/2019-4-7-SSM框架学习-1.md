---
layout:     post
title:      SSM框架学习-1
subtitle:   Spring Maven
date:       2019-4-7
author:     Lvyonghao
header-img: img/post-bg-miui-ux
catalog: true
tags:
    - SSM框架
---

# SSM框架学习-1
---
##0.写在前面
因为要去参加小程序的比赛，迷迷糊糊的入了坑，加了队伍一开始以为我是要去做小程序开发的，后来发现是我想多了，让我去搞后台，emmm后台一窍不通怎么办，经过两天刻苦的网上冲浪以及社会各界人士的帮助下，我找到了一份SSM框架系统学习的网盘资料，学习嘛总是要做一点笔记，从今天开始，就是正式的学习ssm框架了，大家与我一起见证这段艰辛的旅程吧！（一起学习的至少要看完JAVA的基础内容以及关系型数据库的基本操作才行哦）

---
## 1.什么是SSM
SSM框架是由Spring，MyBatis两个框架组成的，但为什么叫SSM呢？因为还有一个SpringMVC，它是包含在Spring中的部分。SSM框架用于比较简单的Web项目开发（虽然咋看也不简单）。

Spring是用来装配bean的大工厂，balabala反正说了一堆看不懂的，简单的来说就是让你可以不用再实例化对象了，其中核心思想是ioC（控制反转），看不懂没关系，慢慢学就知道了。

SpringMVC在项目中拦截用户请求，它的核心Servlet即DispatcherServlet承担中介或是前台这样的职责，将用户请求通过HandlerMapping去匹配Controller，Controller就是具体对应请求所执行的操作，这一段看不懂也没有关系，它是用来简化web技术人员开发工作的目标。

mybatis是对JBDC的封装，JDBC是什么？JDBC是用来操作数据库的Java API，用起来很方便，mybatis他让数据库底层操作变的透明，先不考虑他的实现原理，先知道它的功能就好啦。

---

# 2.Spring
- Spring是一个开源框架
- Spring为简化企业级应用开发而生，使用Spring可以使简单的JavaBean实现以前只有EJB才能实现的功能（我也不懂慢慢来）
- Spring是JavaSE/EE的一站式开发
+ Spring的优点:方便解耦，简化开发，AOP编程支持，声明式事务的支持，方便程序的测试，方便集成各种框架（就是前文提到的那两种框架），降低JavaEE API难度

---

# 3.Spring IOC底层原理
工厂+反射+配置文件
```
//根据ID找到Class反射
<bean id="us" class="com.lvyonghao.UserServicelmp1">
class FactoryBean{
	public static Object getBean(String id){
		//反射生成实例
	}
}
```
避免了传统的开发模式，接口 + 实现的耦合性，当你要修改程序时不得不修改实体或者接口，现在在两者中间加入工厂，但不用修改两者也要修改工厂呀，这时候就用到了反射，和配置文件来做到。


---

# 4.Spring IOC的快速入门
- 下载Spring开发包
- 复制Spring开发jar包到工程
- 理解IOC控制反转和DI依赖注入
- 编写Spring核心配置文件
- 程序中读取Spring配置文件，通过Spring框架获得Bean（相应的Java类），完成相应操作


1.[Spring下载地址](http://www.baidu.com)我选择了4.X的版本，打开一个版本里面有三种文件，选择最大的结尾是dist的下载并解压就好了。
>解压后打开文件夹，其中docs文件夹就是Spring的文档文件夹包含API文档和开发规范，libs文件夹下就是Spring框架的依赖，以及依赖的源代码，schema是框架的约束。

2.打开IDEA创建一个新的项目，选择Maven项目作为管理jar包，选择一个web模版，webapp结尾的那个就是了，记得勾选上左上角的Create from archetype。接下来在GroupID上写上组的ID，com.snake_lvyonghao,类似这样的，在下面写上项目的名字，spring_ioc，类似这样的。下一步选择你的Maven路径以及Maven的配置文件，现在就创建好了一个工程。

3.在IDEA中把Spring文件夹中的jar目录导入项目当中。

4.导入jar包，当工程创建结束时，打开项目文件点开pom.xml,在dependencies标签内引入jar包，输入dependency，会有提示的，输入相应的包名以及版本就好。
需要的依赖有

>log4j,commons-logging,spring-expression,spring-beans,spring-context,spring-core这五个jar

5.在main下新建一个文件夹名为java然后对文件夹右键Mark Directory as Source Root设置为源码文件,在其文件夹下就可以新建包了，在包中新建一个接口,实例java，以及测试java通过配置文件，使用工厂的形式把创建类的权利交给Spring，再使用Spring创建类。代码如下分别是接口，实现类，以及测试的代码：

```

package com.snake_lvyonghao.demo1;

public interface Myservice {
    void sayHello();
}
--------------------------
package com.snake_lvyonghao.demo1;

public class MyserviceImp implements Myservice{
    @Override
    public void sayHello() {
        System.out.println("Hello");
    }
}
---------------------------
package com.snake_lvyonghao.demo1;

import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class springTest {
    @Test
    /**
     * 传统方法
     */

    public void demo1(){
        Myservice myservice = new MyserviceImp();
        myservice.sayHello();
    }

    @Test
    /**
     * Spring实现
     */
    public void demo2(){
        //创建Spring工厂
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        //通过工厂获得类
        Myservice myservice = (Myservice)applicationContext.getBean("myserviceImp");

        myservice.sayHello();
    }
}
```
配置文件是在resource配置文件夹下新建一个XML格式的Spring配置文件，在配置文件中设置bean，填写id以及class（你的实现类名）
`    <bean class="com.snake_lvyonghao.demo1.MyserviceImp" id="myserviceImp"></bean>`

---
# 5.总结
传统方法先创建接口，实现类重写接口，再去创建类对象的高耦合性，Spring IOC使用了工厂，把创建类对象的工作交给了Spring的Bean。
[完整的项目代码点击这里](https://github.com/snake-lvyonghao/SSM_study)
