# JUC并发编程与源码分析



## 基础知识

![image-20210805143653369](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805143653369.png)

### juc口诀

- 高内聚低耦合前提下，封装思想
  - 线程
  - 操作
  - 资源类
- 判断、干活、通知
- 防止虚假唤醒,wait方法要注意使用while判断
- 注意标志位flag，可能是volatile的

### 为什么多线程重要？

- 硬件方面

摩尔定律失效

摩尔定律：
它是由英特尔创始人之一Gordon Moore(戈登·摩尔)提出来的。其内容为：
当价格不变时，集成电路上可容纳的元器件的数目约每隔18-24个月便会增加一倍，性能也将提升一倍。
换言之，每一美元所能买到的电脑性能，将每隔18-24个月翻一倍以上。这一定律揭示了信息技术进步的速度。

可是从2003年开始CPU主频已经不再翻倍，而是采用多核而不是更快的主频。

摩尔定律失效。

在主频不再提高且核数在不断增加的情况下，要想让程序更快就要用到并行或并发编程。

- 软件方面

zhuanability--高并发系统，异步+回调等生产需求

### 从start一个线程说起

#### Java线程理解以及openjdk中的实现

- private native void start0();
- Java语言本身底层就是C++语言
- OpenJDK源码网址
  - http://openjdk.java.net/
  - openjdk8\hotspot\src\share\vm\runtime

#### 底层c++源码解读

- openjdk8\jdk\src\share\native\java\lang---thread.c

| java线程是通过start的方法启动执行的，主要内容在native方法start0中，<br/>Openjdk的写JNI一般是一一对应的，Thread.java对应的就是Thread.c<br/>start0其实就是JVM_StartThread。此时查看源代码可以看到在jvm.h中找到了声明，jvm.cpp中有实现。 |
| ------------------------------------------------------------ |

![image-20210805144733875](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805144733875.png)

- openjdk8\hotspot\src\share\vm\prims---jvm.cpp

![image-20210805144805946](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805144805946.png)

![image-20210805144814118](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805144814118.png)

- openjdk8\hotspot\src\share\vm\runtime---thread.cpp

![image-20210805144844274](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805144844274.png)

### Java多线程相关概念

#### 进程

是程序的⼀次执⾏，是系统进⾏资源分配和调度的独⽴单位，每⼀个进程都有它⾃⼰的内存空间和系统资源

#### 线程

- 在同⼀个进程内⼜可以执⾏多个任务，⽽这每⼀个任务我们就可以看做是⼀个线程

- ⼀个进程会有1个或多个线程的

#### 何为进程和线程？

![image-20210805145050014](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145050014.png)

![image-20210805145100293](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145100293.png)

![image-20210805145111753](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145111753.png)

![image-20210805145126133](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145126133.png)

![image-20210805145134334](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145134334.png)

![image-20210805145142327](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145142327.png)

![image-20210805145155411](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145155411.png)

![image-20210805145203198](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145203198.png)

![image-20210805145211048](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145211048.png)

![image-20210805145218448](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145218448.png)

![image-20210805145226581](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145226581.png)

#### 管程

Monitor(监视器)，也就是我们平时所说的锁

```java
Monitor其实是一种同步机制，他的义务是保证（同一时间）只有一个线程可以访问被保护的数据和代码。
 
JVM中同步是基于进入和退出监视器对象(Monitor,管程对象)来实现的，每个对象实例都会有一个Monitor对象，
 Object o = new Object();

new Thread(() -> {
    synchronized (o)
    {

    }
},"t1").start();
 
Monitor对象会和Java对象一同创建并销毁，它底层是由C++语言来实现的。
```

![image-20210805145346100](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145346100.png)

### 用户线程和守护线程

Java线程分为用户线程和守护线程，线程的daemon属性为true表示是守护线程，false表示是用户线程

#### 守护线程

是一种特殊的线程，在后台默默地完成一些系统性的服务，比如垃圾回收线程

#### 用户线程

是系统的工作线程，它会完成这个程序需要完成的业务操作

```java
public class DaemonDemo{
    public static void main(String[] args){
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t 开始运行，"+(Thread.currentThread().isDaemon() ? "守护线程":"用户线程"));
            while (true) {

            }
        }, "t1");
        //线程的daemon属性为true表示是守护线程，false表示是用户线程
        t1.setDaemon(true);
        t1.start();
        //3秒钟后主线程再运行
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("----------main线程运行完毕");
    }

}
```

#### 重点

- 当程序中所有用户线程执行完毕之后，不管守护线程是否结束，系统都会自动退出
- 设置守护线程，需要在start()方法之前进行

## CompletableFuture

### Future和Callable接口

![image-20210805145841612](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145841612.png)

Future接口定义了操作异步任务执行一些方法，如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕等。

Callable接口中定义了需要有返回的任务需要实现的方法。

比如主线程让一个子线程去执行任务，子线程可能比较耗时，启动子线程开始执行任务后，
主线程就去做其他事情了，过了一会才去获取子任务的执行结果。

### 从之前的FutureTask说开去

#### 本源的Future接口相关架构

![image-20210805145933283](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145933283.png)

![image-20210805145940671](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805145940671.png)

#### get()阻塞

一旦调用get()方法，不管是否计算完成都会导致阻塞

```java

import java.util.concurrent.*;

public class CompletableFutureDemo{
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException{
        FutureTask<String> futureTask = new FutureTask<>(() -> {
            System.out.println("-----come in FutureTask");
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            return ""+ThreadLocalRandom.current().nextInt(100);
        });

        Thread t1 = new Thread(futureTask,"t1");
        t1.start();

        //3秒钟后才出来结果，还没有计算你提前来拿(只要一调用get方法，对于结果就是不见不散，会导致阻塞)
        //System.out.println(Thread.currentThread().getName()+"\t"+futureTask.get());

        //3秒钟后才出来结果，我只想等待1秒钟，过时不候
        System.out.println(Thread.currentThread().getName()+"\t"+futureTask.get(1L,TimeUnit.SECONDS));

        System.out.println(Thread.currentThread().getName()+"\t"+" run... here");

    }
}
```

#### isDone()轮询

- 轮询的方式会耗费无谓的CPU资源，而且也不见得能及时地得到计算结果.
- 如果想要异步获取结果,通常都会以轮询的方式去获取结果，尽量不要阻塞

```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class CompletableFutureDemo2{
    public static void main(String[] args) throws ExecutionException, InterruptedException{
        FutureTask<String> futureTask = new FutureTask<>(() -> {
            System.out.println("-----come in FutureTask");
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            return ""+ThreadLocalRandom.current().nextInt(100);
        });

        new Thread(futureTask,"t1").start();

        System.out.println(Thread.currentThread().getName()+"\t"+"线程完成任务");

        /**
         * 用于阻塞式获取结果,如果想要异步获取结果,通常都会以轮询的方式去获取结果
         */
        while(true)
        {
            if (futureTask.isDone())
            {
                System.out.println(futureTask.get());
                break;
            }
        }

    }
}
```

总结：

- 不见不散
- 过时不候
- 轮询

想完成一些复杂的任务：

- 应对Future的完成时间，完成了可以告诉我，也就是我们的回调通知
- 将两个异步计算合成一个异步计算，这两个异步计算互相独立，同时第二个又依赖第一个的结果。
- 当Future集合中某个任务最快结束时，返回结果。
- 等待Future结合中的所有任务都完成。
- ......

### 对Future的改进

#### CompletableFuture和CompletionStage源码介绍

##### 类架构说明

![image-20210805150520273](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805150520273.png)

![image-20210805150526632](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805150526632.png)

##### 接口CompletionStage

![image-20210805150604286](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805150604286.png)

代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段，有些类似Linux系统的管道分隔符传参数。

##### 类CompletableFuture

![image-20210805150629724](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805150629724.png)

#### 核心的四个静态方法，来创建一个异步操作

##### runAsync 无 返回值

- public static CompletableFuture<Void> runAsync(Runnable runnable)
- public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)

##### supplyAsync 有 返回值

- public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
- public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,Executor executor)

##### 上述Executor executor参数说明

- 没有指定Executor的方法，直接使用默认的ForkJoinPool.commonPool() 作为它的线程池执行异步代码。
- 如果指定线程池，则使用我们自定义的或者特别指定的线程池执行异步代码

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class CompletableFutureDemo2{
    public static void main(String[] args) throws ExecutionException, InterruptedException{
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName()+"\t"+"-----come in");
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println("-----task is over");
        });
        System.out.println(future.get());
    }
}
```

![image-20210805150847979](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805150847979.png)

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class CompletableFutureDemo2{
    public static void main(String[] args) throws ExecutionException, InterruptedException{
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "-----come in");
            //暂停几秒钟线程
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return ThreadLocalRandom.current().nextInt(100);
        });

        System.out.println(completableFuture.get());
    }
}
```

##### Code之通用演示，减少阻塞和轮询

从Java8开始引入了CompletableFuture，它是Future的功能增强版，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class cfuture4{
    public static void main(String[] args) throws Exception{
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "-----come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println("-----计算结束耗时1秒钟，result： "+result);
            if(result > 6)
            {
                int age = 10/0;
            }
            return result;
        }).whenComplete((v,e) ->{
            if(e == null)
            {
                System.out.println("-----result: "+v);
            }
        }).exceptionally(e -> {
            System.out.println("-----exception: "+e.getCause()+"\t"+e.getMessage());
            return -44;
        });

        //主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:暂停3秒钟线程
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
    }
}
```

##### CompletableFuture的优点

- 异步任务结束时，会自动回调某个对象的方法；
- 异步任务出错时，会自动回调某个对象的方法；
- 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行

### 电商比价

#### 函数式编程---Lambda +Stream+链式调用+Java8函数式编程带走

![image-20210805151248791](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151248791.png)

![image-20210805151256342](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151256342.png)

![image-20210805151303881](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151303881.png)

![image-20210805151310821](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151310821.png)

![image-20210805151317996](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151317996.png)

![image-20210805151325489](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805151325489.png)

```java
import lombok.Getter;

import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

public class T1{
    static List<NetMall> list = Arrays.asList(
            new NetMall("jd"),
            new NetMall("tmall"),
            new NetMall("pdd"),
            new NetMall("mi")
    );

    public static List<String> findPriceSync(List<NetMall> list,String productName){
        return list.stream().map(mall -> String.format(productName+" %s price is %.2f",mall.getNetMallName(),mall.getPriceByName(productName))).collect(Collectors.toList());
    }

    public static List<String> findPriceASync(List<NetMall> list,String productName){
        return list.stream().map(mall -> CompletableFuture.supplyAsync(() -> String.format(productName + " %s price is %.2f", mall.getNetMallName(), mall.getPriceByName(productName)))).collect(Collectors.toList()).stream().map(CompletableFuture::join).collect(Collectors.toList());
    }


    public static void main(String[] args){
        long startTime = System.currentTimeMillis();
        List<String> list1 = findPriceSync(list, "thinking in java");
        for (String element : list1) {
            System.out.println(element);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒");

        long startTime2 = System.currentTimeMillis();
        List<String> list2 = findPriceASync(list, "thinking in java");
        for (String element : list2) {
            System.out.println(element);
        }
        long endTime2 = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime2 - startTime2) +" 毫秒");
    }
}

class NetMall{
    @Getter
    private String netMallName;

    public NetMall(String netMallName){
        this.netMallName = netMallName;
    }

    public double getPriceByName(String productName){
        return calcPrice(productName);
    }

    private double calcPrice(String productName){
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        return ThreadLocalRandom.current().nextDouble() + productName.charAt(0);
    }
}
```

### CompletableFuture常用方法

#### 1.获得结果和触发计算

- 获取结果

  - public T    get()---不见不散

  - public T    get(long timeout, TimeUnit unit)---过时不候

  - public T    getNow(T valueIfAbsent)

    - 没有计算完成的情况下，给我一个替代结果
    - 立即获取结果不阻塞
      - 计算完，返回计算完成后的结果
      - 没算完，返回设定的valueIfAbsent值

    ```java
    import java.util.concurrent.CompletableFuture;
    import java.util.concurrent.ExecutionException;
    import java.util.concurrent.TimeUnit;
    
    public class CompletableFutureDemo2{
        public static void main(String[] args) throws ExecutionException, InterruptedException{
            CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                return 533;
            });
    
            //去掉注释上面计算没有完成，返回444
            //开启注释上满计算完成，返回计算结果
            try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
    
            System.out.println(completableFuture.getNow(444));
    
    
        }
    }
    ```

  - public T    join()

  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  
  public class CompletableFutureDemo2{
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          System.out.println(CompletableFuture.supplyAsync(() -> "abc").thenApply(r -> r + "123").join());
      }
  }
  ```

- 主动触发计算

  - public boolean complete(T value) 
  - 是否打断get方法立即返回括号值

  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.TimeUnit;
  
  public class CompletableFutureDemo2{
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
              try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
              return 533;
          });
  
          //注释掉暂停线程，get还没有算完只能返回complete方法设置的444；暂停2秒钟线程，异步线程能够计算完成返回get
          try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
  
          //当调用CompletableFuture.get()被阻塞的时候,complete方法就是结束阻塞并get()获取设置的complete里面的值.
          System.out.println(completableFuture.complete(444)+"\t"+completableFuture.get());
  
  
      }
  }
  ```

#### 2.对计算结果进行处理

- thenApply

  - 计算结果存在依赖关系，这两个线程串行化

  ```java
  import java.util.concurrent.*;
  
  public class CompletableFutureDemo2{
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          //当一个线程依赖另一个线程时用 thenApply 方法来把这两个线程串行化,
          CompletableFuture.supplyAsync(() -> {
              //暂停几秒钟线程
              try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
              System.out.println("111");
              return 1024;
          }).thenApply(f -> {
              System.out.println("222");
              return f + 1;
          }).thenApply(f -> {
              //int age = 10/0; // 异常情况：那步出错就停在那步。
              System.out.println("333");
              return f + 1;
          }).whenCompleteAsync((v,e) -> {
              System.out.println("*****v: "+v);
          }).exceptionally(e -> {
              e.printStackTrace();
              return null;
          });
  
          System.out.println("-----主线程结束，END");
  
          // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
          try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
      }
  }
  ```

  - 由于存在依赖关系(当前步错，不走下一步)，当前步骤有异常的话就叫停。

- handle

  ```java
  import lombok.Getter;
  import lombok.Setter;
  
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.TimeUnit;
  
  
  public class CompletableFutureDemo2{
  
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          //当一个线程依赖另一个线程时用 handle 方法来把这两个线程串行化,
          // 异常情况：有异常也可以往下一步走，根据带的异常参数可以进一步处理
          CompletableFuture.supplyAsync(() -> {
              //暂停几秒钟线程
              try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
              System.out.println("111");
              return 1024;
          }).handle((f,e) -> {
              int age = 10/0;
              System.out.println("222");
              return f + 1;
          }).handle((f,e) -> {
              System.out.println("333");
              return f + 1;
          }).whenCompleteAsync((v,e) -> {
              System.out.println("*****v: "+v);
          }).exceptionally(e -> {
              e.printStackTrace();
              return null;
          });
  
          System.out.println("-----主线程结束，END");
  
          // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
          try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
      }
  }
  
  ```

  - 有异常也可以往下一步走，根据带的异常参数可以进一步处理

![image-20210805152628580](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805152628580.png)

![image-20210805152634561](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805152634561.png)

#### 3.对计算结果进行消费

- 接收任务的处理结果，并消费处理，无返回结果
- thenAccept

```java
public static void main(String[] args) throws ExecutionException, InterruptedException{
    CompletableFuture.supplyAsync(() -> {
        return 1;
    }).thenApply(f -> {
        return f + 2;
    }).thenApply(f -> {
        return f + 3;
    }).thenApply(f -> {
        return f + 4;
    }).thenAccept(r -> System.out.println(r));
}
```

- Code之任务之间的顺序执行

  - thenRun
    - thenRun(Runnable runnable)
    - 任务 A 执行完执行 B，并且 B 不需要 A 的结果
  - thenAccept
    - thenAccept(Consumer action)
    - 任务 A 执行完执行 B，B 需要 A 的结果，但是任务 B 无返回值
  - thenApply
    - thenApply(Function fn)
    - 任务 A 执行完执行 B，B 需要 A 的结果，同时任务 B 有返回值

  ```java
  System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenRun(() -> {}).join());
   
  
  System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenAccept(resultA -> {}).join());
  
  
  System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenApply(resultA -> resultA + " resultB").join());
  ```

#### 4.对计算速度进行选用

- 谁快用谁
- applyToEither

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class CompletableFutureDemo2{
    public static void main(String[] args) throws ExecutionException, InterruptedException{
        CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
            return 10;
        });

        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            return 20;
        });

        CompletableFuture<Integer> thenCombineResult = completableFuture1.applyToEither(completableFuture2,f -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
            return f + 1;
        });

        System.out.println(Thread.currentThread().getName() + "\t" + thenCombineResult.get());
    }
}
```

#### 5.对计算结果进行合并

- 两个CompletionStage任务都完成后，最终能把两个任务的结果一起交给thenCombine 来处理

- 先完成的先等着，等待其它分支任务

- thenCombine

  - code标准版，好理解先拆分

  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  
  
  public class CompletableFutureDemo2{
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
              return 10;
          });
  
          CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
              return 20;
          });
  
          CompletableFuture<Integer> thenCombineResult = completableFuture1.thenCombine(completableFuture2, (x, y) -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in ");
              return x + y;
          });
          
          System.out.println(thenCombineResult.get());
      }
  }
  ```

  - code表达式

  ```java
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutionException;
  
  
  public class CompletableFutureDemo2{
      public static void main(String[] args) throws ExecutionException, InterruptedException{
          CompletableFuture<Integer> thenCombineResult = CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in 1");
              return 10;
          }).thenCombine(CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in 2");
              return 20;
          }), (x,y) -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in 3");
              return x + y;
          }).thenCombine(CompletableFuture.supplyAsync(() -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in 4");
              return 30;
          }),(a,b) -> {
              System.out.println(Thread.currentThread().getName() + "\t" + "---come in 5");
              return a + b;
          });
          System.out.println("-----主线程结束，END");
          System.out.println(thenCombineResult.get());
  
  
          // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
          try { TimeUnit.SECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
      }
  }
  ```

## Java锁

### 乐观锁和悲观锁

#### 悲观锁

认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。synchronized关键字和Lock的实现类都是悲观锁

- 适合写操作多的场景，先加锁可以保证写操作时数据正确。
- 显式的锁定之后再操作同步资源

#### 伪代码说明

```java
 
//=============悲观锁的调用方式
public synchronized void m1(){
    //加锁后的业务逻辑......
}

// 保证多个线程使用的是同一个lock对象的前提下
ReentrantLock lock = new ReentrantLock();
public void m2() {
    lock.lock();
    try {
        // 操作同步资源
    }finally {
        lock.unlock();
    }
}

//=============乐观锁的调用方式
// 保证多个线程使用的是同一个AtomicInteger
private AtomicInteger atomicInteger = new AtomicInteger();
atomicInteger.incrementAndGet();
```

#### 乐观锁

- 适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。
- 乐观锁则直接去操作同步资源，是一种无锁算法，得之我幸不得我命，再抢
- 乐观锁一般有两种实现方式：
  - 采用版本号机制
  - CAS（Compare-and-Swap，即比较并替换）算法实现

### 8种情况演示锁运行案例

