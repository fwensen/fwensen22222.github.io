---
title: Java基础三
date: 2017-04-04 23:10:37
tags: Java
categories: Java
---

### 1、Java反射  
[谈谈Java反射机制](http://www.jianshu.com/p/6277c1f9f48d#)  
[Java反射机制应用实践](http://www.ziwenxie.site/2017/03/22/java-reflection/)  
[详解Java反射各种应用](https://github.com/byhieg/JavaTutorial/tree/master/src/main/java/cn/byhieg/reflectiontutorial)  
#### 总结
- **为什么需要反射？** 因为Java不是动态语言，然而若要能有一种动态的机制，那么需要反射，例如在动态语言python中，我们可以动态的赋值，而不需要考虑具体的类型。为支持类似这一种机制引入了反射，反射是什么呢？我的理解是能在程序运行时访问更改类信息，如方法变量等，以支持一种更加灵活操作类的行为。而这些行为又是很多框架所依赖的，例如在Spring中，需要读取XML文件，然后使用反射来创建类等。同时使用反射可以动态加载类，这样可使得程序更加灵活，不会对于未使用的类预先支付现金，而是在使用时，什么叫动态加载类呢， 运行时刻加载的类是动态加载类，而在编译时刻加载的类是静态加载类。  
- 反射有哪些内容？
    1. **java.lang.Class类**： 可理解成**任何一个类都是Class的实例类**，即任何一个类都可以通过Class来得到相应的实例类，可以说是一种类类型，任何一个类都可以通过以下三个方式之一获得相应的Class类：  
        - className.class: 其中className为类名，如Integer.class；  
        - instance.getClass(): instance为类实例，如String string = new String("Hello"), 那么就有string.getClass();  
        - Class.forName("className"): 这里className为类的全名，即包括了类的包信息，如Class.forName("com.xxx.C");  
        - 我们可以通过getName()函数得到类名信息，还有getSimpleName()函数等。  

        在得到了Class类之后，我们就可以使用方法newInstance()方法创建类的实例，就和new类似。  
    2. **方法的反射**  
    通过1中三种方式在得到Class类信息后，我们便可以通过该类信息得到类的方法，getMethods()方法会得到Method数组，里面都是类的方法信息，同时在Method中，通过getName()函数得到方法名，通过getReturnType()得到返回值类型，在返回值类信息上调用getName()也可以得到返回类型的类型名，getParameterTypes()得到参数类型信息，同时在参数类型信息上调用getName()可得到类型名。  
    
    3. **成员变量信息**  
    成员变量信息可知类类型信息上调用getFields()函数得到成员变量信息(Field)，成员变量信息上调用getType()可得到Class类型的类类型信息，调用getName()可得到变量名，getFields()函数是针对public变量的，若想要访问private变量，则需调用getDeclaredFields(methodName)函数进行相应的操作，该函数返回Filed信息，然后在其上调用setAccessible(true)，即可得到相应信息，还可更改，其实这可对应于方法的反射，方法的反射也可使用这种方式调用private方法。  
    
    4. **构造函数的反射**  
    构造函数的反射不再赘述。  
    
    5. **注解信息反射**  
    ...
    
- 反射的应用  
    1. 结合注解使用：  
    我们可以通过反射来访问方法的注解信息，然后作相应的判断，如现在Spring中中提倡使用注解方式来装配类，那么其中的实现方式则是通过反射获得注解，然后作一定的判断。  
    2. 结合XML使用，例如Spring中使用XML配置文件管理类。  
    3. 解决泛型擦除问题  
    泛型擦除问题：在Java泛型中，有一个泛型擦除问题:在使用泛型时，任何具体的类型都被擦除，唯一知道的是你在使用一个对象。比如：List<String>和List<Integer>在运行事实上是相同的类型。他们都被擦除成他们的原生类型，这个问题可使用泛型来解决，这时可使用newInstance()方法来获得Class类所真正指向的类实例。  


---  
### 2. 动态代理  

[Java 动态代理作用是什么？](https://www.zhihu.com/question/20794107)  
[java动态代理原理及解析](http://blog.csdn.net/scplove/article/details/52451899)  
[ JAVA学习篇--静态代理VS动态代理](http://blog.csdn.net/hejingyuan6/article/details/36203505)  
[JDK动态代理实现原理](http://rejoy.iteye.com/blog/1627405)  
[Java 动态代理机制分析及扩展](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/#ibm-pcon)   

1. 什么是代理模式  
> 代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。   
![](http://img.blog.csdn.net/20140701192620959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGVqaW5neXVhbjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
> 更通俗的说，代理解决的问题当两个类需要通信时，引入第三方代理类，将两个类的关系解耦，让我们只了解代理类即可，而且代理的出现还可以让我们完成与另一个类之间的关系的统一管理，但是切记，代理类和委托类要实现相同的接口，因为代理真正调用的还是委托类的方法。  
> 按照代理的创建时期，代理类可以分为两种：   
>   静态：由程序员创建代理类或特定工具自动生成源代码再对其编译。在程序运行前代理类的.class文件就已经存在了。  
>   动态：在程序运行时运用反射机制动态创建而成。

#### 1. 代理模式总结  
代理模式中，客户端不需要知道实现类的具体类，它只需要实现相应的接口即可，这样可实现**解耦**，但同时由一些问题：1. 代理类和委托类实现了相同的接口，代理类通过委托类实现了相同的方法。这样就出现了**大量的代码重复**。如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。2. 代理对象**只服务于一种类型的对象**，如果要服务多类型的对象。势必要为每一种对象都进行代理，静态代理在程序规模稍大时就无法胜任了。  
代理可以对实现类进行统一的管理，如在调用具体实现类之前，需要打印日志等信息，这样我们只需要添加一个代理类，在代理类中添加打印日志的功能，然后调用实现类，这样就避免了修改具体实现类。满足我们所说的**开闭原则**。==**但是如果想让每个实现类都添加打印日志的功能的话，就需要添加多个代理类，以及代理类中各个方法都需要添加打印日志功能**==，即静态代理类只能为特定的接口(Service)服务。如想要为多个接口服务则需要建立很多个代理类。  

基于以上原因，引入了动态代理机制。  

2. 动态代理
> 动态代理是在运行时，通过反射机制实现动态代理，并且能够代理各种类型的对象。  
    1. 动态代理如何使用？  
    代理类需要实现InvocationHandler接口，实现里面的invoke函数，获得相应的实现类时需使用Proxy类的newProxyInstance()函数。  


3. 动态代理实现原理  

    - newProxyInstance(ClassLoader loader,  Class<?>[] interfaces, InvocationHandler h): 该函数会调用getProxyClass(loader, interfaces)获得代理类对象类型(Class)，然后调用Class的getConstructor得到对象类型的构造函数($ProxyN, N指代创建次数)，调用构造函数(newInstance)得到代理类的实例对象。  
    - getProxyClass(classLoader loader, Class<?>... interfaces)：加载目标实现的接口到内存中，然后以缓存的方式提高效率，在ProxyGenerator的generateProxyClass函数中获得动态代理类的字节码，生成代理类实例，返回。  
    - generateProxyClass(final String name, Class[] interfaces)：生成class字节码的地方。   
    - 谁来调用InvocationHandler的invoke方法： $ProxyN类继承于Proxy类，然后实现代理接口，里面的实现函数(即希望有业务的函数)会调用InvocationHandler的invoke函数，达到代理的作用。  
    
    1. 问题：为什么动态代理只能是代理接口？因为**内部实现代理类时继承了Proxy，而Java只能是单继承的**，但能多实现接口，故只能代理接口。  


#### 总结  
若问到动态代理的问题时  
- 首先回答静态代理的实现原理(代理模式)，静态代理的优缺点：优点是实现解耦，缺点是1. **每个代理类对应一个实现类，这样会有大量的冗余代码； 2. 因为每个代理对应一个实现类，所以若是需要对实现类增加打印、日志、安全等功能时，那么需要有多个代理类才可，这样同样会有大量的冗余，同时在每个代理类中都要加入相同的日志、打印、安全等功能**，而动态代理的引入正是解决这两个问题：使用一个代理类就可以对应多个实现类，这样在实现日志、安全、打印事务等功能时，只需修改该代理类即可，类职责更加单一，复用性更强，无需大量的修改。例如Spring的AOP。  
- 然后回答动态代理实现原理：**动态代理会实现一个代理类$ProxyN，该代理类继承于Proxy类，实现代理接口，该代理类$ProxyN是通过生成代理对象的字节码，然后得到代理类的对象实例**， 动态代理之所以只能代理接口的原因是Java不能实现类的多继承。
