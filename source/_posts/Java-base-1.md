---
title: Java基础一
date: 2017-04-03 22:58:52
tags: Java
categories: Java
---

## 1. Java内存模型
[全面理解Java内存模型](http://blog.csdn.net/suifeng3051/article/details/52611310)   
[volatile和synchronized的区别](http://blog.csdn.net/suifeng3051/article/details/52611233)  
[深入理解Java内存模型（一）](http://www.infoq.com/cn/articles/java-memory-model-1)  

* #### **JMM决定一个线程对共享变量的写入何时对另一个线程可见,线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序员提供内存可见性保证。** 
* 所有原始类型(boolean,byte,short,char,int,long,float,double)的本地变量都直接保存在线程栈当中，对于它们的值各个线程之间都是独立的。对于原始类型的本地变量，一个线程可以传递一个副本给另一个线程，当它们之间是无法共享的。
堆区包含了Java应用创建的所有对象信息，不管对象是哪个线程创建的，其中的对象包括原始类型的封装类（如Byte、Integer、Long等等）。不管对象是属于一个成员变量还是方法中的本地变量，它都会被存储在堆区。
![](http://img.blog.csdn.net/20160921182337904)  
![](http://img.blog.csdn.net/20160921182903818)  
一个本地变量如果是**原始类型**，那么它会被完全存储到栈区。   
一个本地变量也有可能是一个**对象的引用**，这种情况下，这个本地引用会被存储到栈中，但是对象本身仍然存储在堆区。  
对于一个**对象的成员方法**，这些方法中包含本地变量，仍需要存储在栈区，即使它们所属的对象在堆区。   
对于一个**对象的成员变量**，不管它是原始类型还是包装类型，都会被存储到堆区。  
**Static类型的变量以及类本身相关信息**都会随着类本身存储在堆区。  

* #### 硬件内存架构  
![](http://img.blog.csdn.net/20160921183013570)  
![](http://img.blog.csdn.net/20160921183144995)  

* #### 共享对象可见性及共享对象的竞争  
1. 当多个线程同时操作同一个共享对象时，如果没有合理的使用volatile和synchronization关键字，一个线程对共享对象的更新有可能导致其它线程不可见。volatile 关键字可以保证变量会直接从主存读取，而对变量的更新也会直接写到主存。
2. synchronized代码块可以保证同一个时刻只能有一个线程进入代码竞争区，synchronized代码块也能保证代码块中所有变量都将会从主存中读，当线程退出代码块时，对所有变量的更新将会flush到主存，不管这些变量是不是volatile类型的。  

* #### 支撑Java内存模型的基础原理  
1. **指令重排序**：编译器优化重排序，指令级并行重排序，内存系统的重排序    
    1. **as-if-serial**： 不管怎么重排序，单线程下的执行结果不能被改变  
    2. **数据依赖性**： 如果两个操作访问同一个变量，其中一个为写操作，此时这两个操作之间存在数据依赖性。 
编译器和处理器不会改变存在数据依赖性关系的两个操作的执行顺序，即不会重排序。
2. ***内存屏障**： 通过内存屏障可以禁止特定类型处理器的重排序，从而让程序按我们预想的流程去执行。Memory Barrier所做的另外一件事是强制刷出各种CPU cache，如一个Write-Barrier（写入屏障）将刷出所有在Barrier之前写入 cache 的数据，因此，任何CPU上的线程都能读取到这些数据的最新版本。如果一个变量是volatile修饰的，JMM会在**写入这个字段之后插进一个Write-Barrier指令**，并在**读这个字段之前插入一个Read-Barrier指令**。
3. **happens-before**, 在JMM中，如果一个操作的执行结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系,有以下几种基本happens-before关系：              
    1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中任意的后续操作。
    2. 监视器锁规则：对一个锁的解锁操作，happens-before于随后对这个锁的加锁操作。
    3. volatile域规则：对一个volatile域的写操作，happens-before于任意线程后续对这个volatile域的读。
    4. 传递性规则：如果 A happens-before B，且 B happens-before C，那么A happens-before C。

* #### volatile和synchronized的区别  
1. volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；   synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。  
2. volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的  
3. volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性  
4. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。  
5. volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化  

---  
## 2. 多态  
[理解java的三大特性之多态](http://www.cnblogs.com/chenssy/p/3372798.html)  
[Java多态实现原理](http://blog.csdn.net/huangrunqing/article/details/51996424)  


面向对象编程有三大特性：封装、继承、多态。  
封装隐藏了类的内部实现机制，可以在不影响使用的情况下改变类的内部结构，同时也保护了数据。对外界而已它的内部细节是隐藏的，暴露给外界的只是它的访问方法。  
继承是为了重用父类代码。两个类若存在IS-A的关系就可以使用继承。，同时继承也为实现多态做了铺垫。   
**多态简单来说就是在运行时决定调用哪个实例，调用哪个实例的方法**。    
多态分为编译时多态和运行时多态：  
    **编译时多态**：静态的，主要指方法的重载，它根据参数列表的不同来区分不同的函数；  
    **运行时多态**：通过动态绑定实现的，主要指方法的重写。    
 
* #### Java多态实现方式  
Java 对于方法调用动态绑定的实现主要依赖于方法表，但通过类引用调用和接口引用调用的实现则有所不同。总体而言，当某个方法被调用时，JVM 首先要查找相应的常量池，得到方法的符号引用，并查找调用类的方法表以确定该方法的直接引用，最后才真正调用该方法。  

1. **Java运行时结构**   
![](https://www.ibm.com/developerworks/cn/java/j-lo-polymorph/image003.jpg)  

当程序运行需要某个类的定义时，载入子系统 (class loader subsystem) 装入所需的 class 文件，并在内部建立该类的**类型信息**，这个类型信息就存贮在**方法区**。类型信息一般包括该类的方法代码、类变量、成员变量的定义等等。  
该类型信息和 **class 对象是不同**的。class 对象是 JVM 在==载入某个类后于堆 (heap) 中创建的代表该类的对象==，可以通过该 class 对象访问到该类型信息。  

2. **Java的方法调用方式**
* **静态调用**：指对于类的静态方法的调用方式，是静态绑定的，类调用 (invokestatic) 是在编译时刻就已经确定好具体调用方法的情况， JVM的方法调用指令为invokestatic, invokespecial. 
* **动态调用**：需要有方法调用所作用的对象，是动态绑定的。 实例调用 (invokevirtual) 则是在调用的时候才确定具体的调用方法，这就是动态绑定。 动态绑定的JVM指令有invokevirtual和invokeinterface.  

3. **常量池**  
常量池中保存的是一个 **Java 类引用的一些常量信息**，包含一些字符串常量及对于类的符号引用信息等。Java 代码编译生成的类文件中的常量池是**静态常量池**，当类被载入到虚拟机内部的时候，在内存中产生类的常量池叫**运行时常量池**。  

4. **方法表与方法调用**
方法表是动态调用的核心，也是 Java 实现**动态调用的主要方式**。它被存储于**方法区中的类型信息**，包含有该类型所定义的所有方法及指向这些方法代码的**指针**，注意这些具体的方法代码可能是被==覆写==的方法，也可能是==继承==自基类的方法。  

实现方式是：子类和父类各自在方法区创建自己的方法表，当子类重写父类的方法时，会在**子类单独的方法代码中**写入，而子类未重写的方法，则父类和子类**同时指向**（也即父类的方法代码）。当方法调用时，访问实际类的方法表，若该方法和父类共享（未重写），根据偏移量则会指向共享的父类中的方法指针，若是子类重写的方法，根据偏移量则会指向子类的方法代码。  

5. **接口调用**  
由于接口实现中实际方法不能通过方法表的偏移量来正确调用，所以调用接口方法有其专用的调用指令(invokeinterface)，Java对于接口方法的调用是采用**搜索方法表**的方式。例如对如下方法调用：  
```java 
invokeinterface #13
```
JVM 首先查看常量池，确定方法调用的符号引用（名称、返回值等等），然后利用 this指向的实例得到该实例的方法表，进而搜索方法表来找到合适的方法地址。
因为每次接口调用都要搜索方法表，所以从效率上来说，==接口方法的调用总是慢于类方法的调用的==。

---  
## 3. Object类中的方法
[深入研究java.lang.Object类](http://lavasoft.blog.51cto.com/62575/15456/)  
Object类中方法有以下方法：
``` Java
public final native Class<?> getClass(); //返回对象的一个运行时类
public native int hashCode(); //返回该对象的hashcode
public boolean equals(Object obj); //指示某个其它对象是否与此对象“相等”
protected native Object clone(); //创建并返回此对象的一个副本
public String toString(); //返回在此对象监视器上等待的单个线程
public final native void notify(); //唤醒在此对象监视器上等待的单个线程
public final native void notifyAll(); //唤醒在此对象上等待的所有线程
public final native void wait(long timeout) throws InterruptedException; //导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。 
public final void wait(long timeout, int nanos) throws InterruptedException; //导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。
public final void wait() throws InterruptedException; //导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。
protected void finalize() throws Throwable; //当垃圾回收器确定不存在该对象的更多引用时，由对象的垃圾回收器调用此方法
```
* equals()方法：用于测试某个对象是否同**另一个对象相等**。它在Object类中的实现是判断两个对象是否指向同一块内存区域。这中测试用处不大，因为即使内容相同的对象，内存区域也是不同的。如果想测试对象是否相等，就需要**覆盖此方法**，进行更有意义的比较。  

Java语言规范要求equals方法具有下面的特点：  
**自反性**：对于任何非空引用值 x，x.equals(x) 都应返回 true。   
**对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。   
**传递性**：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。   
**一致性**：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。   
**对于任何非空引用值 x，x.equals(null) 都应返回 false。**  

* **toString()**：返回该对象的字符串表示。Object类中的toString()方法会打印出类名和对象的内存位置。几乎每个类都会覆盖该方法，以便打印对该对象当前状态的表示。大多数（非全部）toString()方法都遵循如下格式：类名[字段名＝值，字段名＝值...]，当然，子类应该定义自己的toString()方法。  

* **getClass()**: 返回一个对象的运行时类。该 Class 对象是由所表示类的 static synchronized 方法锁定的对象。  

* **hashCode()**: 返回该对象的哈希码值。
hashCode 的常规协定是：  
    1. 在 Java 应用程序执行期间，在同一对象上多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是对象上 equals 比较中所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。  
    2. 如果根据 equals(Object) 方法，两个对象是相等的，那么在两个对象中的每个对象上调用 hashCode 方法都必须生成相同的整数结果。  
    3. 如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么在两个对象中的任一对象上调用 hashCode 方法必定会生成不同的整数结果。  

* **clone()**: 创建并返回此对象的一个副本。此方法返回的对象应该独立于该对象（正被克隆的对象）  

* **toString()**: 返回该对象的字符串表示。通常，toString 方法会返回一个“以文本方式表示”此对象的字符串。结果应是一个简明但易于读懂。  

* **notify()**: 唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 

* **notifyAll()**: 唤醒在此对象监视器上等待的所有线程。线程通过调用其中一个 wait 方法，在对象的监视器上等待。

* **wait()**: 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法, 会导致线程被禁用，且处于休眠状态，直到发生以下四种情况之一：  
    1. 其他某个线程调用此对象的 notify 方法，并且线程 T 碰巧被任选为被唤醒的线程。  
    2. 其他某个线程调用此对象的 notifyAll 方法。  
    3. 其他某个线程中断线程 T。    
对于超时版本的，还增加一个条件：  
    4. 已经到达指定的实际时间。但是，如果 timeout 为零，则不考虑实际时间，该线程将一直等待，直到获得通知。

* **finalize()**: 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。子类重写 finalize 方法，以配置系统资源或执行其他清除。

---  
## 4. 类访问权限
[Java类成员的四种访问权限](http://blog.csdn.net/leilu2008/article/details/6719591)  

作用域 | 当前类 | 同一package | 子类 | 其它package
---|---|---|---|---
public | OK | OK | OK | OK
protected | OK | OK | OK | NO
private | OK | NO | NO | NO  

Java中有四种访问修饰符：  
* package: 包访问，没有任何修饰符时就采用这种保护模式，包访问允许域和方法被同一个包内任何类的任何方法访问。  
* private： 表示私有的域和方法只能被同一类中的其它方法访问  
* public: 暴露域和方法，以便在类定义的包外都能访问它们；  
* protected: 限制其它package访问。

---  
## 5. sleep、wait方法的联系与区别
[java sleep和wait的区别的疑惑?](https://www.zhihu.com/question/23328075/answer/24228413)  
[wait方法和sleep方法的区别](http://www.cnblogs.com/bethunebtj/p/5696999.html)   

sleep和wait的区别联系[孙立伟回答](https://www.zhihu.com/question/23328075/answer/24228413)：  
* sleep是Thread类的方法,wait是Object类中定义的方法，尽管这两个方法都会影响线程的执行行为，但是本质上是有区别的。  
* Thread.sleep不会导致锁行为的改变，如果当前线程是拥有锁的，那么Thread.sleep不会让线程释放锁, 而wait()方法则会在线程休眠的同时释放掉机锁，其他线程可以访问该对象。  
* Thread.sleep和Object.wait都会暂停当前的线程，对于CPU资源来说，不管是哪种方式暂停的线程，都表示它暂时不再需要CPU的执行时间。OS会将执行时间分配给其它线程。区别是，调用wait后，需要别的线程执行notify/notifyAll才能够重新获得CPU执行时间。  
* 线程的状态参考 Thread.State的定义。新创建的但是没有执行（还没有调用start())的线程处于“就绪”，或者说Thread.State.NEW状态。  
* Thread.State.BLOCKED（阻塞）表示线程正在获取锁时，因为锁不能获取到而被迫暂停执行下面的指令，一直等到这个锁被别的线程释放。BLOCKED状态下线程，OS调度机制需要决定下一个能够获取锁的线程是哪个，这种情况下，就是产生锁的争用，无论如何这都是很耗时的操作。  
博客中增加的几点：  
* join()方法使当前线程停下来等待，直至另一个调用join方法的线程终止。  
*  Yield()方法是停止当前线程，让同等优先权的线程运行。如果没有同等优先权的线程，那么Yield() 方法将不会起作用。  
*  sleep()静态方法能使**线程暂停一段时间**。sleep与wait的不同点是：**sleep并不释放锁**，并且sleep的暂停和wait暂停是不一样的。obj.wait会使线程进入**obj对象的等待集合中并等待唤醒**。但是wait()和sleep()都可以**通过interrupt()方法打断线程的暂停状态**，从而使线程立刻**抛出InterruptedException**。如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep/join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。需要注意的是，**InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的**。对某一线程调用interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到wait()/sleep()/join()后，就会立刻抛出InterruptedException。
 
---  
## 6. String、StringBuffer、StringBuilder联系区别与源码
[【总结】String in Java](http://www.iteye.com/topic/522167)    

### String对象的创建  
- Java class文件结构和常量池  
在class文件中有一个非常重要的项——常量池 。这个常量池专门放置源代码中的符号信息(并且不同的符号信息放置在不同标志的常量表中)。  

- JVM运行class文件  
源代码编译成class文件之后，JVM就要运行这个class文件。它首先会用类装载器加载进class文件。然后需要创建许多内存数据结构来存放class文件中的字节数据。比如class文件对应的类信息数据、常量池结构、方法中的二进制指令序列、类方法与字段的描述信息等等。当然，在运行的时候，还需要为方法创建栈帧等。这么多的内存结构当然需要管理，JVM会把这些东西都组织到几个“运行时数据区 ”中。这里面就有我们经常说的“方法区 ”、“堆 ”、“Java栈 ”等。  
**在Java源代码中的每一个字面值字符串，都会在编译成class文件阶段，形成标志号 为8(CONSTANT_String_info)的常量表 。 当JVM加载 class文件的时候，会为对应的常量池建立一个内存数据结构，并存放在方法区中。同时JVM会自动为CONSTANT_String_info常量表中 的字符串常量字面值 在堆中 创建 新的String对象(intern字符串 对象(注：现在存的是字符串引用) ，又叫拘留字符串对象)。然后把CONSTANT_String_info常量表的入口地址转变成这个堆中String对象的直接地址(常量池解 析)。源代码中所有相同字面值的字符串常量只可能建立唯一一个拘留字符串对象。**  

(1): String s=new String("Hello world");  
在运行这段指令之前，JVM就已经为"Hello world"在堆中创建了一个拘留字符串( 值得注意的是：如果源程序中还有一个"Hello world"字符串常量，那么他们都对应了同一个堆中的拘留字符串)。然后用这个拘留字符串的值来初始化堆中用new指令创建出来的新的String对象，局部变量s实际上存储的是new出来的堆对象地址。  

(2) String s="Hello world";  
局部变量s存储的是早已创建好的拘留字符串的堆地址。 大家好好想想，如果还有一条穿件语句String s1="Hello word"；此时堆中有几个值为"Hello world"的字符串呢?答案是1个。  
- 问题1： 
``` Java
//代码1  
String sa=new String("Hello world");            
String sb=new String("Hello world");      
System.out.println(sa==sb);  // false       
//代码2    
String sc="Hello world";    
String sd="Hello world";  
System.out.println(sc==sd);  // true   
``` 
 代码1中**局部变量sa,sb中存储的是JVM在堆中new出来的两个String对象的内存地址。虽然这两个String对象的值(char[]存放的字符序列)都是"Hello world"。 因此"=="比较的是两个不同的堆地址**。**代码2中局部变量sc,sd中存储的也是地址，但却都是常量池中"Hello world"指向的堆的唯一的那个拘留字符串对象的地址 。自然相等了。**  
 
- 问题2.  
``` Java
//代码1  
String sa = "ab";                                          
String sb = "cd";                                       
String sab=sa+sb;                                      
String s="abcd";  
System.out.println(sab==s); // false  
//代码2  
String sc="ab"+"cd";  
String sd="abcd";  
System.out.println(sc==sd); //true  
```
代码1中**局部变量sa,sb存储的是堆中两个拘留字符串对象的地址。而当执行sa+sb时，JVM首先会在堆中创建一个StringBuilder类，同时用sa指向的拘留字符串对象完成初始化，然后调用append方法完成对sb所指向的拘留字符串的合并操作，接着调用StringBuilder的toString()方法在堆中创建一个String对象，最后将刚生成的String对象的堆地址存放在局部变量sab中。而局部变量s存储的是常量池中"abcd"所对应的拘留字符串对象的地址。** sab与s地址当然不一样了。  
**代码2中"ab"+"cd"会直接在编译期就合并成常量"abcd"， 因此相同字面值常量"abcd"所对应的是同一个拘留字符串对象，自然地址也就相同。**  

### String、StringBuffer和StringBuilder  
- String: 不可变字符序列  
- StringBuffer: 线程安全的可变字符序列  
- StringBuilder: 非线程安全的可变字符序列

#### 1. StringBuffer和String的可变性问题  
**(1)  String中的是常量(final)数组，只能被赋值一次。**  
注意：这个对初学者来说有个误区，有人说String str1=new String("abc"); str1=new String("cba");不是改变了字符串str1吗？那么你有必要先搞懂对象引用和对象本身的区别。这里我简单的说明一下，对象本身指的是存放在堆空间中的该对象的实例数据(非静态非常量字段)。而对象引用指的是堆中对象本身所存放的地址，一般方法区和Java栈中存储的都是对象引用，而非对象本身的数据。  

**(2) StringBuffer中的value[]就是一个很普通的数组，而且可以通过append()方法将新字符串加入value[]末尾。这样也就改变了value[]的内容和大小了。**  
**总结，讨论String和StringBuffer可不可变。本质上是指对象中的value[]字符数组可不可变，而不是对象引用可不可变。 **  

#### 2. StringBuffer和StringBuilder的线程安全性问题  
StringBuffer和StringBuilder可以算是双胞胎了，这两者的方法没有很大区别。但在线程安全性方面，StringBuffer允许多线程进行字符操作。这是因为在源代码中StringBuffer的很多方法都被**关键字synchronized 修饰**了，而StringBuilder没有。  

#### synchronized含义  
==每一个类对象都对应一把锁，当某个线程A调用类对象O中的synchronized方法M时，必须获得对象O的锁才能够执行M方法，否则线程A阻塞。一旦线程A开始执行M方法，将独占对象O的锁。使得其它需要调用O对象的M方法的线程阻塞。只有线程A执行完毕，释放锁后。那些阻塞线程才有机会重新调用M方法。这就是解决线程同步问题的锁机制。==  


#### 3. String和StringBuffer的效率问题  
**大量字符串累加时，StringBuffer的append()效率远好于String对象的"累+"连接**   

### 总结  

- ==**(1) 在编译阶段就能够确定的字符串常量，完全没有必要创建String或StringBuffer对象。直接使用字符串常量的"+"连接操作效率最高。**==  
- ==**(2) StringBuffer对象的append效率要高于String对象的"+"连接操作。**==  
- ==**(3) 不停的创建对象是程序低效的一个重要原因。那么相同的字符串值能否在堆中只创建一个String对象那。显然拘留字符串能够做到这一点，除了程序中的字符串常量会被JVM自动创建拘留字符串之外，调用String的intern()方法也能做到这一点。当调用intern()时，如果常量池中已经有了当前String的值，那么返回这个常量指向拘留对象的地址。如果没有，则将String值加入常量池中，并创建一个新的拘留字符串对象。**==  
