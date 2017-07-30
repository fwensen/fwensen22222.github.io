---
title: Java基础四
date: 2017-04-04 23:10:48
tags: Java
categories: Java
---

---  
### 1. Enum类型  
[Java 语言中 Enum 类型的使用介绍](https://www.ibm.com/developerworks/cn/java/j-lo-enum/)  
[Java 枚举7常见种用法](http://softbeta.iteye.com/blog/1185573)  
[小谈Java Enum的多态性](http://pf-miles.iteye.com/blog/187155)  
[java enum(枚举)使用详解 + 总结](http://www.cnblogs.com/hyl8218/p/5088287.html)  


1. 为什么引入enum？在没有enum时，我们实现常量使用的方式是static final，但这种方式存在以下的问题：1.类型不安全，因为程序中可能会传入任意值；2. 没有命名空间，static final常量只是类的属性，使用时需要通过类来访问？？？；3. 一致性差，因为这种常量值属于编译器常量，当修改值时，所有引用该常量的地方都会重新编译；4. 类型无指意性，这些常量值在仅是些无任何意义的整型值，若写入日志可能仅是些数字，除了开发的程序员一般不能了解其中的意义。所以引入了enum关键字。  

2. enum的使用：
    - 作为常量；  
    - 用于switch;  
    - 向枚举中添加新方法，注意在最后一个枚举常量后需加分号，构造函数需为private；  
    - 覆盖枚举方法，如覆盖toString();   
    - 实现接口，由于所有的枚举都继承于java.lang.Enum类，所以只能实现接口；  
    - 使用接口组织枚举，即接口内声明枚举，这些枚举都实现该接口；  
    - 枚举集合：如EnumSet和EnumMap，

3. enum实现原理  

enum类型的变量都是**继承于Enum类**，然后实现时，里面会声明这些static final的Enum类，同时里面会有一个static静态块，然后这些静态块各自实例化这些Enum对象，**由于JVM类初始化是线程安全的，所以可以采用枚举类实现一个线程安全的单例模式**：因为enum里的字段在实现时是static final的，所以它们会在初始化时期<clinit>初始化，而初始化时期是线程安全的。  例子如下：  
```Java
public enum Fruit {  
    APPLE, PEAR, PEACH, ORANGE;  
}  

//反编译后
public final class Fruit extends Enum  
{  
  
    private Fruit(String s, int i)  
    {  
        super(s, i);  
    }  
  
    public static Fruit[] values()  
    {  
        Fruit afruit[];  
        int i;  
        Fruit afruit1[];  
        System.arraycopy(afruit = ENUM$VALUES, 0, afruit1 = new Fruit[i = afruit.length], 0, i);  
        return afruit1;  
    }  
  
    public static Fruit valueOf(String s)  
    {  
        return (Fruit)Enum.valueOf(test/Fruit, s);  
    }  
  
    //静态的Fruit类
    public static final Fruit APPLE;  
    public static final Fruit PEAR;  
    public static final Fruit PEACH;  
    public static final Fruit ORANGE;  
    private static final Fruit ENUM$VALUES[];  
  
    //静态块内初始化
    static   
    {  
        APPLE = new Fruit("APPLE", 0);  
        PEAR = new Fruit("PEAR", 1);  
        PEACH = new Fruit("PEACH", 2);  
        ORANGE = new Fruit("ORANGE", 3);  
        ENUM$VALUES = (new Fruit[] {  
            APPLE, PEAR, PEACH, ORANGE  
        });  
    }  
}  
```
4. 使用enum实现单例类  
> 使用enum关键字来实现单例模式的好处是这样非常简洁，并且无偿地提供了序列化机制，绝对防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候。  
```Java
public enum Singleton {
    
    INSTANECE;
    public void foo(){}
    ...//其它函数
}

//使用时
Singleton instacne = Singleton.INSTACNE;
instance.foo();
```

---  

### 2. 注解  

[Java中的注解是如何工作的？](http://www.importnew.com/10294.html)  
[Java注解全面解析](http://www.cnblogs.com/longshiyVip/p/5189525.html)  

