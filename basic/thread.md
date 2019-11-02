# 创建线程的方式及实现
Java中创建线程主要有三种方式：
一、继承Thread类创建线程类
（1）定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完
成的任务。因此把run()方法称为执行体。
（2）创建Thread子类的实例，即创建了线程对象。
（3）调用线程对象的start()方法来启动该线程。
```java
package com.thread;
public class FirstThreadTest extends Thread{
    int i = 0;
    //重写run方法，run方法的方法体就是现场执行体
    public void run()
    {
        for(;i<100;i++){
        System.out.println(getName()+" "+i);
        }
    }
    public static void main(String[] args)
    {
        for(int i = 0;i< 100;i++)
    {
        System.out.println(Thread.currentThread().getName()+" :"+i);
        if(i==20)
        {
            new FirstThreadTest().start();
            new FirstThreadTest().start();
        }
        }
    }
}
```
上述代码中Thread.currentThread()方法返回当前正在执行的线程对象。getName()方法返
回调用该方法的线程的名字。
二、通过Runnable接口创建线程类
（1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是
该线程的线程执行体。
（2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该
Thread对象才是真正的线程对象。
（3）调用线程对象的start()方法来启动该线程。
```java
package com.thread;
public class RunnableThreadTest implements Runnable
{
    private int i;
    public void run()
    {
        for(i = 0;i <100;i++)
        {
        System.out.println(Thread.currentThread().getName()+"
        "+i);
        }
    }
    public static void main(String[] args)
    {
        for(int i = 0;i < 100;i++)
        {
            System.out.println(Thread.currentThread().getName()+"
            "+i);
            if(i==20)
        {
        RunnableThreadTest rtt = new RunnableThreadTest();
            new Thread(rtt,"新线程1").start();
            new Thread(rtt,"新线程2").start();
        }
        }
    }
}
```
三、通过Callable和Future创建线程
（1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且
有返回值。
（2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对
象封装了该Callable对象的call()方法的返回值。
（3）使用FutureTask对象作为Thread对象的target创建并启动新线程。
（4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
```java
package com.thread;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
public class CallableThreadTest implements Callable<Integer>
{
    public static void main(String[] args)
    {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);
        for(int i = 0;i < 100;i++)
        {
            System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
        if(i==20)
        {
         new Thread(ft,"有返回值的线程").start();
        }
        }
        try
        {
            System.out.println("子线程的返回值："+ft.get());
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        } catch (ExecutionException e)
        {
            e.printStackTrace();
        }
    }
    @Overrid
    public Integer call() throws Exception
    {
        int i = 0;
        for(;i<100;i++)
        {
            System.out.println(Thread.currentThread().getName()+""+i);
        }
        return i;
    }
}
```
创建线程的三种方式的对比
采用实现Runnable、Callable接口的方式创见多线程时，
优势是：
    线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
    在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同
    一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面
    向对象的思想。
劣势是：
    编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。
    使用继承Thread类的方式创建多线程时优势是：
    编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this
    即可获得当前线程。
    线程类已经继承了Thread类，所以不能再继承其他父类

# sleep() 、join（）、yield（）有什么区别
1、sleep()方法
    在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度
    程序精度和准确性的影响。 让其他线程有机会继续执行，但它并不释放对象锁。也就是如
    果有Synchronized同步块，其他线程仍然不能访问共享数据。注意该方法要捕获异常
    比如有两个线程同时执行(没有Synchronized)，一个线程优先级为MAX_PRIORITY，另一
    个为MIN_PRIORITY，如果没有Sleep()方法，只有高优先级的线程执行完成后，低优先级
    的线程才能执行；但当高优先级的线程sleep(5000)后，低优先级就有机会执行了。
    总之，sleep()可以使低优先级的线程得到执行的机会，当然也可以让同优先级、高优先级的
    线程有执行的机会。
2、yield()方法
    yield()方法和sleep()方法类似，也不会释放“锁标志”，区别在于，它没有参数，即yield()方
    法只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态
    后马上又被执行，另外yield()方法只能使同优先级或者高优先级的线程得到执行机会，这也
    和sleep()方法不同。
3、join()方法
    Thread的非静态方法join()让一个线程B“加入”到另外一个线程A的尾部。在A执行完毕之前，
    B不能工作。
```java
Thread t = new MyThread();
t.start();
t.join();
```
保证当前线程停止执行，直到该线程所加入的线程完成为止。然而，如果它加入的线程没有
存活，则当前线程不需要停止。

# 说说 CountDownLatch 原理
参考：
[分析CountDownLatch的实现原理](https://www.jianshu.com/p/7c7a5df5bda6?ref=myread)
[什么时候使用CountDownLatch]()
[Java并发编程：CountDownLatch、CyclicBarrier和Semaphore]()

# 说说 CyclicBarrier 原理

