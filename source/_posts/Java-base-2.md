---
title: Java基础二
date: 2017-04-04 23:09:09
tags: Java
categories: Java
---

### 1. Java异常处理机制  
[深入理解java异常处理机制](http://blog.csdn.net/hguisu/article/details/6155636)  
[Java异常处理总结](http://lavasoft.blog.51cto.com/62575/18920/)   

#### 1. Java异常  
异常指不期而至的各种状况，如：文件找不到、网络连接失败、非法参数等。异常是一个事件，它发生在程序运行期间，干扰了正常的指令流程。Java通 过API中Throwable类的众多子类描述各种不同的异常。因而，Java异常都是对象，是Throwable子类的实例，描述了出现在一段编码中的 错误条件。当条件生成时，错误将引发异常。  
![](http://img.my.csdn.net/uploads/201211/27/1354020417_5176.jpg)  

##### Error异常  
指程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。  

#### Exception
指程序本身可以处理的异常  

#### 异常分类  
**可检异常与不可检异常**：  
可检异常是编译器要求必须处理的异常，除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。  
不可检异常：包括运行时异常（RuntimeException与其子类）和错误（Error）  

**Exception分类两类：运行时异常和非运行异常**，运行时异常都是RuntimeException类及其子类，运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。非运行异常是RuntimeException以外的异常，从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。  


#### 2. 异常处理机制
- 分为抛出异常和捕获异常。  
- 一旦某个catch捕获到匹配的异常类型，将进入异常处理代码。一经处理结束，就意味着整个try-catch语句结束。其他的catch子句不再有匹配和捕获异常类型的机会。  
- try 块：用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。  
- catch 块：用于处理try捕获到的异常。   
- finally 块：无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行。在以下4种特殊情况下，finally块不会被执行：  
    - 在finally语句块中发生了异常。  
    - 在前面的代码中用了System.exit()退出程序。  
    - 程序所在的线程死亡。  
    - 关闭CPU。  


### try-catch-finally 规则  

- 必须在 try 之后添加 catch 或 finally 块。try 块后可同时接 catch 和 finally 块，但至少有一个块。  
- 必须遵循块顺序：若代码同时使用 catch 和 finally 块，则必须将 catch 块放在 try 块之后。  
- catch 块与相应的异常类的类型相关。  
- 一个 try 块可能有多个 catch 块。若如此，则执行第一个匹配块。即Java虚拟机会把实际抛出的异常对象依次和各个catch代码块声明的异常类型匹配，如果异常对象为某个异常类型或其子类的实例，就执行这个catch代码块，不会再执行其他的 catch代码块  
- 可嵌套 try-catch-finally 结构。  
- 在 try-catch-finally 结构中，可重新抛出异常。
- 除了下列情况，总将执行 finally 做为结束：JVM 过早终止（调用 System.exit(int)）；在 finally 块中抛出一个未处理的异常；计算机断电、失火、或遭遇病毒攻击。  


### 抛出异常

1. throws抛出  
- 如果一个方法可能会出现异常，但没有能力处理这种异常，可以在方法声明处用throws子句来声明抛出异常。  
- throws语句用在方法定义时声明该方法要抛出的异常类型，如果抛出的是Exception异常类型，则该方法被声明为抛出所有的异常。多个异常可使用逗号分割。  
- 使用throws关键字将异常抛给调用者后，如果调用者不想处理该异常，可以继续向上抛出，但最终要有能够处理该异常的调用者。  
throws抛出异常规则：  
    - 如果是不可查异常（unchecked exception），即Error、RuntimeException或它们的子类，那么可以不使用throws关键字来声明要抛出的异常，编译仍能顺利通过，但在运行时会被系统抛出。  
    - **必须声明方法可抛出的任何可查异常（checked exception）。即如果一个方法可能出现受可查异常，要么用try-catch语句捕获，要么用throws子句声明将它抛出，否则会导致编译错误**  
    - 仅当抛出了异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出，而不是囫囵吞枣。  
    - 调用方法必须遵循任何可查异常的处理和声明规则。若覆盖一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或子类。

2. 使用throw抛出异常  
throw总是出现在函数体中，用来抛出一个Throwable类型的异常。程序会在throw语句后立即终止，它后面的语句执行不到，然后在包含它的所有try块中（可能在上层调用函数中）从里向外寻找含有与其匹配的catch子句的try块。  

3. 异常链  
Java方法抛出的可查异常将依据调用栈、沿着方法调用的层次结构一直传递到具备处理能力的调用方法，最高层次到main方法为止。如果异常传递到main方法，而main不具备处理能力，也没有通过throws声明抛出该异常，将可能出现编译错误。  

### Java常见异常  
1. runtimeException子类:  
    1、 java.lang.ArrayIndexOutOfBoundsException数组索引越界异常。当对数组的索引值为负数或大于等于数组大小时抛出。  
    2、java.lang.ArithmeticException算术条件异常。譬如：整数除零等。  
    3、java.lang.NullPointerException空指针异常。当应用试图在要求使用对象的地方使用了null时，抛出该异常。譬如：调用null对象的实例方法、访问null对象的属性、计算null对象的长度、使用throw语句抛出null等等  
    4、java.lang.ClassNotFoundException找不到类异常。当应用试图根据字符串形式的类名构造类，而在遍历CLASSPAH之后找不到对应名称的class文件时，抛出该异常。  
    5、java.lang.NegativeArraySizeException  数组长度为负异常  
    6、java.lang.ArrayStoreException 数组中包含不兼容的值抛出的异常  
    7、java.lang.SecurityException 安全性异常  
    8、java.lang.IllegalArgumentException 非法参数异常    
2. IOException  
    IOException：操作输入流和输出流时可能出现的异常。  
    EOFException   文件已结束异常  
    FileNotFoundException   文件未找到异常  
3. 其他  
    ClassCastException    类型转换异常类  
    ArrayStoreException  数组中包含不兼容的值抛出的异常  
    SQLException   操作数据库异常类  
    NoSuchFieldException   字段未找到异常  
    NoSuchMethodException   方法未找到抛出的异常  
    NumberFormatException    字符串转换为数字抛出的异常  
    StringIndexOutOfBoundsException 字符串索引超出范围抛出的异常  
    IllegalAccessException  不允许访问某类异常  
    InstantiationException  当应用程序试图使用Class类中的newInstance()方法创建一个类的实例，而指定的类对象无法被实例化时，抛出该异常  

### 其它问题  
1. throw和return都可以使方法退出，而且可以互相覆盖  
2. finally里如果有return会把catch里面throw出来的异常覆盖掉  
如下例：  
```Java
public class TestException {  
    public TestException() {  
    }  
  
    boolean testEx() throws Exception {  
        boolean ret = true;  
        try {  
            ret = testEx1();  
        } catch (Exception e) {  
            System.out.println("testEx, catch exception");  
            ret = false;  
            throw e;  
        } finally {  
            System.out.println("testEx, finally; return value=" + ret);  
            return ret;  
        }  
    }  
  
    boolean testEx1() throws Exception {  
        boolean ret = true;  
        try {  
            ret = testEx2();  
            if (!ret) {  
                return false;  
            }  
            System.out.println("testEx1, at the end of try");  
            return ret;  
        } catch (Exception e) {  
            System.out.println("testEx1, catch exception");  
            ret = false;  
            throw e;  
        } finally {  
            System.out.println("testEx1, finally; return value=" + ret);  
            return ret;  
        }  
    }  
  
    boolean testEx2() throws Exception {  
        boolean ret = true;  
        try {  
            int b = 12;  
            int c;  
            for (int i = 2; i >= -2; i--) {  
                c = b / i;  
                System.out.println("i=" + i);  
            }  
            return true;  
        } catch (Exception e) {  
            System.out.println("testEx2, catch exception");  
            ret = false;  
            throw e;  
        } finally {  
            System.out.println("testEx2, finally; return value=" + ret);  
            return ret;  
        }  
    }  
  
    public static void main(String[] args) {  
        TestException testException1 = new TestException();  
        try {  
            testException1.testEx();  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}  
将输出：  
i=2
i=1
testEx2, catch exception
testEx2, finally; return value=false
testEx1, finally; return value=false
testEx, finally; return value=false
因为finally里的return会吃掉testEx中的异常
```
3. 例子1
``` Java
public int inc(){
    int x;
    try {
        x = 1;
        return x;
    } catch (Exception e) {
        x =2;
        return x;
    } finally{
        x = 3;
        // System.out.println("x:" + x);
        //return x;
    }
}
```
这是《深入理解Java虚拟机-JVM高级特性与最佳实践》第二版书中的例子（P187~P188）。出现这种情况的原因是：在没有出线异常的情况下，先执行了x=1;然后执行return x;时，首先是将x的一个副本保存在本地变量表中，执行return之前必须执行finally中的操作：x=3;将x的值设置为了3，但是return时是将本地变量表中保存的x的那个副本拿出来放到栈顶返回。故没出异常时，返回值为1；出Exception异常或其子类异常时，返回值是2；如果出现非Exception异常，则执行完x=3之后，抛出异常，没有返回值。  

#### ==**总结**==
若问道异常相关的问题，这样回答：  
    1. 首先回答异常的概念，即异常是各种不期而至的意外情况，如IO错误，空指针错误等，然后述说异常的继承体系，即异常有两个主要的类Error和Exception，它们都继承于Throwable，其中Error表示编译时和系统错误，Exception表示可以被抛出的异常，分为两种类型：运行时异常和非运行时异常，运行时异常为RuntimeException类及其子类类型，这种异常不需要主动捕获；非运行时异常则需要亲自捕获，否则不能通过编译，这种异常是从程序语法角度上考虑必须处理的异常。  
    2. 然后回答在有多个catch时，只会匹配到最佳的异常，然后会略过其他异常；  
    3. 然后回答try catch finally配对问题，try中必须配对catch或finally,finally作清理工作，如释放数据库连接；  
    4. 然后回答异常覆盖以及异常覆盖返回值问题：即finally里的return会覆盖掉异常的问题和最后个例子；  
    5. 然后回答一般会自定义自己的异常，异常的，异常的处理中需要根据业务判断是否应该处理该异常，不处理则重新抛出。  

---  
### 2. static关键字

[Java Static关键字详解](http://www.cnblogs.com/heimianshusheng/p/5828844.html)  

通常来说，当创建类时，就是在描述那个类的对象的外观与行为。除非用new创建那个对象，否则，实际上并未获得任何对象。执行new来创建对象的时候，数据存储空间才被分配，其方法才供外界调用。有两种情形用上述方法是无法解决的。一种情形是，只想为某特定域分配单一存储空间，而不去考虑究竟要创建多少个对象，甚至根本不需要创建任何对象。另一种情形是，希望某个方法不与包含他的类的任何对象关联在一起。也就是说，==即使没有创建对象，也能够调用方法。==简单来说，static的主要目的就是创建独立于具体对象的域变量与方法。  


- 静态变量  
static变量并不是所在类的某个具体对象所有，而是==该类的所有对象所共有的==，静态变量既能被对象调用，也能直接拿类来调用。  
==类加载时分为加载，链接和初始化三个步骤，同时链接分为三个步骤：验证、准备和解析。在准备阶段会分配static变量内存及设置初始值。这些变量所使用的内存都将在方法区中进行分配==，**在准备阶段的内存分配仅包括static变量而不是实例变量，同时这里所说的初始值通常情况下为数据类型的零值。**， 在初始化阶段会==执行static块以及static变量的初始化==。  
除此之外，静态变量不能引用非静态方法，原因正如前面描述静态加载时机中说的那样，加载静态的时候，非静态的变量、方法等还不存在，当然就无法引用了。  

- 静态方法  

静态方法和静态变量一样，属于类所有，在类加载的同时执行，不属于某个具体的对象，所有对象均能调用。对于静态方法需要注意以下几点：  

    - 它们仅能调用其他的static 方法。  
    - 它们只能访问static数据。  
    - 它们不能以任何方式引用this 或super。  

**静态方法一般用于工具类中，可以直接拿类名调用工具方法进行使用。**  

- 静态类  

一般来说，一个普通类是不允许被声明为static的，但是，在内部类中可以将其声明为static的，这个时候，外部类可以直接调用内部类，因为static的内部类是在加载外部类的同时加载的，所以也就是说，并不要实例化外部类就能直接调用静态内部类。  

```Java
public class BaseStatic {
　　static {
        System.out.println("Load base static");
    }

    public BaseStatic(){
        System.out.println("BaseStatic constructor");
    }
    
    static class BaseInnerClass{
        static{
            System.out.println("Base inner class static");
        }
        
        public BaseInnerClass(){
            System.out.println("BaseInnerClass constructor");
        }
    }
}

public class StaticLoadOrderTest{

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        new BaseStatic.BaseInnerClass();
    }

}
```  
首先，在进入StaticLoadOrderTest的main方法之前，加载StaticLoadOrderTest类，然后执行new BaseStatic.BaseInnerClass();这里需要注意：因为BaseInnerClass是静态的，所以这里==并不需要加载外部类和实例化外部类==，可以直接加载BaseInnerClass并实例化。所以输出为：  
```Java
 Base inner class static
 BaseInnerClass constructor
```  

- 例子  
```Java  
public class BaseStatic {
    static {
        System.out.println("Load base static");
    }
    
    public BaseStatic(){
        System.out.println("BaseStatic constructor");
    }
    
    static class BaseInnerClass{
        static{
            System.out.println("Base inner class static");
        }
        
        public BaseInnerClass(){
            System.out.println("BaseInnerClass constructor");
        }
    }
}

public class StaticLoadOrderTest extends BaseStatic{
    
    static {
        System.out.println("Load test");
    }
    
    public StaticLoadOrderTest(){
        System.out.println("Test constructor");
    }

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        new BaseStatic.BaseInnerClass();new StaticLoadOrderTest(); 
        new BaseStatic();
    }
}
```

会输出：  
```Java
Load test
Base inner class static
BaseInnerClass constructor
Test constructor
Load base static
BaseStatic constructor
```
为什么呢？因为在进入main方法之前，需要加载StaticLoadOrderTest类，这时候发现有static代码块，先加载静态代码块，然后进入main方法内部，所以这里首先会输出Load test，然后调用了new BaseStatic.BaseInnerClass()，因为该类是static类，独立于外部类，所以这里输出Base inner class static以及BaseInnerClass constructor，然后调用new StaticLoadOrderTest()，所以调用了构造函数，所以输出Test constructor，最后同样的道理，调用new BaseStatic()时，首先会调用static静态块，然后构造函数，结果就是剩下的。  

#### ==**总结**==
在问到static相关的问题时。  
    1. 首先需要说出Java虚拟机类加载过程，即那五个步骤：加载、验证、准备、解析和初始化，分别解释下，即加载阶段是读取Java class字节流数据，在方法区生成Class对象，验证是验证字节流中对应结构是否有错，准备阶段则会初始化static变量，这里的初始化是将相应的变量初始化为默认值，解析阶段是将符号引用解析成直接引用，初始化阶段是执行<clinit>函数的阶段，执行该方法时会对static变量进行赋值以及执行static静态块等。这里就说出了static变量及块的执行过程，以及说明了为什么static函数中为什么不能调用非static函数，因为这时非static函数还不存在。      
    2. 然后static为什么需要提出，主要用于创建独立于具体对象的域变量与方法。  
    3. 然后说明static中的用法：static变量、static函数、static静态块以及static静态类，分别说明各自的用法。  
    4. 然后说明静态 

---  
### 3. final关键字  

[深入理解Java中的final关键字](http://www.importnew.com/7553.html)  
[深入理解Java内存模型（六）——final](http://www.infoq.com/cn/articles/java-memory-model-6)  
[Java-Final](http://a123159521.iteye.com/blog/689441)  
final在Java中是一个保留的关键字，可以声明成员变量、方法、类以及本地变量。一旦你将引用声明作final，你将不能改变这个引用了，编译器会检查代码，如果你试图将变量再次初始化的话，编译器会报编译错误。  

- final变量  
final变量经常和static关键字一起使用，作为常量。  

- final方法  

final也可以声明方法。方法前面加上final关键字，代表这个方法不可以被子类的方法重写。final方法比非final方法要**快**，因为==在编译的时候已经静态绑定==了，不需要在运行时再动态绑定。  

- final类  
使用final来修饰的类叫作final类。final类通常功能是完整的，它们不能被继承。Java中有许多类是final的，譬如String, Interger以及其他包装类。  

- final的好处  
    - final关键字提高了性能。JVM和Java应用都会缓存final变量。  
    - final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。  
    - 使用final关键字，JVM会对方法、变量及类进行优化。  

- 关于final的重要知识点  
    - final关键字可以用于成员变量、本地变量、方法以及类。  
    - final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。  
    - 你不能够对final变量再次赋值。   
    - 本地变量必须在声明时赋值。  
    - 在**匿名类**中所有变量都必须是final变量。   
    - final方法不能被重写。  
    - final类**不能被继承**。  
    - final关键字不同于finally关键字，后者用于异常处理。  
    - final关键字容易与finalize()方法搞混，后者是在Object类中定义的方法，是在垃圾回收之前被JVM调用的方法。  
    - ==**接口中声明的所有变量本身是final的**==。  
    - final**和abstract这两个关键字是反相关的**，final类就不可能是abstract的。  
    - ==final方法在**编译阶段绑定，称为静态绑定**==(static binding)。  
    - 没有在声明时初始化final变量的称为空白final变量(blank final variable)，它们必须在构造器中初始化，或者调用this()初始化。不这么做的话，编译器会报错“final变量(变量名)需要进行初始化”。  
    - **将类、方法、变量声明为final能够==提高性能==**，这样JVM就有机会进行估计，然后优化。  
    - 按照Java代码惯例，final变量就是常量，而且通常常量名要大写  

#### **final域的内存模型**
对于final域，编译器和处理器要遵守两个重排序规则：  
    - **在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。**  
    - **初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。**  

```Java
public class FinalExample {
    int i;                            //普通变量
    final int j;                      //final变量
    static FinalExample obj;

    public void FinalExample () {     //构造函数
        i = 1;                        //写普通域
        j = 2;                        //写final域
    }

    public static void writer () {    //写线程A执行
        obj = new FinalExample ();
    }

    public static void reader () {       //读线程B执行
        FinalExample object = obj;       //读对象引用
        int a = object.i;                //读普通域
        int b = object.j;                //读final域
    }
}
```  

- 写final域的重排序规则  

    写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则的实现包含下面2个方面：  
        - JMM禁止编译器把final域的写重排序到构造函数之外。  
        - 编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。  

