# JUC并发编程与源码分析



## 基础知识

![image-20210805143653369](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805143653369.png)

### juc口诀

- 高内聚低耦合前提下，封装思想
  - 线程
  - 操作
  - 资源来
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

![image-20210805170751369](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805170751369.png)

#### synchronized有三种应用方式

- JDK源码(notify方法)说明举例![image-20210805170851515](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805170851515.png)
- 8种锁的案例实际体现在3个地方
  - 作用于实例方法，当前实例加锁，进入同步代码前要获得当前实例的锁；
  - 作用于代码块，对括号里配置的对象加锁。
  - 作用于静态方法，当前类加锁，进去同步代码前要获得当前类对象的锁；

#### 从字节码角度分析synchronized实现

- javap -c ***.class文件反编译

  -  -c                       对代码进行反汇编
  - javap -v ***.class文件反编译
  -  -v  -verbose             输出附加信息（包括行号、本地变量表，反汇编等详细信息）

- synchronized同步代码块

  - javap -c ***.class文件反编译

  - 反编译![image-20210805171148740](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171148740.png)

    ![image-20210805171218348](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171218348.png)

  - synchronized同步代码块 --- 实现使用的是monitorenter和monitorexit指令

  - 一定是一个enter两个exit吗？![image-20210805171326300](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171326300.png)

- synchronized普通同步方法

  - javap -v ***.class文件反编译
  - ![image-20210805171408418](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171408418.png)
  - synchronized普通同步方法

  调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置。如果设置了，执行线程会将先持有monitor然后再执行方法，最后在方法完成(无论是正常完成还是非正常完成)时释放 monitor

- synchronized静态同步方法

  - javap -v ***.class文件反编译
  - ![image-20210805171518475](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171518475.png)
  - synchronized静态同步方法 --- ACC_STATIC, ACC_SYNCHRONIZED访问标志区分该方法是否静态同步方法

#### 反编译synchronized锁的是什么

- 什么是管程monitor

管程 (英语：Monitors，也称为监视器) 是一种程序结构，结构内的多个子程序（对象或模块）形成的多个工作线程互斥访问共享资源。
这些共享资源一般是硬件设备或一群变量。对共享变量能够进行的所有操作集中在一个模块中。（把信号量及其操作原语“封装”在一个对象内部）管程实现了在一个时间点，最多只有一个线程在执行管程的某个子程序。管程提供了一种机制，管程可以看做一个软件模块，它是将共享的变量和对于这些共享变量的操作封装起来，形成一个具有一定接口的功能模块，进程可以调用管程来实现进程级别的并发控制。

![image-20210805171643264](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171643264.png)

- 在HotSpot虚拟机中，monitor采用ObjectMonitor实现

- C++源码解读

  - ObjectMonitor.java→ObjectMonitor.cpp→objectMonitor.hpp

  - objectMonitor.hpp![image-20210805171735570](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171735570.png)

    ![image-20210805171758225](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171758225.png)

  - 每个对象天生都带着一个对象监视器

 synchronized必须作用于某个对象中，所以Java在对象的头文件存储了锁的相关信息。锁升级功能主要依赖于 MarkWord 中的锁标志位和释放偏向锁标志位

![image-20210805171957696](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805171957696.png)

### 公平锁和非公平锁

- 从ReentrantLock卖票编码演示公平和非公平现象

```java
import java.util.concurrent.locks.ReentrantLock;

class Ticket{
    private int number = 30;
    ReentrantLock lock = new ReentrantLock();

    public void sale(){
        lock.lock();
        try{
            if(number > 0){
                System.out.println(Thread.currentThread().getName()+"卖出第：\t"+(number--)+"\t 还剩下:"+number);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}



public class SaleTicketDemo{
    public static void main(String[] args){
        Ticket ticket = new Ticket();

        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"a").start();
        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"b").start();
        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"c").start();
    }
}
```

- 何为公平锁/非公平锁?---⽣活中，排队讲求先来后到视为公平。程序中的公平性也是符合请求锁的绝对时间的，其实就是 FIFO，否则视为不公平

  - 源码解读 --- 按序排队公平锁，就是判断同步队列是否还有先驱节点的存在(我前面还有人吗?)，如果没有先驱节点才能获取锁；
    先占先得非公平锁，是不管这个事的，只要能抢获到同步状态就可以![image-20210805172320215](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805172320215.png)
  - 为什么会有公平锁/非公平锁的设计为什么默认非公平？

  ```java
   
  1
  恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从CPU的角度来看，这个时间差存在的还是很明显的。所以非公平锁能更充分的利用CPU 的时间片，尽量减少 CPU 空闲状态时间。
   
  2
  使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当1个线程请求锁获取同步状态，然后释放同步状态，因为不需要考虑是否还有前驱节点，所以刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大，所以就减少了线程的开销。
  
  ```

  - 使⽤公平锁会有什么问题 ---  公平锁保证了排队的公平性，非公平锁霸气的忽视这个规则，所以就有可能导致排队的长时间在排队，也没有机会获取到锁，这就是传说中的 “锁饥饿”
  - 什么时候用公平？什么时候用非公平？ ---  如果为了更高的吞吐量，很显然非公平锁是比较合适的，因为节省很多线程切换时间，吞吐量自然就上去了；否则那就用公平锁，大家公平使用。

- 预埋伏AQS

![image-20210805172557641](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805172557641.png)

![image-20210805172625128](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805172625128.png)

### 可重入锁(又名递归锁)


可重入锁又名递归锁

是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁(前提，锁对象得是同一个对象)，不会因为之前已经获取过还没释放而阻塞。

如果是1个有 synchronized 修饰的递归调用方法，程序第2次进入被自己阻塞了岂不是天大的笑话，出现了作茧自缚。
所以Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

- 可：可以。
- 重：再次。
- 入：进入。(进入同步域（即同步代码块/方法或显式锁锁定的代码）)
- 锁：同步锁。

一个线程中的多个流程可以获取同一把锁，持有这把同步锁可以再次进入。自己可以获取自己的内部锁

#### 可重入锁种类

##### 1.隐式锁（即synchronized关键字使用的锁）默认是可重入锁

指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁，这样的锁就叫做可重入锁。
简单的来说就是：在一个synchronized修饰的方法或代码块的内部调用本类的其他synchronized修饰的方法或代码块时，是永远可以得到锁的


与可重入锁相反，不可重入锁不可递归调用，递归调用就发生死锁。

```java

/**
 * 同步块
 */
public class ReEntryLockDemo{
    public static void main(String[] args){
        final Object objectLockA = new Object();

        new Thread(() -> {
            synchronized (objectLockA){
                System.out.println("-----外层调用");
                synchronized (objectLockA){
                    System.out.println("-----中层调用");
                    synchronized (objectLockA){
                        System.out.println("-----内层调用");
                    }
                }
            }
        },"a").start();
    }
}
```

```java

/**
 * 同步方法
 * 在一个Synchronized修饰的方法或代码块的内部调用本类的其他Synchronized修饰的方法或代码块时，是永远可以得到锁的
 */
public class ReEntryLockDemo{
    public synchronized void m1(){
        System.out.println("-----m1");
        m2();
    }
    public synchronized void m2(){
        System.out.println("-----m2");
        m3();
    }
    public synchronized void m3(){
        System.out.println("-----m3");
    }

    public static void main(String[] args){
        ReEntryLockDemo reEntryLockDemo = new ReEntryLockDemo();

        reEntryLockDemo.m1();
    }
}
```

##### 2.Synchronized的重入的实现机理


每个锁对象拥有一个锁计数器和一个指向持有该锁的线程的指针。

当执行monitorenter时，如果目标锁对象的计数器为零，那么说明它没有被其他线程所持有，Java虚拟机会将该锁对象的持有线程设置为当前线程，并且将其计数器加1。

在目标锁对象的计数器不为零的情况下，如果锁对象的持有线程是当前线程，那么 Java 虚拟机可以将其计数器加1，否则需要等待，直至持有线程释放该锁。

当执行monitorexit时，Java虚拟机则需将锁对象的计数器减1。计数器为零代表锁已被释放。

##### 3.显式锁（即Lock）也有ReentrantLock这样的可重入锁。

```java
package com.atguigu.juc.senior.prepare;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 在一个Synchronized修饰的方法或代码块的内部调用本类的其他Synchronized修饰的方法或代码块时，是永远可以得到锁的
 */
public class ReEntryLockDemo{
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        new Thread(() -> {
            lock.lock();
            try{
                System.out.println("----外层调用lock");
                lock.lock();
                try{
                    System.out.println("----内层调用lock");
                }finally {
                    // 这里故意注释，实现加锁次数和释放次数不一样
                    // 由于加锁次数和释放次数不一样，第二个线程始终无法获取到锁，导致一直在等待。
                    lock.unlock(); // 正常情况，加锁几次就要解锁几次
                }
            }finally {
                lock.unlock();
            }
        },"a").start();

        new Thread(() -> {
            lock.lock();
            try{
                System.out.println("b thread----外层调用lock");
            }finally {
                lock.unlock();
            }
        },"b").start();

    }
}
```

### 死锁及排查

死锁是指两个或两个以上的线程在执行过程中,因争夺资源而造成的一种互相等待的现象,若无外力干涉那它们都将无法推进下去，如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

![image-20210805173249888](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805173249888.png)

产生死锁主要原因:

- 系统资源不足
- 进程运行推进的顺序不合适
- 资源分配不当

#### 死锁代码case

```java
import java.util.concurrent.TimeUnit;

/**
 */
public class DeadLockDemo{
    public static void main(String[] args){
        final Object objectLockA = new Object();
        final Object objectLockB = new Object();

        new Thread(() -> {
            synchronized (objectLockA){
                System.out.println(Thread.currentThread().getName()+"\t"+"自己持有A，希望获得B");
                //暂停几秒钟线程
                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                synchronized (objectLockB){
                    System.out.println(Thread.currentThread().getName()+"\t"+"A-------已经获得B");
                }
            }
        },"A").start();

        new Thread(() -> {
            synchronized (objectLockB){
                System.out.println(Thread.currentThread().getName()+"\t"+"自己持有B，希望获得A");
                //暂停几秒钟线程
                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                synchronized (objectLockA){
                    System.out.println(Thread.currentThread().getName()+"\t"+"B-------已经获得A");
                }
            }
        },"B").start();

    }
}
```

#### 排查死锁

- 纯命令
  - jps -l
  - jstack 进程编号
- 图形化
  - jconsole

## LockSupport与线程中断

### 线程中断机制

![image-20210805221440659](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805221440659.png)

#### 什么是中断？

首先
一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止。
所以，Thread.stop, Thread.suspend, Thread.resume 都已经被废弃了。

其次
在Java中没有办法立即停止一条线程，然而停止线程却显得尤为重要，如取消一个耗时操作。
因此，Java提供了一种用于停止线程的机制——中断。

中断只是一种协作机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。
若要中断一个线程，你需要手动调用该线程的interrupt方法，该方法也仅仅是将线程对象的中断标识设成true；
接着你需要自己写代码不断地检测当前线程的标识位，如果为true，表示别的线程要求这条线程中断，
此时究竟该做什么需要你自己写代码实现。

每个线程对象中都有一个标识，用于表示线程是否被中断；该标识位为true表示中断，为false表示未中断；
通过调用线程对象的interrupt方法将该线程的标识位设为true；可以在别的线程中调用，也可以在自己的线程中调用。

#### 中断的相关API方法

![image-20210805221559015](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805221559015.png)

| public void interrupt()             | 实例方法，实例方法interrupt()仅仅是设置线程的中断状态为true，不会停止线程 |
| ----------------------------------- | ------------------------------------------------------------ |
| public static boolean interrupted() | 静态方法，Thread.interrupted();  <br/>判断线程是否被中断，并清除当前中断状态<br/>这个方法做了两件事：<br/>1 返回当前线程的中断状态<br/>2 将当前线程的中断状态设为false<br/> 这个方法有点不好理解，因为连续调用两次的结果可能不一样。 |
| public boolean isInterrupted()      | 实例方法，判断当前线程是否被中断（通过检查中断标志位）       |

#### 面试题：如何使用中断标识停止线程？

##### 在需要中断的线程中不断监听中断状态，一旦发生中断，就执行相应的中断处理业务逻辑。

- 修改状态
- 停止程序的运行

##### 方法

- 通过一个volatile变量实现

```java
import java.util.concurrent.TimeUnit;


public class InterruptDemo{
private static volatile boolean isStop = false;

public static void main(String[] args){
    new Thread(() -> {
        while(true){
            if(isStop)
            {
                System.out.println(Thread.currentThread().getName()+"线程------isStop = true,自己退出了");
                break;
            }
            System.out.println("-------hello interrupt");
        }
    },"t1").start();

    //暂停几秒钟线程
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    isStop = true;
}

}
```

- 通过AtomicBoolean

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;


public class StopThreadDemo{
    private final static AtomicBoolean atomicBoolean = new AtomicBoolean(true);

    public static void main(String[] args){
        Thread t1 = new Thread(() -> {
            while(atomicBoolean.get())
            {
                try { TimeUnit.MILLISECONDS.sleep(500); } catch (InterruptedException e) { e.printStackTrace(); }
                System.out.println("-----hello");
            }
        }, "t1");
        t1.start();

        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

        atomicBoolean.set(false);
    }
}
```

- 通过Thread类自带的中断api方法实现

  - 实例方法interrupt()，没有返回值![image-20210805222140147](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222140147.png)

    | public void interrupt() | 实例方法，<br/>调用interrupt()方法仅仅是在当前线程中打了一个停止的标记，并不是真正立刻停止线程。 |
    | ----------------------- | ------------------------------------------------------------ |

    ![image-20210805222306630](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222306630.png)

    ![image-20210805222328256](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222328256.png)

    - 实例方法isInterrupted，返回布尔值

    ![image-20210805222509207](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222509207.png)

    | public boolean isInterrupted() | 实例方法，<br/>获取中断标志位的当前值是什么，<br/>判断当前线程是否被中断（通过检查中断标志位），默认是false |
    | ------------------------------ | ------------------------------------------------------------ |

    ![image-20210805222549355](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222549355.png)

    ```java
    import java.util.concurrent.TimeUnit;
    
    
    public class InterruptDemo{
        public static void main(String[] args){
            Thread t1 = new Thread(() -> {
                while(true){
                    if(Thread.currentThread().isInterrupted())
                    {
                        System.out.println("-----t1 线程被中断了，break，程序结束");
                        break;
                    }
                    System.out.println("-----hello");
                }
            }, "t1");
            t1.start();
    
            System.out.println("**************"+t1.isInterrupted());
            //暂停5毫秒
            try { TimeUnit.MILLISECONDS.sleep(5); } catch (InterruptedException e{ e.printStackTrace(); }
            t1.interrupt();
            System.out.println("**************"+t1.isInterrupted());
        }
    }
    ```

  ##### 当前线程的中断标识为true，是不是就立刻停止？

  具体来说，当对一个线程，调用 interrupt() 时：

  ①  如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。
  被设置中断标志的线程将继续正常运行，不受影响。所以， interrupt() 并不能真正的中断线程，需要被调用的线程自己进行配合才行。


  ②  如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），在别的线程中调用当前线程对象的interrupt方法，
  那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。

   ```java
   import java.util.concurrent.TimeUnit;
   
   
   public class InterruptDemo2{
       public static void main(String[] args) throws InterruptedException{
           Thread t1 = new Thread(() -> {
               for (int i=0;i<300;i++) {
                   System.out.println("-------"+i);
               }
               System.out.println("after t1.interrupt()--第2次---: "+Thread.currentThread().isInterrupted());
           },"t1");
           t1.start();
   
           System.out.println("before t1.interrupt()----: "+t1.isInterrupted());
           //实例方法interrupt()仅仅是设置线程的中断状态位设置为true，不会停止线程
           t1.interrupt();
           //活动状态,t1线程还在执行中
           try { TimeUnit.MILLISECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
           System.out.println("after t1.interrupt()--第1次---: "+t1.isInterrupted());
           //非活动状态,t1线程不在执行中，已经结束执行了。
           try { TimeUnit.MILLISECONDS.sleep(3000); } catch (InterruptedException e) { e.printStackTrace(); }
           System.out.println("after t1.interrupt()--第3次---: "+t1.isInterrupted());
       }
   }
   ```

  ![image-20210805222907558](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222907558.png)

  ![image-20210805222917593](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805222917593.png)

中断只是一种协同机制，修改中断标识位仅此而已，不是立刻stop打断

##### 静态方法Thread.interrupted()

###### 静态方法Thread.interrupted()

```java

/**
 * 作用是测试当前线程是否被中断（检查中断标志），返回一个boolean并清除中断状态，
 * 第二次再调用时中断状态已经被清除，将返回一个false。
 */
public class InterruptDemo{

    public static void main(String[] args) throws InterruptedException{
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println("111111");
        Thread.currentThread().interrupt();
        System.out.println("222222");
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
    }
}
```

| public static boolean interrupted() | 静态方法，Thread.interrupted();  <br/>判断线程是否被中断，并清除当前中断状态，类似i++<br/>这个方法做了两件事：<br/>1 返回当前线程的中断状态<br/>2 将当前线程的中断状态设为false<br/> <br/>这个方法有点不好理解，因为连续调用两次的结果可能不一样。 |
| ----------------------------------- | ------------------------------------------------------------ |

![image-20210805223226438](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223226438.png)

###### 都会返回中断状态，两者对比

![image-20210805223245539](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223245539.png)

![image-20210805223357650](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223357650.png)

![image-20210805223405390](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223405390.png)

![image-20210805223415322](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223415322.png)

方法的注释也清晰的表达了“中断状态将会根据传入的ClearInterrupted参数值确定是否重置”。

所以，
静态方法interrupted将       会清除中断状态（传入的参数ClearInterrupted为true），

实例方法isInterrupted则不会（传入的参数ClearInterrupted为false）。


线程中断相关的方法：

interrupt()方法是一个实例方法
它通知目标线程中断，也就是设置目标线程的中断标志位为true，中断标志位表示当前线程已经被中断了。

isInterrupted()方法也是一个实例方法
它判断当前线程是否被中断（通过检查中断标志位）并获取中断标志

Thread类的静态方法interrupted()
返回当前线程的中断状态(boolean类型)且将当前线程的中断状态设为false，此方法调用之后会清除当前线程的中断标志位的状态（将中断标志置为false了），返回当前值并清零置false

### LockSupport是什么

![image-20210805223533474](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223533474.png)

![image-20210805223554795](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223554795.png)

![image-20210805223606021](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210805223606021.png)

LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

下面这句话，后面详细说
LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程

### 线程等待唤醒机制

#### 3种让线程等待和唤醒的方法

方式1：使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程

方式2：使用JUC包中Condition的await()方法让线程等待，使用signal()方法唤醒线程

方式3：LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程

#### Object类中的wait和notify方法实现线程等待和唤醒

```java
import java.util.concurrent.TimeUnit;

/**
 *
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 *
 * 1 正常程序演示
 *
 * 以下异常情况：
 * 2 wait方法和notify方法，两个都去掉同步代码块后看运行效果
 *   2.1 异常情况
 *   Exception in thread "t1" java.lang.IllegalMonitorStateException at java.lang.Object.wait(Native Method)
 *   Exception in thread "t2" java.lang.IllegalMonitorStateException at java.lang.Object.notify(Native Method)
 *   2.2 结论
 *   Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行（必须用到关键字synchronized）。
 *
 * 3 将notify放在wait方法前面
 *   3.1 程序一直无法结束
 *   3.2 结论
 *   先wait后notify、notifyall方法，等待中的线程才会被唤醒，否则无法唤醒
 */
public class LockSupportDemo{

    public static void main(String[] args)//main方法，主线程一切程序入口{
        Object objectLock = new Object(); //同一把锁，类似资源类

        new Thread(() -> {
            synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            synchronized (objectLock) {
                objectLock.notify();
            }

            //objectLock.notify();

            /*synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }*/
        },"t2").start();

    }
}
```

##### 正常

```java
import java.util.concurrent.TimeUnit;

/**
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 *
 * 1 正常程序演示
 *
 */
public class LockSupportDemo{
    public static void main(String[] args)//main方法，主线程一切程序入口{
        Object objectLock = new Object(); //同一把锁，类似资源类

        new Thread(() -> {
            synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            synchronized (objectLock) {
                objectLock.notify();
            }
        },"t2").start();
    }
}
```

##### 异常

```java
import java.util.concurrent.TimeUnit;

/**
 *
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 * 以下异常情况：
 * 2 wait方法和notify方法，两个都去掉同步代码块后看运行效果
 *   2.1 异常情况
 *   Exception in thread "t1" java.lang.IllegalMonitorStateException at java.lang.Object.wait(Native Method)
 *   Exception in thread "t2" java.lang.IllegalMonitorStateException at java.lang.Object.notify(Native Method)
 *   2.2 结论
 *   Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行（必须用到关键字synchronized）。
 */
public class LockSupportDemo{

    public static void main(String[] args)//main方法，主线程一切程序入口{
        Object objectLock = new Object(); //同一把锁，类似资源类

        new Thread(() -> {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            objectLock.notify();
        },"t2").start();
    }
}
```

wait方法和notify方法，两个都去掉同步代码块

![image-20210806101104959](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101104959.png)

##### 异常

```java
import java.util.concurrent.TimeUnit;

/**
 *
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 *
 * 3 将notify放在wait方法前先执行，t1先notify了，3秒钟后t2线程再执行wait方法
 *   3.1 程序一直无法结束
 *   3.2 结论
 *   先wait后notify、notifyall方法，等待中的线程才会被唤醒，否则无法唤醒
 */
public class LockSupportDemo{

    public static void main(String[] args)//main方法，主线程一切程序入口{
        Object objectLock = new Object(); //同一把锁，类似资源类

        new Thread(() -> {
            synchronized (objectLock) {
                objectLock.notify();
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"通知了");
        },"t1").start();

        //t1先notify了，3秒钟后t2线程再执行wait方法
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t2").start();
    }
}
```

将notify放在wait方法前面

程序无法执行，无法唤醒

总结：

- wait和notify方法必须要在同步块或者方法里面，且成对出现使用
- 先wait后notify才OK

#### Condition接口中的await后signal方法实现线程的等待和唤醒

##### 正常

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 */
public class LockSupportDemo2{
    public static void main(String[] args){
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            lock.lock();
            try{
                System.out.println(Thread.currentThread().getName()+"\t"+"start");
                condition.await();
                System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            lock.lock();
            try{
                condition.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"通知了");
        },"t2").start();

    }
}
```

##### 异常

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 异常：
 * condition.await();和condition.signal();都触发了IllegalMonitorStateException异常
 *
 * 原因：调用condition中线程等待和唤醒的方法的前提是，要在lock和unlock方法中,要有锁才能调用
 */
public class LockSupportDemo2{
    public static void main(String[] args){
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            try{
                System.out.println(Thread.currentThread().getName()+"\t"+"start");
                condition.await();
                System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            try{
                condition.signal();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"通知了");
        },"t2").start();

    }
}
```

去掉lock/unlock

![image-20210806101516987](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101516987.png)

**condition.await();和 condition.signal();都触发了 IllegalMonitorStateException异常。**

**结论：**
**lock、unlock对里面才能正确调用调用condition中线程等待和唤醒的方法**

##### 异常

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 异常：
 * 程序无法运行
 *
 * 原因：先await()后signal才OK，否则线程无法被唤醒
 */
public class LockSupportDemo2{
    public static void main(String[] args){
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            lock.lock();
            try{
                condition.signal();
                System.out.println(Thread.currentThread().getName()+"\t"+"signal");
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            lock.lock();
            try{
                System.out.println(Thread.currentThread().getName()+"\t"+"等待被唤醒");
                condition.await();
                System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒");
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        },"t2").start();

    }
}
```

先signal后await

**总结：**

- **Condtion中的线程等待和唤醒方法之前，需要先获取锁**
- **一定要先await后signal，不要反了**

#### Object和Condition使用的限制条件

- **线程先要获得并持有锁，必须在锁块(synchronized或lock)中**
- **必须要先等待后唤醒，线程才能够被唤醒**

#### LockSupport类中的park等待和unpark唤醒

通过park()和unpark(thread)方法来实现阻塞和唤醒线程的操作

![image-20210806101752091](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101752091.png)

**LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。**

**LockSupport类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能， 每个线程都有一个许可(permit)，**
**permit只有两个值1和零，默认是零。**
**可以把许可看成是一种(0,1)信号量（Semaphore），但与 Semaphore 不同的是，许可的累加上限是1。**

##### 主要方法

![image-20210806101822162](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101822162.png)

###### 阻塞

park() /park(Object blocker) 

调用LockSupport.park()时

![image-20210806101901725](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101901725.png)

permit默认是零，所以一开始调用park()方法，当前线程就会阻塞，直到别的线程将当前线程的permit设置为1时，park方法会被唤醒，
然后会将permit再次设置为零并返回。

阻塞当前线程/阻塞传入的具体线程

###### 唤醒

unpark(Thread thread) 

LockSupport.unpark(thread);

![image-20210806101933824](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806101933824.png)

调用unpark(thread)方法后，就会将thread线程的许可permit设置成1(注意多次调用unpark方法，不会累加，permit值还是1)会自动唤醒thread线程，即之前阻塞中的LockSupport.park()方法会立即返回。

唤醒处于阻塞状态的指定线程

###### 正常+无锁块要求

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

/**
 */
public class LockSupportDemo3{
    public static void main(String[] args){
        //正常使用+不需要锁块
Thread t1 = new Thread(() -> {
    System.out.println(Thread.currentThread().getName()+" "+"1111111111111");
    LockSupport.park();
    System.out.println(Thread.currentThread().getName()+" "+"2222222222222------end被唤醒");
},"t1");
t1.start();

//暂停几秒钟线程
try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

LockSupport.unpark(t1);
System.out.println(Thread.currentThread().getName()+"   -----LockSupport.unparrk() invoked over");

    }
}
```

###### 之前错误的先唤醒后等待，LockSupport照样支持

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

/**
 */
public class T1{
    public static void main(String[] args){
        Thread t1 = new Thread(() -> {
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis());
            LockSupport.park();
            System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis()+"---被叫醒");
        },"t1");
        t1.start();

        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        LockSupport.unpark(t1);
        System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis()+"---unpark over");
    }
}
```

![image-20210806102109963](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806102109963.png)

成双成对要牢记

## Java内存模型之JMM

### 计算机硬件存储体系

 计算机存储结构，从本地磁盘到主存到CPU缓存，也就是从硬盘到内存，到CPU。
一般对应的程序的操作就是从数据库查数据到内存然后到CPU进行计算

![image-20210806103103743](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103103743.png)

因为有这么多级的缓存(cpu和物理主内存的速度不一致的)，CPU的运行并不是直接操作内存而是先把内存里边的数据读到缓存，而内存的读和写操作的时候就会造成不一致的问题

![image-20210806103124902](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103124902.png)

Java虚拟机规范中试图定义一种Java内存模型（java Memory Model，简称JMM) 来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。推导出我们需要知道JMM

### Java内存模型Java Memory Model


JMM(Java内存模型Java Memory Model，简称JMM)本身是一种抽象的概念并不真实存在它仅仅描述的是一组约定或规范，通过这组规范定义了程序中(尤其是多线程)各个变量的读写访问方式并决定一个线程对共享变量的写入何时以及如何变成对另一个线程可见，关键技术点都是围绕多线程的原子性、可见性和有序性展开的。

 原则：
 JMM的关键技术点都是围绕多线程的原子性、可见性和有序性展开的

能干嘛？
1 通过JMM来实现线程和主内存之间的抽象关系。
2 屏蔽各个硬件平台和操作系统的内存访问差异以实现让Java程序在各种平台下都能达到一致的内存访问效果。

### JMM规范下，三大特性

#### 可见性

**可见性**
是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道该变更 ，JMM规定了所有的变量都存储在主内存中。

![image-20210806103250434](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103250434.png)

![image-20210806103311849](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103311849.png)

Java中普通的共享变量不保证可见性，因为数据修改被写入内存的时机是不确定的，多线程并发下很可能出现"脏读"，所以每个线程都有自己的工作内存，线程自己的工作内存中保存了该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取，赋值等 ）都必需在线程自己的工作内存中进行，而不能够直接读写主内存中的变量。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成

![image-20210806103331694](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103331694.png)

 线程脏读：如果没有可见性保证
1.主内存中有变量 x，初始值为 0
2.线程 A 要将 x 加 1，先将 x=0 拷贝到自己的私有内存中，然后更新 x 的值
3.线程 A 将更新后的 x 值回刷到主内存的时间是不固定的
4.刚好在线程 A 没有回刷 x 到主内存时，线程 B 同样从主内存中读取 x，此时为 0，和线程 A 一样的操作，最后期盼的 x=2 就会变成 x=1

![image-20210806103409433](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103409433.png)

#### 原子性

指一个操作是不可中断的，即多线程环境下，操作不能被其他线程干扰

#### 有序性

对于一个线程的执行代码而言，我们总是习惯性认为代码的执行总是从上到下，有序执行。
但为了提供性能，编译器和处理器通常会对指令序列进行重新排序。
指令重排可以保证串行语义一致，但没有义务保证多线程间的语义也一致，即可能产生"脏读"，简单说，
两行以上不相干的代码在执行的时候有可能先执行的不是第一条，不见得是从上到下顺序执行，执行顺序会被优化。

![image-20210806103517083](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103517083.png)

单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。
处理器在进行重排序时必须要考虑指令之间的数据依赖性
多线程环境中线程交替执行,由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的,结果无法预测

### JMM规范下，多线程对变量的读写过程

读取过程

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝到的线程自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的变量副本拷贝，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成，其简要访问过程如下图:

![image-20210806103624615](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806103624615.png)

**JMM定义了线程和主内存之间的抽象关系**
1 线程之间的共享变量存储在主内存中(从硬件角度来说就是内存条)
2 每个线程都有一个私有的本地工作内存，本地工作内存中存储了该线程用来读/写共享变量的副本(从硬件角度来说就是CPU的缓存，比如寄存器、L1、L2、L3缓存等)

**总结：**

- **我们定义的所有共享变量都储存在物理主内存中**
- **每个线程都有自己独立的工作内存，里面保存该线程使用到的变量的副本(主内存中该变量的一份拷贝)**
- **线程对共享变量所有的操作都必须先在线程自己的工作内存中进行后写回主内存，不能直接从主内存中读写(不能越级)**
- **不同线程之间也无法直接访问其他线程的工作内存中的变量，线程间变量值的传递需要通过主内存来进行(同级不能相互访问)**

### JMM规范下，多线程先行发生原则之happens-before

在JMM中，如果一个操作执行的结果需要对另一个操作可见性或者 代码重排序，那么这两个操作之间必须存在happens-before关系。

| x = 5              | 线程A执行 |
| ------------------ | --------- |
| y = x              | 线程B执行 |
| 上述称之为：写后读 |           |

| y是否等于5呢？<br/> <br/>如果线程A的操作（x= 5）happens-before(先行发生)线程B的操作（y = x）,那么可以确定线程B执行后y = 5 一定成立;<br/> <br/>如果他们不存在happens-before原则，那么y = 5 不一定成立。<br/> <br/>这就是happens-before原则的威力。-------------------》包含可见性和有序性的约束 |
| ------------------------------------------------------------ |

#### 先行发生原则说明

如果Java内存模型中所有的有序性都仅靠volatile和synchronized来完成，那么有很多操作都将会变得非常啰嗦，
但是我们在编写Java并发代码的时候并没有察觉到这一点。

我们没有时时、处处、次次，添加volatile和synchronized来完成程序，这是因为Java语言中JMM原则下
有一个“先行发生”(Happens-Before)的原则限制和规矩

这个原则非常重要： 
它是判断数据是否存在竞争，线程是否安全的非常有用的手段。依赖这个原则，我们可以通过几条简单规则一揽子解决并发环境下两个操
作之间是否可能存在冲突的所有问题，而不需要陷入Java内存模型苦涩难懂的底层编译原理之中。

#### happens-before总原则

- 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，
  而且第一个操作的执行顺序排在第二个操作之前。
- 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。
  如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。

#### happens-before之8条

1.次序规则：

- 一个线程内，按照代码顺序，写在前面的操作先行发生于写在后面的操作；
- 前一个操作的结果可以被后续的操作获取。
  讲白点就是前面一个操作把变量X赋值为1，那后面一个操作肯定能知道X已经变成了1。

2.锁定规则：

一个unLock操作先行发生于后面((这里的“后面”是指时间上的先后))对同一个锁的lock操作；

```java

/**
 */
public class HappenBeforeDemo{
    static Object objectLock = new Object();

    public static void main(String[] args) throws InterruptedException{
        //对于同一把锁objectLock，threadA一定先unlock同一把锁后B才能获得该锁，   A 先行发生于B
        synchronized (objectLock)
        {

        }
    }
}
```

3.volatile变量规则：

对一个volatile变量的写操作先行发生于后面对这个变量的读操作，前面的写对后面的读是可见的，这里的“后面”同样是指时间上的先后。

4.传递规则：

如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；

5.线程启动规则(Thread Start Rule)：

Thread对象的start()方法先行发生于此线程的每一个动作

6.线程中断规则(Thread Interruption Rule)：

- 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；
- 可以通过Thread.interrupted()检测到是否发生中断

7.线程终止规则(Thread Termination Rule)：

线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread::join()方法是否结束、
Thread::isAlive()的返回值等手段检测线程是否已经终止执行。

8.对象终结规则(Finalizer Rule)：

一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始(对象没有完成初始化之前，是不能调用finalized()方法的)

![image-20210806104422760](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104422760.png)

| 假设存在线程A和B，<br/> <br/>线程A先（时间上的先后）调用了setValue(1)，<br/> <br/>然后线程B调用了同一个对象的getValue()，<br/> <br/>那么线程B收到的返回值是什么？ |
| ------------------------------------------------------------ |

| 我们就这段简单的代码一次分析happens-before的规则（规则5、6、7、8 可以忽略，因为他们和这段代码毫无关系）：<br/>1 由于两个方法是由不同的线程调用，不在同一个线程中，所以肯定不满足程序次序规则；<br/>2 两个方法都没有使用锁，所以不满足锁定规则；<br/>3 变量不是用volatile修饰的，所以volatile变量规则不满足；<br/>4 传递规则肯定不满足； |
| ------------------------------------------------------------ |

| 所以我们无法通过happens-before原则推导出线程A happens-before线程B，虽然可以确认在时间上线程A优先于线程B指定，<br/>但就是无法确认线程B获得的结果是什么，所以这段代码不是线程安全的。那么怎么修复这段代码呢？ |
| ------------------------------------------------------------ |

- 把getter/setter方法都定义为synchronized方法
- 把value定义为volatile变量，由于setter方法对value的修改不依赖value的原值，满足volatile关键字使用场景

## volatile与Java内存模型

### 被volatile修改的变量有2大特点

- 可见性
- 有序性（排序要求）

volatile的内存语义：

- 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内存中。
- 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，直接从主内存中读取共享变量
- 所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取。

### 内存屏障

内存屏障（也称内存栅栏，内存栅障，屏障指令等，是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作），避免代码重排序。内存屏障其实就是一种JVM指令，Java内存模型的重排规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令，通过这些内存屏障指令，volatile实现了Java内存模型中的可见性和有序性，但volatile无法保证原子性。

内存屏障之前的所有写操作都要回写到主内存，
内存屏障之后的所有读操作都能获得内存屏障之前的所有写操作的最新结果(实现了可见性)。

![image-20210806104808221](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104808221.png)因此重排序时，不允许把内存屏障之后的指令重排序到内存屏障之前。
一句话：对一个 volatile 域的写, happens-before 于任意后续对这个 volatile 域的读，也叫写后读。

#### JVM中提供了四类内存屏障指令

##### C++源码分析

Unsafe.class

![image-20210806104914192](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104914192.png)

Unsafe.java

![image-20210806104928928](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104928928.png)

Unsafe.cpp

![image-20210806104942815](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104942815.png)

OrderAccess.hpp

![image-20210806104953608](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806104953608.png)

orderAccess_linux_x86.inline.hpp

![image-20210806105004581](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105004581.png)

#### 四大屏障分别是什么意思

![image-20210806105027870](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105027870.png)

![image-20210806105004581](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105004581.png)

#### happens-before 之 volatile 变量规则

| 第一个操作 | 第二个操作：普通写 | 第二个操作：volatile读 | 第二个操作：volatile写 |
| ---------- | ------------------ | ---------------------- | ---------------------- |
| 普通读写   | 可以重排           | 可以重排               | 不可以重排             |
| volatile读 | 不可以重排         | 不可以重排             | 不可以重排             |
| volatile写 | 可以重排           | 不可以重排             | 不可以重排             |

| 当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序。这个操作保证了volatile读之后的操作不会被重排到volatile读之前。 |
| ------------------------------------------------------------ |
| 当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序。这个操作保证了volatile写之前的操作不会被重排到volatile写之后。 |
| 当第一个操作为volatile写时，第二个操作为volatile读时，不能重排。 |

#### JMM 就将内存屏障插⼊策略分为 4 种

![image-20210806105559300](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105559300.png)

**写：**

1. **在每个 volatile 写操作的前⾯插⼊⼀个 StoreStore 屏障**
2.  **在每个 volatile 写操作的后⾯插⼊⼀个 StoreLoad 屏障**

![image-20210806105646798](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105646798.png)

![image-20210806105724064](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806105724064.png)

**读：**

1. **在每个 volatile 读操作的后⾯插⼊⼀个 LoadLoad 屏障**
2. **在每个 volatile 读操作的后⾯插⼊⼀个 LoadStore 屏障**

### volatile特性

#### 保证可见性

**保证不同线程对这个变量进行操作时的可见性，即变量一旦改变所有线程立即可见**

```java
import java.util.concurrent.TimeUnit;

/**
 */
public class VolatileSeeDemo{
    static          boolean flag = true;       //不加volatile，没有可见性
    //static volatile boolean flag = true;       //加了volatile，保证可见性

    public static void main(String[] args){
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t come in");
            while (flag)
            {

            }
            System.out.println(Thread.currentThread().getName()+"\t flag被修改为false,退出.....");
        },"t1").start();

        //暂停2秒钟后让main线程修改flag值
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }

        flag = false;

        System.out.println("main线程修改完成");
    }
}
```

- 不加volatile，没有可见性，程序无法停止
- 加了volatile，保证可见性，程序可以停止

| 线程t1中为何看不到被主线程main修改为false的flag的值？<br/> <br/>问题可能:<br/>1. 主线程修改了flag之后没有将其刷新到主内存，所以t1线程看不到。<br/>2. 主线程将flag刷新到了主内存，但是t1一直读取的是自己工作内存中flag的值，没有去主内存中更新获取flag最新的值。<br/> <br/>我们的诉求：<br/>1.线程中修改了工作内存中的副本之后，立即将其刷新到主内存；<br/>2.工作内存中每次读取共享变量时，都去主内存中重新读取，然后拷贝到工作内存。<br/> <br/>解决：<br/>使用volatile修饰共享变量，就可以达到上面的效果，被volatile修改的变量有以下特点：<br/>1. 线程中读取的时候，每次读取都会去主内存中读取共享变量最新的值，然后将其复制到工作内存<br/>2. 线程中修改了工作内存中变量的副本，修改之后会立即刷新到主内存 |
| ------------------------------------------------------------ |

##### **volatile变量的读写过程**

Java内存模型中定义的8种工作内存与主内存之间的原子操作
read(读取)→load(加载)→use(使用)→assign(赋值)→store(存储)→write(写入)→lock(锁定)→unlock(解锁)

![image-20210806110029085](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110029085.png)

read: 作用于主内存，将变量的值从主内存传输到工作内存，主内存到工作内存
load: 作用于工作内存，将read从主内存传输的变量值放入工作内存变量副本中，即数据加载
use: 作用于工作内存，将工作内存变量副本的值传递给执行引擎，每当JVM遇到需要该变量的字节码指令时会执行该操作
assign: 作用于工作内存，将从执行引擎接收到的值赋值给工作内存变量，每当JVM遇到一个给变量赋值字节码指令时会执行该操作
store: 作用于工作内存，将赋值完毕的工作变量的值写回给主内存
write: 作用于主内存，将store传输过来的变量值赋值给主内存中的变量
由于上述只能保证单条指令的原子性，针对多条指令的组合性原子保证，没有大面积加锁，所以，JVM提供了另外两个原子指令：
lock: 作用于主内存，将一个变量标记为一个线程独占的状态，只是写时候加锁，就只是锁了写变量的过程。
unlock: 作用于主内存，把一个处于锁定状态的变量释放，然后才能被其他线程占用

#### 没有原子性

**volatile变量的复合操作(如i++)不具有原子性**

```java
import java.util.concurrent.TimeUnit;

class MyNumber{
    volatile int number = 0;

    public void addPlusPlus(){
        number++;
    }
}

public class VolatileNoAtomicDemo{
    public static void main(String[] args) throws InterruptedException{
        MyNumber myNumber = new MyNumber();

        for (int i = 1; i <=10; i++) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; j++) {
                    myNumber.addPlusPlus();
                }
            },String.valueOf(i)).start();
        }
        
        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println(Thread.currentThread().getName() + "\t" + myNumber.number);
    }
}
```

从i++的字节码角度说明

![image-20210806110151403](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110151403.png)

原子性指的是一个操作是不可中断的，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。
public void add()
{
        i++; //不具备原子性，该操作是先读取值，然后写回一个新值，相当于原来的值加上1，分3步完成
 }
如果第二个线程在第一个线程读取旧值和写回新值期间读取i的域值，那么第二个线程就会与第一个线程一起看到同一个值，
并执行相同值的加1操作，这也就造成了线程安全失败，因此对于add方法必须使用synchronized修饰，以便保证线程安全.

![image-20210806110209393](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110209393.png)

多线程环境下，"数据计算"和"数据赋值"操作可能多次出现，即操作非原子。若数据在加载之后，若主内存count变量发生修改之后，由于线程工作内存中的值在此前已经加载，从而不会对变更操作做出相应变化，即私有内存和公共内存中变量不同步，进而导致数据不一致
对于volatile变量，JVM只是保证从主内存加载到线程工作内存的值是最新的，也就是数据加载时是最新的。
**由此可见volatile解决的是变量读时的可见性问题，但无法保证原子性，对于多线程修改共享变量的场景必须使用加锁同步**

读取赋值一个普通变量的情况:

当线程1对主内存对象发起read操作到write操作第一套流程的时间里，线程2随时都有可能对这个主内存对象发起第二套操作

![image-20210806110250220](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110250220.png)

**volatile主要是对其中部分指令做了处理**

| 要use(使用)一个变量的时候必需load(载入），要载入的时候必需从主内存read(读取）这样就解决了读的可见性。 <br/> <br/>写操作是把assign和store做了关联(在assign(赋值)后必需store(存储))。store(存储)后write(写入)。<br/>也就是做到了给一个变量赋值的时候一串关联指令直接把变量值写到主内存。<br/> <br/>就这样通过用的时候直接从主内存取，在赋值到直接写回主内存做到了内存可见性要use(使用)一个变量的时候必需load(载入），要载入的时候必需从主内存read(读取）这样就解决了读的可见性。 <br/> <br/>写操作是把assign和store做了关联(在assign(赋值)后必需store(存储))。store(存储)后write(写入)。<br/>也就是做到了给一个变量赋值的时候一串关联指令直接把变量值写到主内存。<br/> <br/>就这样通过用的时候直接从主内存取，在赋值到直接写回主内存做到了内存可见性 |
| ------------------------------------------------------------ |

![image-20210806110346327](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110346327.png)

![image-20210806110412935](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110412935.png)

| read-load-use 和 assign-store-write 成为了两个不可分割的原子操作，但是在use和assign之间依然有极小的一段真空期，有可能变量会被其他线程读取，导致写丢失一次...o(╥﹏╥)o<br/>但是无论在哪一个时间点主内存的变量和任一工作内存的变量的值都是相等的。这个特性就导致了volatile变量不适合参与到依赖当前值的运算，如i = i + 1; i++;之类的那么依靠可见性的特点volatile可以用在哪些地方呢？ 通常volatile用做保存某个状态的boolean值or int值。<br/>《深入理解Java虚拟机》提到： |
| ------------------------------------------------------------ |

![image-20210806110432030](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110432030.png)

**JVM的字节码，i++分成三步，间隙期不同步非原子操作(i++)**

![image-20210806110453546](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110453546.png)

#### 指令禁重排

重排序
重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段，有时候会改变程序语句的先后顺序
不存在数据依赖关系，可以重排序；
    存在数据依赖关系，禁止重排序
但重排后的指令绝对不能改变原有的串行语义！这点在并发设计中必须要重点考虑！

重排序的分类和执行流程

![image-20210806110531670](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110531670.png)

编译器优化的重排序： 编译器在不改变单线程串行语义的前提下，可以重新调整指令的执行顺序
指令级并行的重排序： 处理器使用指令级并行技术来讲多条指令重叠执行，若不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序
内存系统的重排序： 由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是乱序执行

数据依赖性：若两个操作访问同一变量，且这两个操作中有一个为写操作，此时两操作间就存在数据依赖性。

案例 ：
不存在数据依赖关系，可以重排序===> 重排序OK 。

| 重排前                                                     | 重排后                                                     |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| int a = 1;  //1<br/>int b = 20; //2<br/>int c = a + b; //3 | int b = 20;  //1<br/>int a = 1; //2<br/>int c = a + b; //3 |
| 结论：编译器调整了语句的顺序，但是不影响程序的最终结果。   | 重排序OK                                                   |

存在数据依赖关系，禁止重排序===> 重排序发生，会导致程序运行结果不同。
 编译器和处理器在重排序时，会遵守数据依赖性，不会改变存在依赖关系的两个操作的执行,但不同处理器和不同线程之间的数据性不会被编译器和处理器考虑，其只会作用于单处理器和单线程环境，下面三种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变。

![image-20210806110647210](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110647210.png)

##### volatile的底层实现是通过内存屏障

###### volatile有关的禁止指令重排的行为

![image-20210806110733412](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110733412.png)

当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序。这个操作保证了volatile读之后的操作不会被重排到volatile读之前。
当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序。这个操作保证了volatile写之前的操作不会被重排到volatile写之后。
当第一个操作为volatile写时，第二个操作为volatile读时，不能重排。

###### 四大屏障的插入情况

- 在每一个volatile写操作前面插入一个StoreStore屏障
  - 在每一个volatile写操作前面插入一个StoreStore屏障
- 在每一个volatile写操作后面插入一个StoreLoad屏障
  - StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序
- 在每一个volatile读操作后面插入一个LoadLoad屏障
  - LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。
- 在每一个volatile读操作后面插入一个LoadStore屏障
  - LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序。

```java
//模拟一个单线程，什么顺序读？什么顺序写？
public class VolatileTest {
    int i = 0;
    volatile boolean flag = false;
    public void write(){
        i = 2;
        flag = true;
    }
    public void read(){
        if(flag){
            System.out.println("---i = " + i);
        }
    }
}
```

![image-20210806110912360](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110912360.png)

![image-20210806110919245](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806110919245.png)

![image-20210806111001814](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111001814.png)

### 如何正确使用volatile

#### 单一赋值可以，but含复合运算赋值不可以(i++之类)

volatile int a = 10

volatile boolean flag = false

#### 状态标志，判断业务是否结束

```java
import java.util.concurrent.TimeUnit;

/**
 *
 * 使用：作为一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或任务结束
 * 理由：状态标志并不依赖于程序内任何其他状态，且通常只有一种状态转换
 * 例子：判断业务是否结束
 */
public class UseVolatileDemo{
    private volatile static boolean flag = true;

    public static void main(String[] args){
        new Thread(() -> {
            while(flag) {
                //do something......
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(2L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            flag = false;
        },"t2").start();
    }
}
```

#### 开销较低的读，写锁策略

```java
 
public class UseVolatileDemo{
    /**
     * 使用：当读远多于写，结合使用内部锁和 volatile 变量来减少同步的开销
     * 理由：利用volatile保证读取操作的可见性；利用synchronized保证复合操作的原子性
     */
    public class Counter{
        private volatile int value;

        public int getValue(){
            return value;   //利用volatile保证读取操作的可见性
              }
        public synchronized int increment(){
            return value++; //利用synchronized保证复合操作的原子性
               }
    }
}
```

#### DCL双端锁的发布

```java

public class SafeDoubleCheckSingleton{
    private static SafeDoubleCheckSingleton singleton;
    //私有化构造方法
    private SafeDoubleCheckSingleton(){
    }
    //双重锁设计
    public static SafeDoubleCheckSingleton getInstance(){
        if (singleton == null){
            //1.多线程并发创建对象时，会通过加锁保证只有一个线程能创建对象
            synchronized (SafeDoubleCheckSingleton.class){
                if (singleton == null){
                    //隐患：多线程环境下，由于重排序，该对象可能还未完成初始化就被其他线程读取
                    singleton = new SafeDoubleCheckSingleton();
                }
            }
        }
        //2.对象创建完毕，执行getInstance()将不需要获取锁，直接返回创建对象
        return singleton;
    }
}
```

单线程环境下(或者说正常情况下)，在"问题代码处"，会执行如下操作，保证能获取到已完成初始化的实例

![image-20210806111256655](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111256655.png)

隐患：多线程环境下，在"问题代码处"，会执行如下操作，由于重排序导致2,3乱序，后果就是其他线程得到的是null而不是完成初始化的对象

![image-20210806111346551](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111346551.png)

```java

public class SafeDoubleCheckSingleton{
    //通过volatile声明，实现线程安全的延迟初始化。
    private volatile static SafeDoubleCheckSingleton singleton;
    //私有化构造方法
    private SafeDoubleCheckSingleton(){
    }
    //双重锁设计
    public static SafeDoubleCheckSingleton getInstance(){
        if (singleton == null){
            //1.多线程并发创建对象时，会通过加锁保证只有一个线程能创建对象
            synchronized (SafeDoubleCheckSingleton.class){
                if (singleton == null){
                    //隐患：多线程环境下，由于重排序，该对象可能还未完成初始化就被其他线程读取
                                      //原理:利用volatile，禁止 "初始化对象"(2) 和 "设置singleton指向内存空间"(3) 的重排序
                    singleton = new SafeDoubleCheckSingleton();
                }
            }
        }
        //2.对象创建完毕，执行getInstance()将不需要获取锁，直接返回创建对象
        return singleton;
    }
}
```

```java
 
 
//现在比较好的做法就是采用静态内部内的方式实现
 
public class SingletonDemo{
    private SingletonDemo() { }

    private static class SingletonDemoHandler{
        private static SingletonDemo instance = new SingletonDemo();
    }

    public static SingletonDemo getInstance(){
        return SingletonDemoHandler.instance;
    }
}
```

### 总结

![image-20210806111527160](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111527160.png)

#### 内存屏障能干嘛

1. 阻止屏障两边的指令重排序
2. 写数据时加入屏障，强制将线程私有工作内存的数据刷回主物理内存
3. 读数据时加入屏障，线程私有工作内存的数据失效，重新到主物理内存中获取最新数据

#### 内存屏障四大指令

在每一个volatile写操作前面插入一个StoreStore屏障

![image-20210806111618948](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111618948.png)

在每一个volatile写操作后面插入一个StoreLoad屏障

![image-20210806111633147](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111633147.png)

在每一个volatile读操作后面插入一个LoadLoad屏障

![image-20210806111644134](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111644134.png)

在每一个volatile读操作后面插入一个LoadStore屏障

![image-20210806111654090](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111654090.png)

![image-20210806111715872](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111715872.png)

![image-20210806111724080](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111724080.png)

#### volatile可见性

![image-20210806111738880](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111738880.png)

#### volatile禁重排

![image-20210806111753477](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111753477.png)

![image-20210806111801813](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111801813.png)

#### 对比java.util.concurrent.locks.Lock来理解

![image-20210806111817945](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111817945.png)

![image-20210806111828735](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210806111828735.png)

## CAS

多线程环境不使用原子类保证线程安全（基本数据类型）

```java
import java.util.concurrent.atomic.AtomicInteger;


public class T3{
    volatile int number = 0;
    //读取
    public int getNumber(){
        return number;
    }
    //写入加锁保证原子性
    public synchronized void setNumber(){
        number++;
    }
}
```

多线程环境    使用原子类保证线程安全（基本数据类型）

```java
import java.util.concurrent.atomic.AtomicInteger;


public class T3{
    volatile int number = 0;
    //读取
    public int getNumber(){
        return number;
    }
    //写入加锁保证原子性
    public synchronized void setNumber(){
        number++;
    }
    //=================================
    AtomicInteger atomicInteger = new AtomicInteger();

    public int getAtomicInteger(){
        return atomicInteger.get();
    }

    public void setAtomicInteger(){
        atomicInteger.getAndIncrement();
    }


}
```

****CAS**
**compare and swap的缩写，中文翻译成比较并交换,实现并发算法时常用到的一种技术。它包含三个操作数——内存位置、预期原值及更新值。**
**执行CAS操作的时候，将内存位置的值与预期原值比较：**
**如果相匹配，那么处理器会自动将该位置值更新为新值，**
如果不匹配，处理器不做任何操作，多个线程同时执行CAS操作只有一个会成功。** 

 CAS （CompareAndSwap） 
CAS有3个操作数，位置内存值V，旧的预期值A，要修改的更新值B。
当且仅当旧的预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做或重来

  ![image-20210807133222761](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133222761.png)


CAS是JDK提供的非阻塞原子性操作，它通过硬件保证了比较-更新的原子性。
它是非阻塞的且自身原子性，也就是说这玩意效率更高且通过硬件保证，说明这玩意更可靠。

CAS是一条CPU的原子指令（cmpxchg指令），不会造成所谓的数据不一致问题，Unsafe提供的CAS方法（如compareAndSwapXXX）底层实现即为CPU指令cmpxchg。
执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说CAS的原子性实际上是CPU实现的， 其实在这一点上还是有排他锁的，只是比起用synchronized， 这里的排他时间要短的多， 所以在多线程情况下性能会比较好

```java
import java.util.concurrent.atomic.AtomicInteger;


public class CASDemo{
    public static void main(String[] args) throws InterruptedException
    {
        AtomicInteger atomicInteger = new AtomicInteger(5);

        System.out.println(atomicInteger.compareAndSet(5, 2020)+"\t"+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 1024)+"\t"+atomicInteger.get());
    }
}
```

**源码分析compareAndSet(int expect,int update)**

compareAndSet()方法的源代码：

![image-20210807133409768](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133409768.png)

上面三个方法都是类似的，主要对4个参数做一下说明。
var1：表示要操作的对象
var2：表示要操作对象中属性地址的偏移量
var4：表示需要修改数据的期望的值
var5/var6：表示需要修改为的新值

![image-20210807133422184](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133422184.png)

### UnSafe

![image-20210807133510827](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133510827.png)

1 Unsafe
   是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针一样直接操作内存，因为Java中CAS操作的执行依赖于Unsafe类的方法。
注意Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务 

 2 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。

![image-20210807133541948](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133541948.png)

3 变量value用volatile修饰，保证了多线程之间的内存可见性。

### atomicInteger.getAndIncrement()

CAS的全称为Compare-And-Swap，它是一条CPU并发原语。
它的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值，这个过程是原子的。
AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

![image-20210807133637695](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133637695.png)

CAS并发原语体现在JAVA语言中就是sun.misc.Unsafe类中的各个方法。调用UnSafe类中的CAS方法，JVM会帮我们实现出CAS汇编指令。这是一种完全依赖于硬件的功能，通过它实现了原子操作。再次强调，由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成所谓的数据不一致问题。

### new AtomicInteger().getAndIncrement();

![image-20210807133722072](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133722072.png)![image-20210807133738008](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133738008.png)

![image-20210807133744827](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807133744827.png)

| 假设线程A和线程B两个线程同时执行getAndAddInt操作（分别跑在不同CPU上）：<br/> <br/>1  AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据JMM模型，线程A和线程B各自持有一份值为3的value的副本分别到各自的工作内存。<br/> <br/>2  线程A通过getIntVolatile(var1, var2)拿到value值3，这时线程A被挂起。<br/> <br/>3  线程B也通过getIntVolatile(var1, var2)方法获取到value值3，此时刚好线程B没有被挂起并执行compareAndSwapInt方法比较内存值也为3，成功修改内存值为4，线程B打完收工，一切OK。<br/> <br/>4  这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值数字3和主内存的值数字4不一致，说明该值已经被其它线程抢先一步修改过了，那A线程本次修改失败，只能重新读取重新来一遍了。<br/> <br/>5  线程A重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。 |
| ------------------------------------------------------------ |

### 底层汇编

#### native修饰的方法代表是底层方法

（非计算机专业的，不要求懂，可以不听，需要汇编知识）
Unsafe类中的compareAndSwapInt，是一个本地方法，该方法的实现位于unsafe.cpp中

```c++
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
// 先想办法拿到变量value在内存中的地址，根据偏移量valueOffset，计算 value 的地址
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
// 调用 Atomic 中的函数 cmpxchg来进行比较交换，其中参数x是即将更新的值，参数e是原内存的值
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

**(Atomic::cmpxchg(x, addr, e)) == e;**

#### cmpxchg

// 调用 Atomic 中的函数 cmpxchg来进行比较交换，其中参数x是即将更新的值，参数e是原内存的值
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;

| unsigned Atomic::cmpxchg(unsigned int exchange_value,volatile unsigned int* dest, unsigned int compare_value) {<br/>    assert(sizeof(unsigned int) == sizeof(jint), "more work to do");<br/>  /*<br/>   * 根据操作系统类型调用不同平台下的重载函数，这个在预编译期间编译器会决定调用哪个平台下的重载函数*/<br/>    return (unsigned int)Atomic::cmpxchg((jint)exchange_value, (volatile jint*)dest, (jint)compare_value);<br/>} |
| ------------------------------------------------------------ |

#### 在不同的操作系统下会调用不同的cmpxchg重载函数

```c++
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
  //判断是否是多核CPU
  int mp = os::is_MP();
  __asm {
    //三个move指令表示的是将后面的值移动到前面的寄存器上
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    //CPU原语级别，CPU触发
    LOCK_IF_MP(mp)
    //比较并交换指令
    //cmpxchg: 即“比较并交换”指令
    //dword: 全称是 double word 表示两个字，一共四个字节
    //ptr: 全称是 pointer，与前面的 dword 连起来使用，表明访问的内存单元是一个双字单元 
    //将 eax 寄存器中的值（compare_value）与 [edx] 双字内存单元中的值进行对比，
    //如果相同，则将 ecx 寄存器中的值（exchange_value）存入 [edx] 内存单元中
    cmpxchg dword ptr [edx], ecx
  }
}
```

最终是由操作系统的汇编指令完成的。

你只需要记住：CAS是靠硬件实现的从而在硬件层面提升效率，最底层还是交给硬件来保证原子性和可见性
实现方式是基于硬件平台的汇编指令，在intel的CPU中(X86机器上)，使用的是汇编指令cmpxchg指令。 

核心思想就是：比较要更新变量的值V和预期值E（compare），相等才会将V的值设为新值N（swap）如果不相等自旋再来。

### 原子引用

AtomicReferenceDemo

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

import java.util.concurrent.atomic.AtomicReference;

@Getter
@ToString
@AllArgsConstructor
class User{
    String userName;
    int    age;
}


public class AtomicReferenceDemo{
    public static void main(String[] args){
        User z3 = new User("z3",24);
        User li4 = new User("li4",26);

        AtomicReference<User> atomicReferenceUser = new AtomicReference<>();

        atomicReferenceUser.set(z3);
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
    }
}
```

### 自旋锁，借鉴CAS思想

自旋锁（spinlock）

是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，
当线程发现锁被占用时，会不断循环判断锁的状态，直到获取。这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU

![image-20210807134158613](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807134158613.png)

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

/**
 * 题目：实现一个自旋锁
 * 自旋锁好处：循环比较获取没有类似wait的阻塞。
 *
 * 通过CAS操作完成自旋锁，A线程先进来调用myLock方法自己持有锁5秒钟，B随后进来后发现
 * 当前有线程持有锁，不是null，所以只能通过自旋等待，直到A释放锁后B随后抢到。
 */
public class SpinLockDemo{
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"\t come in");
        while(!atomicReference.compareAndSet(null,thread))
        {

        }
    }

    public void myUnLock(){
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null);
        System.out.println(Thread.currentThread().getName()+"\t myUnLock over");
    }

    public static void main(String[] args){
        SpinLockDemo spinLockDemo = new SpinLockDemo();

        new Thread(() -> {
            spinLockDemo.myLock();
            try { TimeUnit.SECONDS.sleep( 5 ); } catch (InterruptedException e) { e.printStackTrace(); }
            spinLockDemo.myUnLock();
        },"A").start();

        //暂停一会儿线程，保证A线程先于B线程启动并完成
        try { TimeUnit.SECONDS.sleep( 1 ); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            spinLockDemo.myLock();
            spinLockDemo.myUnLock();
        },"B").start();

    }
}
```

### CAS缺点

#### 循环时间长开销很大。

我们可以看到getAndAddInt方法执行时，有个do while

![image-20210807134309220](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807134309220.png)

如果CAS失败，会一直进行尝试。如果CAS长时间一直不成功，可能会给CPU带来很大的开销

#### 引出来ABA问题


 CAS会导致“ABA问题”。

CAS算法实现一个重要前提需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且线程two进行了一些操作将值变成了B，
然后线程two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后线程one操作成功。

尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。

**版本号时间戳原子引用**

AtomicStampedReference

```java
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;


public class ABADemo{
    static AtomicInteger atomicInteger = new AtomicInteger(100);
    static AtomicStampedReference atomicStampedReference = new AtomicStampedReference(100,1);

    public static void main(String[] args)
    {
        new Thread(() -> {
            atomicInteger.compareAndSet(100,101);
            atomicInteger.compareAndSet(101,100);
        },"t1").start();

        new Thread(() -> {
            //暂停一会儿线程
            try { Thread.sleep( 500 ); } catch (InterruptedException e) { e.printStackTrace(); };            System.out.println(atomicInteger.compareAndSet(100, 2019)+"\t"+atomicInteger.get());
        },"t2").start();

        //暂停一会儿线程,main彻底等待上面的ABA出现演示完成。
        try { Thread.sleep( 2000 ); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("============以下是ABA问题的解决=============================");

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 首次版本号:"+stamp);//1
            //暂停一会儿线程,
            try { Thread.sleep( 1000 ); } catch (InterruptedException e) { e.printStackTrace(); }
            atomicStampedReference.compareAndSet(100,101,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 2次版本号:"+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 3次版本号:"+atomicStampedReference.getStamp());
        },"t3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 首次版本号:"+stamp);//1
            //暂停一会儿线程，获得初始值100和初始版本号1，故意暂停3秒钟让t3线程完成一次ABA操作产生问题
            try { Thread.sleep( 3000 ); } catch (InterruptedException e) { e.printStackTrace(); }
            boolean result = atomicStampedReference.compareAndSet(100,2019,stamp,stamp+1);
            System.out.println(Thread.currentThread().getName()+"\t"+result+"\t"+atomicStampedReference.getReference());
        },"t4").start();
    }
}
```

## 原子操作类之18罗汉增强

![image-20210807134528160](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807134528160.png)

### atomic

1. AtomicBoolean
2. AtomicInteger
3. AtomicIntegerArray
4. AtomicIntegerFieldUpdater
5. AtomicLong
6. AtomicLongArray
7. AtomicLongFieldUpdater
8. AtomicMarkableReference
9. AtomicReference
10. AtomicReferenceArray
11. AtomicReferenceFieldUpdater
12. AtomicStampedReference
13. **DoubleAccumulator**
14. **DoubleAdder**
15. **LongAccumulator**
16. **LongAdder**

### 基本类型原子类

- AtomicInteger
- AtomicBoolean
- AtomicLong

#### 常用API简介

1. public final int get() //获取当前的值
2. public final int getAndSet(int newValue)//获取当前的值，并设置新的值
3. public final int getAndIncrement()//获取当前的值，并自增
4. public final int getAndDecrement() //获取当前的值，并自减
5. public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
6. boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）

```java
import lombok.Getter;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

class MyNumber{
    @Getter
    private AtomicInteger atomicInteger = new AtomicInteger();
    public void addPlusPlus(){
        atomicInteger.incrementAndGet();
    }
}


public class AtomicIntegerDemo{
    public static void main(String[] args) throws InterruptedException{
        MyNumber myNumber = new MyNumber();
        CountDownLatch countDownLatch = new CountDownLatch(100);

        for (int i = 1; i <=100; i++) {
            new Thread(() -> {
                try {
                    for (int j = 1; j <=5000; j++)
                    {
                        myNumber.addPlusPlus();
                    }
                }finally {
                    countDownLatch.countDown();
                }
            },String.valueOf(i)).start();
        }

        countDownLatch.await();

        System.out.println(myNumber.getAtomicInteger().get());
    }
}
```

### 数组类型原子类

1. AtomicIntegerArray
2. AtomicLongArray
3. AtomicReferenceArray

```java
import java.util.concurrent.atomic.AtomicIntegerArray;


public class AtomicIntegerArrayDemo{
    public static void main(String[] args){
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[5]);
        //AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(5);
        //AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[]{1,2,3,4,5});

        for (int i = 0; i <atomicIntegerArray.length(); i++) {
            System.out.println(atomicIntegerArray.get(i));
        }
        System.out.println();
        System.out.println();
        System.out.println();
        int tmpInt = 0;

        tmpInt = atomicIntegerArray.getAndSet(0,1122);
        System.out.println(tmpInt+"\t"+atomicIntegerArray.get(0));
        atomicIntegerArray.getAndIncrement(1);
        atomicIntegerArray.getAndIncrement(1);
        tmpInt = atomicIntegerArray.getAndIncrement(1);
        System.out.println(tmpInt+"\t"+atomicIntegerArray.get(1));
    }
}
```

### 引用类型原子类

- AtomicReference

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

import java.util.concurrent.atomic.AtomicReference;

@Getter
@ToString
@AllArgsConstructor
class User{
    String userName;
    int    age;
}

/**
 * @auther zzyy
 * @create 2018-12-31 17:22
 */
public class AtomicReferenceDemo{
    public static void main(String[] args){
        User z3 = new User("z3",24);
        User li4 = new User("li4",26);

        AtomicReference<User> atomicReferenceUser = new AtomicReference<>();

        atomicReferenceUser.set(z3);
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
    }
}
```

- AtomicStampedReference

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 *解决ABA问题
 */
public class ABADemo{
    static AtomicInteger atomicInteger = new AtomicInteger(100);
    static AtomicStampedReference atomicStampedReference = new AtomicStampedReference(100,1);

    public static void main(String[] args){
        abaProblem();
        abaResolve();
    }

    public static void abaResolve(){
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println("t3 ----第1次stamp  "+stamp);
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            atomicStampedReference.compareAndSet(100,101,stamp,stamp+1);
            System.out.println("t3 ----第2次stamp  "+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println("t3 ----第3次stamp  "+atomicStampedReference.getStamp());
        },"t3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println("t4 ----第1次stamp  "+stamp);
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            boolean result = atomicStampedReference.compareAndSet(100, 20210308, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName()+"\t"+result+"\t"+atomicStampedReference.getReference());
        },"t4").start();
    }

    public static void abaProblem(){
        new Thread(() -> {
            atomicInteger.compareAndSet(100,101);
            atomicInteger.compareAndSet(101,100);
        },"t1").start();

        try { TimeUnit.MILLISECONDS.sleep(200); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            atomicInteger.compareAndSet(100,20210308);
            System.out.println(atomicInteger.get());
        },"t2").start();
    }
}
```

携带版本号的引用类型原子类，可以解决ABA问题

解决修改过几次

状态戳原子引用

- AtomicMarkableReference

原子更新带有标记位的引用类型对象

解决是否修改过(它的定义就是将状态戳简化为true|false)

状态戳(true/false)原子引用

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicMarkableReference;
import java.util.concurrent.atomic.AtomicStampedReference;


public class ABADemo{
    static AtomicInteger atomicInteger = new AtomicInteger(100);
    static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(100,1);
    static AtomicMarkableReference<Integer> markableReference = new AtomicMarkableReference<>(100,false);

    public static void main(String[] args){
        new Thread(() -> {
            atomicInteger.compareAndSet(100,101);
            atomicInteger.compareAndSet(101,100);
            System.out.println(Thread.currentThread().getName()+"\t"+"update ok");
        },"t1").start();

        new Thread(() -> {
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            atomicInteger.compareAndSet(100,2020);
        },"t2").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println(atomicInteger.get());

        System.out.println();
        System.out.println();
        System.out.println();

        System.out.println("============以下是ABA问题的解决,让我们知道引用变量中途被更改了几次=========================");
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t 1次版本号"+stampedReference.getStamp());
            //故意暂停200毫秒，让后面的t4线程拿到和t3一样的版本号
            try { TimeUnit.MILLISECONDS.sleep(200); } catch (InterruptedException e) { e.printStackTrace(); }

            stampedReference.compareAndSet(100,101,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 2次版本号"+stampedReference.getStamp());
            stampedReference.compareAndSet(101,100,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 3次版本号"+stampedReference.getStamp());
        },"t3").start();

        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t =======1次版本号"+stamp);
            //暂停2秒钟,让t3先完成ABA操作了，看看自己还能否修改
            try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
            boolean b = stampedReference.compareAndSet(100, 2020, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName()+"\t=======2次版本号"+stampedReference.getStamp()+"\t"+stampedReference.getReference());
        },"t4").start();

        System.out.println();
        System.out.println();
        System.out.println();

        System.out.println("============AtomicMarkableReference不关心引用变量更改过几次，只关心是否更改过======================");

        new Thread(() -> {
            boolean marked = markableReference.isMarked();
            System.out.println(Thread.currentThread().getName()+"\t 1次版本号"+marked);
            try { TimeUnit.MILLISECONDS.sleep(100); } catch (InterruptedException e) { e.printStackTrace(); }
            markableReference.compareAndSet(100,101,marked,!marked);
            System.out.println(Thread.currentThread().getName()+"\t 2次版本号"+markableReference.isMarked());
            markableReference.compareAndSet(101,100,markableReference.isMarked(),!markableReference.isMarked());
            System.out.println(Thread.currentThread().getName()+"\t 3次版本号"+markableReference.isMarked());
        },"t5").start();

        new Thread(() -> {
            boolean marked = markableReference.isMarked();
            System.out.println(Thread.currentThread().getName()+"\t 1次版本号"+marked);
            //暂停几秒钟线程
            try { TimeUnit.MILLISECONDS.sleep(100); } catch (InterruptedException e) { e.printStackTrace(); }
            markableReference.compareAndSet(100,2020,marked,!marked);
            System.out.println(Thread.currentThread().getName()+"\t"+markableReference.getReference()+"\t"+markableReference.isMarked());
        },"t6").start();
    }
}
```

### 对象的属性修改原子类

- AtomicIntegerFieldUpdater --- 原子更新对象中int类型字段的值
- AtomicLongFieldUpdater --- 原子更新对象中Long类型字段的值
- AtomicReferenceFieldUpdater --- 原子更新引用类型字段的值

以一种线程安全的方式操作非线程安全对象内的某些字段

更新的对象属性必须使用 public volatile 修饰符。

因为对象的属性修改类型原子类都是抽象类，所以每次使用都必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;


class BankAccount{
    private String bankName = "CCB";//银行
    public volatile int money = 0;//钱数
    AtomicIntegerFieldUpdater<BankAccount> accountAtomicIntegerFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(BankAccount.class,"money");

    //不加锁+性能高，局部微创
    public void transferMoney(BankAccount bankAccount){
        accountAtomicIntegerFieldUpdater.incrementAndGet(bankAccount);
    }
}

/**
 * 以一种线程安全的方式操作非线程安全对象的某些字段。
 * 需求：
 * 1000个人同时向一个账号转账一元钱，那么累计应该增加1000元，
 * 除了synchronized和CAS,还可以使用AtomicIntegerFieldUpdater来实现。
 */
public class AtomicIntegerFieldUpdaterDemo{

    public static void main(String[] args){
        BankAccount bankAccount = new BankAccount();

        for (int i = 1; i <=1000; i++) {
            int finalI = i;
            new Thread(() -> {
                bankAccount.transferMoney(bankAccount);
            },String.valueOf(i)).start();
        }

        //暂停毫秒
        try { TimeUnit.MILLISECONDS.sleep(500); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println(bankAccount.money);

    }
}
```

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

class MyVar{
    public volatile Boolean isInit = Boolean.FALSE;
    AtomicReferenceFieldUpdater<MyVar,Boolean> atomicReferenceFieldUpdater = AtomicReferenceFieldUpdater.newUpdater(MyVar.class,Boolean.class,"isInit");


    public void init(MyVar myVar){
        if(atomicReferenceFieldUpdater.compareAndSet(myVar,Boolean.FALSE,Boolean.TRUE)){
            System.out.println(Thread.currentThread().getName()+"\t"+"---init.....");
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t"+"---init.....over");
        }else{
            System.out.println(Thread.currentThread().getName()+"\t"+"------其它线程正在初始化");
        }
    }


}


/**
 * 多线程并发调用一个类的初始化方法，如果未被初始化过，将执行初始化工作，要求只能初始化一次
 */
public class AtomicIntegerFieldUpdaterDemo{
    public static void main(String[] args) throws InterruptedException{
        MyVar myVar = new MyVar();

        for (int i = 1; i <=5; i++) {
            new Thread(() -> {
                myVar.init(myVar);
            },String.valueOf(i)).start();
        }
    }
}
```

### 原子操作增强类原理深度解析

1. DoubleAccumulator
2. DoubleAdder
3. LongAccumulator
4. LongAdder

![image-20210807135813648](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807135813648.png)

* 1    热点商品点赞计算器，点赞数加加统计，不要求实时精确
* 2    一个很大的List，里面都是int类型，如何实现加加，说说思路

![image-20210807135839294](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807135839294.png)

**LongAdder只能用来计算加法，且从零开始计算**

**LongAccumulator提供了自定义的函数操作**

```java
 
long类型的聚合器，需要传入一个long类型的二元操作，可以用来计算各种聚合操作，包括加乘等
 
import java.util.concurrent.atomic.LongAccumulator;
import java.util.concurrent.atomic.LongAdder;
import java.util.function.LongBinaryOperator;


public class LongAccumulatorDemo{

    LongAdder longAdder = new LongAdder();
    public void add_LongAdder(){
        longAdder.increment();
    }

    //LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x + y,0);
    LongAccumulator longAccumulator = new LongAccumulator(new LongBinaryOperator(){
        @Override
        public long applyAsLong(long left, long right)
        {
            return left - right;
        }
    },777);

    public void add_LongAccumulator(){
        longAccumulator.accumulate(1);
    }

    public static void main(String[] args){
        LongAccumulatorDemo demo = new LongAccumulatorDemo();

        demo.add_LongAccumulator();
        demo.add_LongAccumulator();
        System.out.println(demo.longAccumulator.longValue());
    }
}
```

```java
import java.util.concurrent.atomic.LongAccumulator;
import java.util.concurrent.atomic.LongAdder;


public class LongAdderAPIDemo{
    public static void main(String[] args) {
        LongAdder longAdder = new LongAdder();

        longAdder.increment();
        longAdder.increment();
        longAdder.increment();

        System.out.println(longAdder.longValue());

        LongAccumulator longAccumulator = new LongAccumulator((x,y) -> x * y,2);

        longAccumulator.accumulate(1);
        longAccumulator.accumulate(2);
        longAccumulator.accumulate(3);

        System.out.println(longAccumulator.longValue());

    }
}
```

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAccumulator;
import java.util.concurrent.atomic.LongAdder;

class ClickNumberNet{
    int number = 0;
    public synchronized void clickBySync()
    {
        number++;
    }

    AtomicLong atomicLong = new AtomicLong(0);
    public void clickByAtomicLong()
    {
        atomicLong.incrementAndGet();
    }

    LongAdder longAdder = new LongAdder();
    public void clickByLongAdder()
    {
        longAdder.increment();
    }

    LongAccumulator longAccumulator = new LongAccumulator((x,y) -> x + y,0);
    public void clickByLongAccumulator()
    {
        longAccumulator.accumulate(1);
    }
}

/**
 * 50个线程，每个线程100W次，总点赞数出来
 */
public class LongAdderDemo2{
    public static void main(String[] args) throws InterruptedException
    {
        ClickNumberNet clickNumberNet = new ClickNumberNet();

        long startTime;
        long endTime;
        CountDownLatch countDownLatch = new CountDownLatch(50);
        CountDownLatch countDownLatch2 = new CountDownLatch(50);
        CountDownLatch countDownLatch3 = new CountDownLatch(50);
        CountDownLatch countDownLatch4 = new CountDownLatch(50);


        startTime = System.currentTimeMillis();
        for (int i = 1; i <=50; i++) {
            new Thread(() -> {
                try
                {
                    for (int j = 1; j <=100 * 10000; j++) {
                        clickNumberNet.clickBySync();
                    }
                }finally {
                    countDownLatch.countDown();
                }
            },String.valueOf(i)).start();
        }
        countDownLatch.await();
        endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒"+"\t clickBySync result: "+clickNumberNet.number);

        startTime = System.currentTimeMillis();
        for (int i = 1; i <=50; i++) {
            new Thread(() -> {
                try
                {
                    for (int j = 1; j <=100 * 10000; j++) {
                        clickNumberNet.clickByAtomicLong();
                    }
                }finally {
                    countDownLatch2.countDown();
                }
            },String.valueOf(i)).start();
        }
        countDownLatch2.await();
        endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒"+"\t clickByAtomicLong result: "+clickNumberNet.atomicLong);

        startTime = System.currentTimeMillis();
        for (int i = 1; i <=50; i++) {
            new Thread(() -> {
                try
                {
                    for (int j = 1; j <=100 * 10000; j++) {
                        clickNumberNet.clickByLongAdder();
                    }
                }finally {
                    countDownLatch3.countDown();
                }
            },String.valueOf(i)).start();
        }
        countDownLatch3.await();
        endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒"+"\t clickByLongAdder result: "+clickNumberNet.longAdder.sum());

        startTime = System.currentTimeMillis();
        for (int i = 1; i <=50; i++) {
            new Thread(() -> {
                try
                {
                    for (int j = 1; j <=100 * 10000; j++) {
                        clickNumberNet.clickByLongAccumulator();
                    }
                }finally {
                    countDownLatch4.countDown();
                }
            },String.valueOf(i)).start();
        }
        countDownLatch4.await();
        endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒"+"\t clickByLongAccumulator result: "+clickNumberNet.longAccumulator.longValue());


    }
}
```

#### 架构

![image-20210807140056257](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140056257.png)

![image-20210807140105399](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140105399.png)

![image-20210807140111911](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140111911.png)

LongAdder是Striped64的子类

![image-20210807140132578](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140132578.png)

![image-20210807140153674](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140153674.png)

#### Striped64

Striped64有几个比较重要的成员函数

```java
/** Number of CPUS, to place bound on table size        CPU数量，即cells数组的最大长度 */
static final int NCPU = Runtime.getRuntime().availableProcessors();

/**
 * Table of cells. When non-null, size is a power of 2.
cells数组，为2的幂，2,4,8,16.....，方便以后位运算
 */
transient volatile Cell[] cells;

/**基础value值，当并发较低时，只累加该值主要用于没有竞争的情况，通过CAS更新。
 * Base value, used mainly when there is no contention, but also as
 * a fallback during table initialization races. Updated via CAS.
 */
transient volatile long base;

/**创建或者扩容Cells数组时使用的自旋锁变量调整单元格大小（扩容），创建单元格时使用的锁。
 * Spinlock (locked via CAS) used when resizing and/or creating Cells. 
 */
transient volatile int cellsBusy;
```

![image-20210807140239247](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140239247.png)

Striped64中一些变量或者方法的定义

![image-20210807140304651](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140304651.png)

#### Cell

是 java.util.concurrent.atomic 下 Striped64 的一个内部类

![image-20210807140338913](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140338913.png)

LongAdder的基本思路就是分散热点，将value值分散到一个Cell数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回。

sum()会将所有Cell数组中的value和base累加作为返回值，核心的思想就是将之前AtomicLong一个value的更新压力分散到多个value中去，
从而降级更新热点。

![image-20210807140358358](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140358358.png)

Value = Base + ![image-20210807140413285](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140413285.png)

**内部有一个base变量，一个Cell[]数组。**

base变量：非竞态条件下，直接累加到该变量上

Cell[]数组：竞态条件下，累加个各个线程自己的槽Cell[i]中

#### 源码解读深度分析

LongAdder在无竞争的情况，跟AtomicLong一样，对同一个base进行操作，当出现竞争关系时则是采用化整为零的做法，从空间换时间，用一个数组cells，将一个value拆分进这个数组cells。多个线程需要同时对value进行操作时候，可以对线程id进行hash得到hash值，再根据hash值映射到这个数组cells的某个下标，再对该下标所对应的值进行自增操作。当所有线程操作完毕，将数组cells的所有值和无竞争值base都加起来作为最终结果。

![image-20210807140548508](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140548508.png)

![image-20210807140555667](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140555667.png)

**longAdder.increment()**

##### add(1L)

![image-20210807140645244](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140645244.png)

条件递增，逐步解析

![image-20210807140701609](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140701609.png)

![image-20210807140707395](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140707395.png)

**1.最初无竞争时只更新base；**

**2.如果更新base失败后，首次新建一个Cell[]数组**

**3.当多个线程竞争同一个Cell比较激烈时，可能就要对Cell[]扩容**

##### longAccumulate

![image-20210807140757877](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140757877.png)

**Striped64中一些变量或者方法的定义**

![image-20210807140813397](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140813397.png)

![image-20210807140834777](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140834777.png)

###### 线程hash值：probe

![image-20210807140857018](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140857018.png)

![image-20210807140903342](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140903342.png)

![image-20210807140910343](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140910343.png)

![image-20210807140916819](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140916819.png)

###### 总纲

![image-20210807140937890](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807140937890.png)

| 上述代码首先给当前线程分配一个hash值，然后进入一个for(;;)自旋，这个自旋分为三个分支：<br/>CASE1：Cell[]数组已经初始化<br/>CASE2：Cell[]数组未初始化(首次新建)<br/>CASE3：Cell[]数组正在初始化中 |
| ------------------------------------------------------------ |

###### 计算

刚刚要初始化Cell[]数组(首次新建)  --- 未初始化过Cell[]数组，尝试占有锁并首次初始化cells数组

![image-20210807141042711](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141042711.png)

| 如果上面条件都执行成功就会执行数组的初始化及赋值操作， Cell[] rs = new Cell[2]表示数组的长度为2，<br/>rs[h & 1] = new Cell(x) 表示创建一个新的Cell元素，value是x值，默认为1。<br/>h & 1类似于我们之前HashMap常用到的计算散列桶index的算法，通常都是hash & (table.len - 1)。同hashmap一个意思。 |
| ------------------------------------------------------------ |

多个线程尝试CAS修改失败的线程会走到这个分支

![image-20210807141123336](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141123336.png)

该分支实现直接操作base基数，将值累加到base上，也即其它线程正在初始化，多个线程正在更新base的值。

Cell数组不再为空且可能存在Cell数组扩容

1.多个线程同时命中一个cell的竞争

![image-20210807141206816](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141206816.png)

![image-20210807141215681](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141215681.png)

| 上面代码判断当前线程hash后指向的数据位置元素是否为空，<br/>如果为空则将Cell数据放入数组中，跳出循环。<br/>如果不空则继续循环。 |
| ------------------------------------------------------------ |

![image-20210807141233459](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141233459.png)

![image-20210807141242678](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141242678.png)

| 说明当前线程对应的数组中有了数据，也重置过hash值，<br/>这时通过CAS操作尝试对当前数中的value值进行累加x操作，x默认为1，如果CAS成功则直接跳出循环。 |
| ------------------------------------------------------------ |

![image-20210807141301113](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141301113.png)

![image-20210807141308881](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141308881.png)

![image-20210807141315276](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141315276.png)

**总结**

![image-20210807141330929](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141330929.png)

##### sum

sum()会将所有Cell数组中的value和base累加作为返回值。核心的思想就是将之前AtomicLong一个value的更新压力分散到多个value中去，从而降级更新热点。

sum执行时，并没有限制对base和cells的更新(一句要命的话)。所以LongAdder不是强一致性的，它是最终一致性的。

首先，最终返回的sum局部变量，初始被复制为base，而最终返回时，很可能base已经被更新了，而此时局部变量sum不会更新，造成不一致。
其次，这里对cell的读取也无法保证是最后一次写入的值。所以，sum方法在没有并发的情况下，可以获得正确的结果。

![image-20210807141420550](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807141420550.png)

#### 使用总结

- AtomicLong
  - 线程安全，可允许一些性能损耗，要求高精度时可使用
  - 保证精度，性能代价
  - AtomicLong是多个线程针对单个热点值value进行原子操作
- LongAdder
  - 当需要在高并发下有较好的性能表现，且对值的精确度要求不高时，可以使用
  - 保证性能，精度代价
  - LongAdder是每个线程拥有自己的槽，各个线程一般只对自己槽中的那个值进行CAS操作

#### 总结

##### AtomicLong

- 原理
  - CAS+自旋
  - incrementAndGet
- 场景
  - 低并发下的全局计算
  - AtomicLong能保证并发情况下计数的准确性，其内部通过CAS来解决并发安全性的问题。
- 缺陷
  - 高并发后性能急剧下降
  - AtomicLong的自旋会成为瓶颈

N个线程CAS操作修改线程的值，每次只有一个成功过，其它N - 1失败，失败的不停的自旋直到成功，这样大量失败自旋的情况，一下子cpu就打高了。

##### LongAdder vs AtomicLong Performance

http://blog.palominolabs.com/2014/02/10/java-8-performance-improvements-longadder-vs-atomiclong/

##### LongAdder

- 原理
  - CAS+Base+Cell数组分散
  - 空间换时间并分散了热点数据
- 场景
  - 高并发下的全局计算
- 缺陷
  - sum求和后还有计算线程修改结果的话，最后结果不够准确

## 聊聊ThreadLocal

### ThreadLocal简介

#### 是什么

![image-20210807142141911](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142141911.png)

稍微翻译一下：
ThreadLocal提供线程局部变量。这些变量与正常的变量不同，因为每一个线程在访问ThreadLocal实例的时候（通过其get或set方法）都有自己的、独立初始化的变量副本。ThreadLocal实例通常是类中的私有静态字段，使用它的目的是希望将状态（例如，用户ID或事务ID）与线程关联起来。

#### 能干嘛

 实现每一个线程都有自己专属的本地变量副本(自己用自己的变量不麻烦别人，不和其他人共享，人人有份，人各一份)，
主要解决了让每个线程绑定自己的值，通过使用get()和set()方法，获取默认值或将其值更改为当前线程所存的副本的值从而避免了线程安全问题。

![image-20210807142216051](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142216051.png)

#### api介绍

![image-20210807142227940](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142227940.png)

```java
import java.util.concurrent.TimeUnit;

class MovieTicket{
    int number = 50;

    public synchronized void saleTicket()
    {
        if(number > 0)
        {
            System.out.println(Thread.currentThread().getName()+"\t"+"号售票员卖出第： "+(number--));
        }else{
            System.out.println("--------卖完了");
        }
    }
}

/**
 * 三个售票员卖完50张票务，总量完成即可，吃大锅饭，售票员每个月固定月薪
 */
public class ThreadLocalDemo
{
    public static void main(String[] args)
    {
        MovieTicket movieTicket = new MovieTicket();

        for (int i = 1; i <=3; i++) {
            new Thread(() -> {
                for (int j = 0; j <20; j++) {
                    movieTicket.saleTicket();
                    try { TimeUnit.MILLISECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
                }
            },String.valueOf(i)).start();
        }
    }
}
```

```java
class MovieTicket{
    int number = 50;

    public synchronized void saleTicket(){
        if(number > 0){
            System.out.println(Thread.currentThread().getName()+"\t"+"号售票员卖出第： "+(number--));
        }else{
            System.out.println("--------卖完了");
        }
    }
}

class House{
    ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public void saleHouse(){
        Integer value = threadLocal.get();
        value++;
        threadLocal.set(value);
    }
}

/**
 * 1  三个售票员卖完50张票务，总量完成即可，吃大锅饭，售票员每个月固定月薪
 *
 * 2  分灶吃饭，各个销售自己动手，丰衣足食
 */
public class ThreadLocalDemo{
    public static void main(String[] args){
        /*MovieTicket movieTicket = new MovieTicket();

        for (int i = 1; i <=3; i++) {
            new Thread(() -> {
                for (int j = 0; j <20; j++) {
                    movieTicket.saleTicket();
                    try { TimeUnit.MILLISECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
                }
            },String.valueOf(i)).start();
        }*/

        //===========================================
        House house = new House();

        new Thread(() -> {
            try {
                for (int i = 1; i <=3; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();//如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题
            }
        },"t1").start();

        new Thread(() -> {
            try {
                for (int i = 1; i <=2; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();
            }
        },"t2").start();

        new Thread(() -> {
            try {
                for (int i = 1; i <=5; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();
            }
        },"t3").start();


        System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
    }
}
```

- 因为每个 Thread 内有自己的实例副本且该副本只由当前线程自己使用
- 既然其它 Thread 不可访问，那就不存在多线程间共享的问题。
- 统一设置初始值，但是每个线程对这个值的修改都是各自线程互相独立的

如何才能不争抢

1 加入synchronized或者Lock控制资源的访问顺序

2 人手一份，大家各自安好，没必要抢夺

### 从阿里ThreadLocal规范开始

![image-20210807142540101](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142540101.png)

#### 非线程安全的SimpleDateFormat

![image-20210807142603657](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142603657.png)上述翻译：SimpleDateFormat中的日期格式不是同步的。推荐（建议）为每个线程创建独立的格式实例。如果多个线程同时访问一个格式，则它必须保持外部同步。

写时间工具类，一般写成静态的成员变量，不知，此种写法的多线程下的危险性！
课堂上讨论一下SimpleDateFormat线程不安全问题，以及解决方法。

```java
import java.text.SimpleDateFormat;
import java.util.Date;


public class DateUtils{
    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    /**
     * 模拟并发环境下使用SimpleDateFormat的parse方法将字符串转换成Date对象
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDate(String stringDate)throws Exception{
        return sdf.parse(stringDate);
    }
    
    public static void main(String[] args) throws Exception{
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DateUtils.parseDate("2020-11-11 11:11:11"));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

![image-20210807142644665](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142644665.png)

SimpleDateFormat类内部有一个Calendar对象引用,它用来储存和这个SimpleDateFormat相关的日期信息,例如sdf.parse(dateStr),sdf.format(date) 诸如此类的方法参数传入的日期相关String,Date等等, 都是交由Calendar引用来储存的.这样就会导致一个问题如果你的SimpleDateFormat是个static的, 那么多个thread 之间就会共享这个SimpleDateFormat, 同时也是共享这个Calendar引用。

![image-20210807142722722](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807142722722.png)

#### 解决1

将SimpleDateFormat定义成局部变量。

缺点：每调用一次方法就会创建一个SimpleDateFormat对象，方法结束又要作为垃圾回收。

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateUtils{
    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    /**
     * 模拟并发环境下使用SimpleDateFormat的parse方法将字符串转换成Date对象
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDate(String stringDate)throws Exception{
        return sdf.parse(stringDate);
    }

    public static void main(String[] args) throws Exception{
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    System.out.println(sdf.parse("2020-11-11 11:11:11"));
                    sdf = null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

#### 解决2

**ThreadLocal，也叫做线程本地变量或者线程本地存储**

```java
import java.text.SimpleDateFormat;
import java.util.Date;


public class DateUtils{
    private static final ThreadLocal<SimpleDateFormat>  sdf_threadLocal =
            ThreadLocal.withInitial(()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    /**
     * ThreadLocal可以确保每个线程都可以得到各自单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDateTL(String stringDate)throws Exception{
        return sdf_threadLocal.get().parse(stringDate);
    }

    public static void main(String[] args) throws Exception{
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DateUtils.parseDateTL("2020-11-11 11:11:11"));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

#### DateUtils

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;

public class DateUtils{
    /*
    1   SimpleDateFormat如果多线程共用是线程不安全的类
    public static final SimpleDateFormat SIMPLE_DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static String format(Date date)
    {
        return SIMPLE_DATE_FORMAT.format(date);
    }

    public static Date parse(String datetime) throws ParseException
    {
        return SIMPLE_DATE_FORMAT.parse(datetime);
    }*/

    //2   ThreadLocal可以确保每个线程都可以得到各自单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。
    public static final ThreadLocal<SimpleDateFormat> SIMPLE_DATE_FORMAT_THREAD_LOCAL = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static String format(Date date)
    {
        return SIMPLE_DATE_FORMAT_THREAD_LOCAL.get().format(date);
    }

    public static Date parse(String datetime) throws ParseException
    {
        return SIMPLE_DATE_FORMAT_THREAD_LOCAL.get().parse(datetime);
    }


    //3 DateTimeFormatter 代替 SimpleDateFormat
    /*public static final DateTimeFormatter DATE_TIME_FORMAT = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static String format(LocalDateTime localDateTime)
    {
        return DATE_TIME_FORMAT.format(localDateTime);
    }

    public static LocalDateTime parse(String dateString)
    {

        return LocalDateTime.parse(dateString,DATE_TIME_FORMAT);
    }*/
}
```

### ThreadLocal源码分析

Thread，ThreadLocal，ThreadLocalMap 关系

#### Thread和ThreadLocal

![image-20210807143021138](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807143021138.png)

#### ThreadLocal和ThreadLocalMap

![image-20210807143035077](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807143035077.png)

![image-20210807143041971](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807143041971.png)

| threadLocalMap实际上就是一个以threadLocal实例为key，任意对象为value的Entry对象。<br/>当我们为threadLocal变量赋值，实际上就是以当前threadLocal实例为key，值为value的Entry往这个threadLocalMap中存放 |
| ------------------------------------------------------------ |

#### 总结

近似的可以理解为:
ThreadLocalMap从字面上就可以看出这是一个保存ThreadLocal对象的map(其实是以ThreadLocal为Key)，不过是经过了两层包装的ThreadLocal对象：

![image-20210807143130038](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807143130038.png)

JVM内部维护了一个线程版的Map<Thread,T>(通过ThreadLocal对象的set方法，结果把ThreadLocal对象自己当做key，放进了ThreadLoalMap中),每个线程要用到这个T的时候，用当前的线程去Map里面获取，通过这样让每个线程都拥有了自己独立的变量，
人手一份，竞争条件被彻底消除，在并发模式下是绝对安全的变量。

### ThreadLocal内存泄露问题

![image-20210807201719813](C:\Users\larry\AppData\Roaming\Typora\typora-user-images\image-20210807201719813.png)

#### 什么是内存泄漏

不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露。

![image-20210807201825625](C:\Users\larry\AppData\Roaming\Typora\typora-user-images\image-20210807201825625.png)

#### 再回首ThreadLocalMap

![image-20210807143035077](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807143035077.png)

![image-20210807202000957](C:\Users\larry\AppData\Roaming\Typora\typora-user-images\image-20210807202000957.png)

**ThreadLocalMap与WeakReference **
ThreadLocalMap从字面上就可以看出这是一个保存ThreadLocal对象的map(其实是以它为Key)，不过是经过了两层包装的ThreadLocal对象：
（1）第一层包装是使用 WeakReference<ThreadLocal<?>> 将ThreadLocal对象变成一个弱引用的对象；
（2）第二层包装是定义了一个专门的类 Entry 来扩展 WeakReference<ThreadLocal<?>>：

####  整体架构

![image-20210807202038843](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202038843.png)

Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。

![image-20210807202109230](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202109230.png)

#### 新建一个带finalize()方法的对象MyObject

```java
 
class MyObject{
    //一般开发中不用调用这个方法，本次只是为了讲课演示
    @Override
    protected void finalize() throws Throwable{
        System.out.println(Thread.currentThread().getName()+"\t"+"---finalize method invoked....");
    }
}
```

#### 强引用(默认支持模式)


*  当内存不足，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收，死都不收。

强引用是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。在 Java 中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到JVM也不会回收。因此强引用是造成Java内存泄漏的主要原因之一。

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，
一般认为就是可以被垃圾收集的了(当然具体回收时机还是要看垃圾收集策略)。

```java
 
public static void strongReference(){
    MyObject myObject = new MyObject();
    System.out.println("-----gc before: "+myObject);

    myObject = null;
    System.gc();
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

    System.out.println("-----gc after: "+myObject);
}
```

#### 软引用


软引用是一种相对强引用弱化了一些的引用，需要用java.lang.ref.SoftReference类来实现，可以让对象豁免一些垃圾收集。

对于只有软引用的对象来说，

         当系统内存充足时它      不会     被回收，
     
         当系统内存不足时它         会     被回收。
软引用通常用在对内存敏感的程序中，比如高速缓存就有用到软引用，内存够用的时候就保留，不够用就回收！

```java
import java.lang.ref.SoftReference;
import java.util.concurrent.TimeUnit;

class MyObject{
    //一般开发中不用调用这个方法，本次只是为了讲课演示
    @Override
    protected void finalize() throws Throwable {
        System.out.println(Thread.currentThread().getName()+"\t"+"---finalize method invoked....");
    }
}


public class ReferenceDemo{
    public static void main(String[] args){
        //当我们内存不够用的时候，soft会被回收的情况，设置我们的内存大小：-Xms10m -Xmx10m
        SoftReference<MyObject> softReference = new SoftReference<>(new MyObject());

        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("-----gc after内存够用: "+softReference.get());

        try
        {
            byte[] bytes = new byte[9 * 1024 * 1024];
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("-----gc after内存不够: "+softReference.get());
        }
    }

    public static void strongReference(){
        MyObject myObject = new MyObject();
        System.out.println("-----gc before: "+myObject);

        myObject = null;
        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("-----gc after: "+myObject);
    }
}
```

#### 弱引用


弱引用需要用java.lang.ref.WeakReference类来实现，它比软引用的生存期更短，

对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。 

```java
import java.lang.ref.SoftReference;
import java.lang.ref.WeakReference;
import java.util.concurrent.TimeUnit;

class MyObject{
    //一般开发中不用调用这个方法，本次只是为了讲课演示
    @Override
    protected void finalize() throws Throwable{
        System.out.println(Thread.currentThread().getName()+"\t"+"---finalize method invoked....");
    }
}


public class ReferenceDemo{
    public static void main(String[] args){
        WeakReference<MyObject> weakReference = new WeakReference<>(new MyObject());
        System.out.println("-----gc before内存够用: "+weakReference.get());

        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("-----gc after内存够用: "+weakReference.get());
    }

    public static void softReference(){
        //当我们内存不够用的时候，soft会被回收的情况，设置我们的内存大小：-Xms10m -Xmx10m
        SoftReference<MyObject> softReference = new SoftReference<>(new MyObject());

        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("-----gc after内存够用: "+softReference.get());

        try{
            byte[] bytes = new byte[9 * 1024 * 1024];
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("-----gc after内存不够: "+softReference.get());
        }
    }

    public static void strongReference(){
        MyObject myObject = new MyObject();
        System.out.println("-----gc before: "+myObject);

        myObject = null;
        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("-----gc after: "+myObject);
    }
}
```

软引用和弱引用的适用场景

假如有一个应用需要读取大量的本地图片:

     *    如果每次读取图片都从硬盘读取则会严重影响性能,
     *    如果一次性全部加载到内存中又可能造成内存溢出。

此时使用软引用可以解决这个问题。

　　设计思路是：用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免了OOM的问题。

Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();

#### 虚引用


 虚引用需要java.lang.ref.PhantomReference类来实现。

顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。
如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，
它不能单独使用也不能通过它访问对象，虚引用必须和引用队列 (ReferenceQueue)联合使用。

虚引用的主要作用是跟踪对象被垃圾回收的状态。 仅仅是提供了一种确保对象被 finalize以后，做某些事情的机制。 PhantomReference的get方法总是返回null，因此无法访问对应的引用对象。

其意义在于：说明一个对象已经进入finalization阶段，可以被gc回收，用来实现比finalization机制更灵活的回收操作。

换句话说，设置虚引用关联的唯一目的，就是在这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理。

 

![image-20210807202504476](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202504476.png)![image-20210807202510695](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202510695.png)

我被回收前需要被引用队列保存下。

```java
import java.lang.ref.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

class MyObject{
    //一般开发中不用调用这个方法，本次只是为了讲课演示
    @Override
    protected void finalize() throws Throwable{
        System.out.println(Thread.currentThread().getName()+"\t"+"---finalize method invoked....");
    }
}


public class ReferenceDemo{
    public static void main(String[] args){
        ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue();
        PhantomReference<MyObject> phantomReference = new PhantomReference<>(new MyObject(),referenceQueue);
        //System.out.println(phantomReference.get());

        List<byte[]> list = new ArrayList<>();

        new Thread(() -> {
            while (true)
            {
                list.add(new byte[1 * 1024 * 1024]);
                try { TimeUnit.MILLISECONDS.sleep(600); } catch (InterruptedException e) { e.printStackTrace(); }
                System.out.println(phantomReference.get());
            }
        },"t1").start();

        new Thread(() -> {
            while (true)
            {
                Reference<? extends MyObject> reference = referenceQueue.poll();
                if (reference != null) {
                    System.out.println("***********有虚对象加入队列了");
                }
            }
        },"t2").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
    }

    public static void weakReference(){
        WeakReference<MyObject> weakReference = new WeakReference<>(new MyObject());
        System.out.println("-----gc before内存够用: "+weakReference.get());

        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("-----gc after内存够用: "+weakReference.get());
    }

    public static void softReference(){
        //当我们内存不够用的时候，soft会被回收的情况，设置我们的内存大小：-Xms10m -Xmx10m
        SoftReference<MyObject> softReference = new SoftReference<>(new MyObject());

        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("-----gc after内存够用: "+softReference.get());

        try
        {
            byte[] bytes = new byte[9 * 1024 * 1024];
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("-----gc after内存不够: "+softReference.get());
        }
    }

    public static void strongReference(){
        MyObject myObject = new MyObject();
        System.out.println("-----gc before: "+myObject);

        myObject = null;
        System.gc();
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("-----gc after: "+myObject);
    }
}
```

#### GCRoots和四大引用

![image-20210807202614872](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202614872.png)

![image-20210807202759881](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202759881.png)

![image-20210807202806535](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202806535.png)

每个Thread对象维护着一个ThreadLocalMap的引用
ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储
调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值Value是传递进来的对象
调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象
ThreadLocal本身并不存储值，它只是自己作为一个key来让线程从ThreadLocalMap获取value，正因为这个原理，所以ThreadLocal能够实现“数据隔离”，获取当前线程的局部变量值，不受其他线程影响～

#### 为什么要用弱引用?不用如何？

```java
public void function01(){
    ThreadLocal tl = new ThreadLocal<Integer>();    //line1
    tl.set(2021);                                   //line2
    tl.get();                                       //line3
}
```

line1新建了一个ThreadLocal对象，t1 是强引用指向这个对象；
line2调用set()方法后新建一个Entry，通过源码可知Entry对象里的k是弱引用指向这个对象。

![image-20210807202905226](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202905226.png)

![image-20210807202915709](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202915709.png)

![image-20210807202955120](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807202955120.png)

**用完记得手动remove**

#### 总结

1. ThreadLocal 并不解决线程间共享数据的问题
2. ThreadLocal 适用于变量在线程间隔离且在方法间共享的场景
3. ThreadLocal 通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题
4. 每个线程持有一个只属于自己的专属Map并维护了ThreadLocal对象与具体实例的映射，该Map由于只被持有它的线程访问，故不存在线程安全以及锁的问题
5. ThreadLocalMap的Entry对ThreadLocal的引用为弱引用，避免了ThreadLocal对象无法被回收的问题
6. 都会通过expungeStaleEntry，cleanSomeSlots,replaceStaleEntry这三个方法回收键为 null 的 Entry 对象的值（即为具体实例）以及 Entry 对象本身从而防止内存泄漏，属于安全加固的方法

## Java对象内存布局和对象头

### 对象在堆内存中布局

![image-20210807203335613](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807203335613.png)

![image-20210807203340478](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807203340478.png)

### 对象在堆内存中的存储布局

![image-20210807203410382](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807203410382.png)

对象内部结构分为：对象头、实例数据、对齐填充（保证8个字节的倍数）。
对象头分为对象标记（markOop）和类元信息（klassOop），类元信息存储的是指向该对象类元数据（klass）的首地址。

#### 对象头

##### 对象标记Mark Word

![image-20210807203501251](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807203501251.png)

![image-20210807203520374](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807203520374.png)

**在64位系统中，Mark Word占了8个字节，类型指针占了8个字节，一共是16个字节**

![image-20210807204406173](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204406173.png)

默认存储对象的HashCode、分代年龄和锁标志位等信息。
这些信息都是与对象自身定义无关的数据，所以MarkWord被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。
它会根据对象的状态复用自己的存储空间，也就是说在运行期间MarkWord里存储的数据会随着锁标志位的变化而变化。

##### 类元信息(又叫类型指针)

![image-20210807204449946](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204449946.png)

对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

#### 实例数据

存放类的属性(Field)数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。

#### 对齐填充

虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐这部分内存按8字节补充对齐。

http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html

![image-20210807204619148](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204619148.png)

http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/89fb452b3688/src/share/vm/oops/oop.hpp

![image-20210807204647043](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204647043.png)

_mark字段是mark word，_metadata是类指针klass pointer，对象头（object header）即是由这两个字段组成，这些术语可以参考Hotspot术语表，

![image-20210807204703339](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204703339.png)

### 再说对象头的MarkWord

![image-20210807204728454](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204728454.png)

![image-20210807204735968](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204735968.png)

![image-20210807204750692](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204750692.png)![image-20210807204823733](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807204823733.png)

markword(64位)分布图，对象布局、GC回收和后面的锁升级就是对象标记MarkWord里面标志位的变化

### 聊聊Object obj = new Object()

#### JOL证明

http://openjdk.java.net/projects/code-tools/jol/

```xml
 
<!--
官网：http://openjdk.java.net/projects/code-tools/jol/
定位：分析对象在JVM的大小和分布
-->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

```java
import org.openjdk.jol.vm.VM;

public class MyObject{
    public static void main(String[] args){
        //VM的细节详细情况
        System.out.println(VM.current().details());
        //所有的对象分配的字节都是8的整数倍。
        System.out.println(VM.current().objectAlignment());
    }
}
```

```java
import org.openjdk.jol.info.ClassLayout;


public class JOLDemo{
    public static void main(String[] args){
        Object o = new Object();
        System.out.println( ClassLayout.parseInstance(o).toPrintable());
    }
}
 

```

**GC年龄采用4位bit存储，最大为15，例如MaxTenuringThreshold参数默认值就是15**

-XX:MaxTenuringThreshold=16

![image-20210807205103379](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205103379.png)

java -XX:+PrintCommandLineFlags -version

![image-20210807205124481](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205124481.png)

**默认开启压缩说明**

-XX:+UseCompressedClassPointers



![image-20210807205137501](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205137501.png)

-XX:-UseCompressedClassPointers

![image-20210807205152351](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205152351.png)

### 换成其他对象试试

![image-20210807205230189](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205230189.png)![image-20210807205235228](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205235228.png)

## Synchronized与锁升级

synchronized 锁优化的背景
用锁能够实现数据的安全性，但是会带来性能下降。
无锁能够基于线程并行提升程序性能，但是会带来安全性下降。
求平衡？？？

![image-20210807205336004](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205336004.png)

synchronized锁：由对象头中的Mark Word根据锁标志位的不同而被复用及锁升级策略

### Synchronized的性能变化

java5以前，只有Synchronized，这个是操作系统级别的重量级操作

重量级锁，假如锁的竞争比较激烈的话，性能下降

Java5之前，用户态和内核态之间的切换

![image-20210807205428177](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205428177.png)

java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在户态与核心态之间切换，这种切换会消耗大量的系统资源，因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作。

在Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock来实现的，挂起线程和恢复线程都需要转入内核态去完成，阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态切换需要耗费处理器时间，如果同步代码块中内容过于简单，这种切换的时间可能比用户代码执行的时间还长”，时间成本相对较高，这也是为什么早期的synchronized效率低的原因
Java 6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁

#### 为什么每一个对象都可以成为一个锁？？？？

##### markOop.hpp

![image-20210807205502770](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205502770.png)Monitor可以理解为一种同步工具，也可理解为一种同步机制，常常被描述为一个Java对象。Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。

![image-20210807205537822](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205537822.png)

Monitor的本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，成本非常高。

##### Monitor(监视器锁)

![image-20210807205621079](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205621079.png)

Mutex Lock 
Monitor是在jvm底层实现的，底层代码是c++。本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，状态转换需要耗费很多的处理器时间成本非常高。所以synchronized是Java语言中的一个重量级操作。 


Monitor与java对象以及线程是如何关联 ？
1.如果一个java对象被某个线程锁住，则该java对象的Mark Word字段中LockWord指向monitor的起始地址
2.Monitor的Owner字段会存放拥有相关联对象锁的线程id

Mutex Lock 的切换需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间。

#### java6开始，优化Synchronized

- Java 6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁
- 需要有个逐步升级的过程，别一开始就捅到重量级锁

### synchronized锁种类及升级步骤

多线程访问情况，3种:

- 只有一个线程来访问，有且唯一Only One
- 有2个线程A、B来交替访问
- 竞争激烈，多个线程来访问

升级流程:

- synchronized用的锁是存在Java对象头里的Mark Word中锁升级功能主要依赖MarkWord中锁标志位和释放偏向锁标志位
- ![image-20210807205912377](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205912377.png)

#### 无锁

```java
 
import org.openjdk.jol.info.ClassLayout;

public class MyObject{
    public static void main(String[] args){
        Object o = new Object();

        System.out.println("10进制hash码："+o.hashCode());
        System.out.println("16进制hash码："+Integer.toHexString(o.hashCode()));
        System.out.println("2进制hash码："+Integer.toBinaryString(o.hashCode()));

        System.out.println( ClassLayout.parseInstance(o).toPrintable());
    }
}
```

程序不会有锁的竞争

![image-20210807205953750](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807205953750.png)

![image-20210807210012983](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210012983.png)

#### 偏锁

##### 主要作用

- 当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁

- Hotspot 的作者经过研究发现，大多数情况下：

  多线程的情况下，锁不仅不存在多线程竞争，还存在锁由同一线程多次获得的情况，

  偏向锁就是在这种情况下出现的，它的出现是为了解决只有在一个线程执行同步时提高性能。

![image-20210807210121813](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210121813.png)

通过CAS方式修改markword中的线程ID

##### 偏向锁的持有

理论落地：
      在实际应用运行过程中发现，“锁总是同一个线程持有，很少发生竞争”，也就是说锁总是被第一个占用他的线程拥有，这个线程就是锁的偏向线程。
      那么只需要在锁第一次被拥有的时候，记录下偏向线程ID。这样偏向线程就一直持有着锁(后续这个线程进入和退出这段加了同步锁的代码块时，不需要再次加锁和释放锁。而是直接比较对象头里面是否存储了指向当前线程的偏向锁)。
如果相等表示偏向锁是偏向于当前线程的，就不需要再尝试获得锁了，直到竞争发生才释放锁。以后每次同步，检查锁的偏向线程ID与当前线程ID是否一致，如果一致直接进入同步。无需每次加锁解锁都去CAS更新对象头。如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外开销，性能极高。
      假如不一致意味着发生了竞争，锁已经不是总是偏向于同一个线程了，这时候可能需要升级变为轻量级锁，才能保证线程间公平竞争锁。偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程是不会主动释放偏向锁的。

技术实现：
一个synchronized方法被一个线程抢到了锁时，那这个方法所在的对象就会在其所在的Mark Word中将偏向锁修改状态位，同时还
会有占用前54位来存储线程指针作为标识。若该线程再次访问同一个synchronized方法时，该线程只需去对象头的Mark Word 中去判断一下是否有偏向锁指向本身的ID，无需再进入 Monitor 去竞争对象了。

![image-20210807210159856](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210159856.png)

偏向锁的操作不用直接捅到操作系统，不涉及用户到内核转换，不必要直接升级为最高级，我们以一个account对象的“对象头”为例，

![image-20210807210227066](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210227066.png)

假如有一个线程执行到synchronized代码块的时候，JVM使用CAS操作把线程指针ID记录到Mark Word当中，并修改标偏向标示，标示当前线程就获得该锁。锁对象变成偏向锁（通过CAS修改对象头里的锁标志位），字面意思是“偏向于第一个获得它的线程”的锁。执行完同步代码块后，线程并不会主动释放偏向锁。

![image-20210807210236724](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210236724.png)

这时线程获得了锁，可以执行同步代码块。当该线程第二次到达同步代码块时会判断此时持有锁的线程是否还是自己（持有锁的线程ID也在对象头里），JVM通过account对象的Mark Word判断：当前线程ID还在，说明还持有着这个对象的锁，就可以继续进入临界区工作。由于之前没有释放锁，这里也就不需要重新加锁。 如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外开销，性能极高。

结论：JVM不用和操作系统协商设置Mutex(争取内核)，它只需要记录下线程ID就标示自己获得了当前锁，不用操作系统接入。
上述就是偏向锁：在没有其他线程竞争的时候，一直偏向偏心当前线程，当前线程可以一直执行。

##### 偏向锁JVM命令

java -XX:+PrintFlagsInitial |grep BiasedLock*

![image-20210807210301396](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210301396.png)

![image-20210807210308615](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210308615.png)

| * 实际上偏向锁在JDK1.6之后是默认开启的，但是启动时间有延迟，<br/>* 所以需要添加参数-XX:BiasedLockingStartupDelay=0，让其在程序启动时立刻启动。<br/>*<br/>* 开启偏向锁：<br/>* -XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0<br/>*<br/>* 关闭偏向锁：关闭之后程序默认会直接进入------------------------------------------>>>>>>>>   轻量级锁状态。<br/>* -XX:-UseBiasedLocking |
| ------------------------------------------------------------ |

![image-20210807210330813](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210330813.png)

```java
import org.openjdk.jol.info.ClassLayout;

public class MyObject{
    public static void main(String[] args){
        Object o = new Object();

        new Thread(() -> {
            synchronized (o){
                System.out.println(ClassLayout.parseInstance(o).toPrintable());
            }
        },"t1").start();
    }
}
```

![image-20210807210357441](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210357441.png)

![image-20210807210405967](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210405967.png)

| -XX:+UseBiasedLocking                       开启偏向锁(默认)           <br/>-XX:-UseBiasedLocking                       关闭偏向锁<br/>-XX:BiasedLockingStartupDelay=0             关闭延迟(演示偏向锁时需要开启) |
| ------------------------------------------------------------ |

| 参数说明：<br/>偏向锁在JDK1.6以上默认开启，开启后程序启动几秒后才会被激活，可以使用JVM参数来关闭延迟 -XX:BiasedLockingStartupDelay=0<br/> <br/>如果确定锁通常处于竞争状态则可通过JVM参数 -XX:-UseBiasedLocking 关闭偏向锁，那么默认会进入轻量级锁 |
| ------------------------------------------------------------ |

关闭延时参数，启用该功能

![image-20210807210440419](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210440419.png)

**-XX:BiasedLockingStartupDelay=0**

##### 偏向锁的撤销

- 当有另外线程逐步来竞争锁的时候，就不能再使用偏向锁了，要升级为轻量级锁
- 竞争线程尝试CAS更新对象头失败，会等待到全局安全点（此时不会执行任何代码）撤销偏向锁。


偏向锁的撤销
偏向锁使用一种等到竞争出现才释放锁的机制，只有当其他线程竞争锁时，持有偏向锁的原来线程才会被撤销。
撤销需要等待全局安全点(该时间点上没有字节码正在执行)，同时检查持有偏向锁的线程是否还在执行： 

①  第一个线程正在执行synchronized方法(处于同步块)，它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级。
此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
②  第一个线程执行完成synchronized方法(退出同步块)，则将对象头设置成无锁状态并撤销偏向锁，重新偏向 。

 

![image-20210807210550400](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210550400.png)



![image-20210807210658601](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807210658601.png)

#### 轻锁

主要作用:

- 有线程来参与锁的竞争，但是获取锁的冲突时间极短
- 本质就是自旋锁

##### 轻量级锁的获取

轻量级锁是为了在线程近乎交替执行同步块时提高性能。
主要目的： 在没有多线程竞争的前提下，通过CAS减少重量级锁使用操作系统互斥量产生的性能消耗，说白了先自旋再阻塞。
升级时机： 当关闭偏向锁功能或多线程竞争偏向锁会导致偏向锁升级为轻量级锁

假如线程A已经拿到锁，这时线程B又来抢该对象的锁，由于该对象的锁已经被线程A拿到，当前该锁已是偏向锁了。
而线程B在争抢时发现对象头Mark Word中的线程ID不是线程B自己的线程ID(而是线程A)，那线程B就会进行CAS操作希望能获得锁。
此时线程B操作中有两种情况：
如果锁获取成功，直接替换Mark Word中的线程ID为B自己的ID(A → B)，重新偏向于其他线程(即将偏向锁交给其他线程，相当于当前线程"被"释放了锁)，该锁会保持偏向锁状态，A线程Over，B线程上位；

![image-20210807213744584](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807213744584.png)

如果锁获取失败，则偏向锁升级为轻量级锁，此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程B会进入自旋等待获得该轻量级锁。

![image-20210807213756396](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807213756396.png)

![image-20210807213804387](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807213804387.png)

如果关闭偏向锁，就可以直接进入轻量级锁

-XX:-UseBiasedLocking

![image-20210807213838317](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807213838317.png)

##### 自旋达到一定次数和程度

java6之前:

- 默认启用，默认情况下自旋的次数是 10 次 -XX:PreBlockSpin=10来修改
- 或者自旋线程数超过cpu核数一半

Java6之后:

自适应意味着自旋的次数不是固定不变的

而是根据：同一个锁上一次自旋的时间。拥有锁线程的状态来决定。

##### 轻量锁与偏向锁的区别和不同

争夺轻量级锁失败时，自旋尝试抢占锁

轻量级锁每次退出同步块都需要释放锁，而偏向锁是在竞争发生时才释放锁

#### 重锁

有大量的线程参与锁的竞争，冲突性很高

![image-20210807214057421](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214057421.png)

![image-20210807214104398](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214104398.png)

#### 各种锁优缺点、synchronized锁升级和实现原理

![image-20210807214123719](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214123719.png)

synchronized锁升级过程总结：一句话，就是先自旋，不行再阻塞。
实际上是把之前的悲观锁(重量级锁)变成在一定条件下使用偏向锁以及使用轻量级(自旋锁CAS)的形式

synchronized在修饰方法和代码块在字节码上实现方式有很大差异，但是内部实现还是基于对象头的MarkWord来实现的。
JDK1.6之前synchronized使用的是重量级锁，JDK1.6之后进行了优化，拥有了无锁->偏向锁->轻量级锁->重量级锁的升级过程，而不是无论什么情况都使用重量级锁。

偏向锁:适用于单线程适用的情况，在不存在锁竞争的时候进入同步方法/代码块则使用偏向锁。
轻量级锁：适用于竞争较不激烈的情况(这和乐观锁的使用范围类似)， 存在竞争时升级为轻量级锁，轻量级锁采用的是自旋锁，如果同步方法/代码块执行时间很短的话，采用轻量级锁虽然会占用cpu资源但是相对比使用重量级锁还是更高效。
重量级锁：适用于竞争激烈的情况，如果同步方法/代码块执行时间很长，那么使用轻量级锁自旋带来的性能消耗就比使用重量级锁更严重，这时候就需要升级为重量级锁。

### JIT编译器对锁的优化

#### JIT

Just In Time Compiler，一般翻译为即时编译器

#### 锁消除

```java

/**
 * 锁消除
 * 从JIT角度看相当于无视它，synchronized (o)不存在了,这个锁对象并没有被共用扩散到其它线程使用，
 * 极端的说就是根本没有加这个锁对象的底层机器码，消除了锁的使用
 */
public class LockClearUPDemo{
    static Object objectLock = new Object();//正常的

    public void m1(){
        //锁消除,JIT会无视它，synchronized(对象锁)不存在了。不正常的
        Object o = new Object();

        synchronized (o){
            System.out.println("-----hello LockClearUPDemo"+"\t"+o.hashCode()+"\t"+objectLock.hashCode());
        }
    }

    public static void main(String[] args){
        LockClearUPDemo demo = new LockClearUPDemo();

        for (int i = 1; i <=10; i++) {
            new Thread(() -> {
                demo.m1();
            },String.valueOf(i)).start();
        }
    }
}
```

#### 锁粗化

```java

/**
 * 锁粗化
 * 假如方法中首尾相接，前后相邻的都是同一个锁对象，那JIT编译器就会把这几个synchronized块合并成一个大块，
 * 加粗加大范围，一次申请锁使用即可，避免次次的申请和释放锁，提升了性能
 */
public class LockBigDemo{
    static Object objectLock = new Object();


    public static void main(String[] args){
        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("11111");
            }
            synchronized (objectLock) {
                System.out.println("22222");
            }
            synchronized (objectLock) {
                System.out.println("33333");
            }
        },"a").start();

        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("44444");
            }
            synchronized (objectLock) {
                System.out.println("55555");
            }
            synchronized (objectLock) {
                System.out.println("66666");
            }
        },"b").start();

    }
}
```

## AbstractQueuedSynchronizer之AQS

### 是什么

抽象的队列同步器

![image-20210807214353593](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214353593.png)

AbstractOwnableSynchronizer
AbstractQueuedLongSynchronizer
AbstractQueuedSynchronizer                  **通常地：AbstractQueuedSynchronizer简称为AQS**

**是用来构建锁或者其它同步器组件的重量级基础框架及整个JUC体系的基石，通过内置的FIFO队列来完成资源获取线程的排队工作，并通过一个int类变量表示持有锁的状态**

![image-20210807214432643](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214432643.png)

| CLH：Craig、Landin and Hagersten 队列，是一个单向链表，AQS中的队列是CLH变体的虚拟双向队列FIFO |
| ------------------------------------------------------------ |

### AQS为什么是JUC内容中最重要的基石

![image-20210807214509842](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214509842.png)

#### ReentrantLock

![image-20210807214539829](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214539829.png)

#### CountDownLatch

![image-20210807214550872](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214550872.png)

#### ReentrantReadWriteLock

![image-20210807214600939](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214600939.png)

#### Semaphore

![image-20210807214611413](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214611413.png)

#### 进一步理解锁和同步器的关系

锁，面向锁的使用者 --- 定义了程序员和锁交互的使用层API，隐藏了实现细节，你调用即可。

同步器，面向锁的实现者 --- 比如Java并发大神DougLee，提出统一规范并简化了锁的实现，屏蔽了同步状态管理、阻塞线程排队和通知、唤醒机制等。

**加锁会导致阻塞 --- 有阻塞就需要排队，实现排队必然需要队列**

抢到资源的线程直接使用处理业务，抢不到资源的必然涉及一种排队等候机制。抢占资源失败的线程继续去等待(类似银行业务办理窗口都满了，暂时没有受理窗口的顾客只能去候客区排队等候)，但等候线程仍然保留获取锁的可能且获取锁流程仍在继续(候客区的顾客也在等着叫号，轮到了再去受理窗口办理业务)。

既然说到了排队等候机制，那么就一定会有某种队列形成，这样的队列是什么数据结构呢？

如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象表现。它将请求共享资源的线程封装成队列的结点（Node），通过CAS、自旋以及LockSupport.park()的方式，维护state变量的状态，使并发达到同步的效果。

![image-20210807214741631](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214741631.png)

### AQS初识

![image-20210807214809628](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214809628.png)

AQS使用一个volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作将每条要去抢占资源的线程封装成一个Node节点来实现锁的分配，通过CAS完成对State值的修改。

![image-20210807214831488](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214831488.png)

### AQS内部体系架构

![image-20210807214855673](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214855673.png)

![image-20210807214831488](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807214831488.png)

#### AQS的int变量

AQS的同步状态State成员变量

![image-20210807215009495](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215009495.png)

#### AQS的CLH队列

![image-20210807215031030](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215031030.png)

**有阻塞就需要排队，实现排队必然需要队列 --- state变量+CLH双端队列**

#### 内部类Node(Node类在AQS类内部)

##### Node的int变量

Node的等待状态waitState成员变量 --- volatile int waitStatus

![image-20210807215200907](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215200907.png)

![image-20210807215209209](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215209209.png)

#### AQS同步队列的基本结构

![image-20210807215224392](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215224392.png)

| CLH：Craig、Landin and Hagersten 队列，是个单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO） |
| ------------------------------------------------------------ |

### 从我们的ReentrantLock开始解读AQS

Lock接口的实现类，基本都是通过【聚合】了一个【队列同步器】的子类完成线程访问控制的

#### ReentrantLock的原理

![image-20210807215312282](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215312282.png)

#### 从最简单的lock方法开始看看公平和非公平

![image-20210807215357185](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215357185.png)

![image-20210807215403390](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215403390.png)

| 可以明显看出公平锁与非公平锁的lock()方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：<br/>hasQueuedPredecessors()<br/>hasQueuedPredecessors是公平锁加锁时判断等待队列中是否存在有效节点的方法 |
| ------------------------------------------------------------ |

![image-20210807215422232](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215422232.png)

#### 非公平锁走起，方法lock()

对比公平锁和非公平锁的 tryAcquire()方法的实现代码，其实差别就在于非公平锁获取锁时比公平锁中少了一个判断 !hasQueuedPredecessors()

hasQueuedPredecessors() 中判断了是否需要排队，导致公平锁和非公平锁的差异如下：

公平锁：公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中；

非公平锁：不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说队列的第一个排队线程在unpark()，之后还是需要竞争锁（存在线程竞争的情况下）

![image-20210807215447633](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215447633.png)

##### lock()

![image-20210807215506101](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215506101.png)

##### acquire()

![image-20210807215521416](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215521416.png)

![image-20210807215526880](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215526880.png)

##### tryAcquire(arg)

![image-20210807215544097](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215544097.png)

![image-20210807215548543](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215548543.png)

nonfairTryAcquire(acquires)

![image-20210807215600757](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215600757.png)

return false;

return true;

##### addWaiter(Node.EXCLUSIVE)

addWaiter(Node mode)

![image-20210807215639929](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215639929.png)

假如3号ThreadC线程进来:

- prev
- compareAndSetTail
- next

##### acquireQueued(addWaiter(Node.EXCLUSIVE), arg)

acquireQueued

![image-20210807215726858](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215726858.png)

假如再抢抢失败就会进入shouldParkAfterFailedAcquire 和 parkAndCheckInterrupt 方法中

![image-20210807215803069](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215803069.png)

shouldParkAfterFailedAcquire 

![image-20210807215812954](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215812954.png)

如果前驱节点的 waitStatus 是 SIGNAL状态，即 shouldParkAfterFailedAcquire 方法会返回 true 程序会继续向下执行 parkAndCheckInterrupt 方法，用于将当前线程挂起

parkAndCheckInterrupt 

![image-20210807215837208](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807215837208.png)

#### unlock

sync.release(1);

tryRelease(arg)

unparkSuccessor

## ReentrantLock、ReentrantReadWriteLock、StampedLock

无锁→独占锁→读写锁→邮戳锁

### ReentrantReadWriteLock

读写锁定义为
一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。

『读写锁』意义和特点


『读写锁ReentrantReadWriteLock』并不是真正意义上的读写分离，它只允许读读共存，而读写和写写依然是互斥的，
大多实际场景是“读/读”线程间并不存在互斥关系，只有"读/写"线程或"写/写"线程间的操作需要互斥的。因此引入ReentrantReadWriteLock。


一个ReentrantReadWriteLock同时只能存在一个写锁但是可以存在多个读锁，但不能同时存在写锁和读锁(切菜还是拍蒜选一个)。
也即一个资源可以被多个读操作访问或一个写操作访问，但两者不能同时进行。


只有在读多写少情境之下，读写锁才具有较高的性能体现。

####  特点

- 可重入
- 读写分离
- 无锁无序→加锁→读写锁演变复习



```java

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class MyResource{
    Map<String,String> map = new HashMap<>();
    //=====ReentrantLock 等价于 =====synchronized
    Lock lock = new ReentrantLock();
    //=====ReentrantReadWriteLock 一体两面，读写互斥，读读共享
    ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void write(String key,String value){
        rwLock.writeLock().lock();
        try
        {
            System.out.println(Thread.currentThread().getName()+"\t"+"---正在写入");
            map.put(key,value);
            //暂停毫秒
            try { TimeUnit.MILLISECONDS.sleep(500); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t"+"---完成写入");
        }finally {
            rwLock.writeLock().unlock();
        }
    }
    public void read(String key){
        rwLock.readLock().lock();
        try
        {
            System.out.println(Thread.currentThread().getName()+"\t"+"---正在读取");
            String result = map.get(key);
            //后续开启注释修改为2000，演示一体两面，读写互斥，读读共享，读没有完成时候写锁无法获得
            //try { TimeUnit.MILLISECONDS.sleep(200); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t"+"---完成读取result："+result);
        }finally {
            rwLock.readLock().unlock();
        }
    }
}





public class ReentrantReadWriteLockDemo{
    public static void main(String[] args){
        MyResource myResource = new MyResource();

        for (int i = 1; i <=10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI +"", finalI +"");
            },String.valueOf(i)).start();
        }

        for (int i = 1; i <=10; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.read(finalI +"");
            },String.valueOf(i)).start();
        }

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        //读全部over才可以继续写
        for (int i = 1; i <=3; i++) {
            int finalI = i;
            new Thread(() -> {
                myResource.write(finalI +"", finalI +"");
            },"newWriteThread==="+String.valueOf(i)).start();
        }
    }
}
```

#### 从写锁→读锁，ReentrantReadWriteLock可以降级

![image-20210807220308298](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220308298.png)锁降级：将写入锁降级为读锁(类似Linux文件读写权限理解，就像写权限要高于读权限一样)

#### 读写锁降级演示

##### 可以降级

锁降级：遵循获取写锁→再获取读锁→再释放写锁的次序，写锁能够降级成为读锁。
如果一个线程占有了写锁，在不释放写锁的情况下，它还能占有读锁，即写锁降级为读锁。

![image-20210807220355849](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220355849.png)

![image-20210807220410803](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220410803.png)

**重入还允许通过获取写入锁定，然后读取锁然后释放写锁从写锁到读取锁, 但是，从读锁定升级到写锁是不可能的。**

锁降级是为了让当前线程感知到数据的变化，目的是保证数据可见性

```java

import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 锁降级：遵循获取写锁→再获取读锁→再释放写锁的次序，写锁能够降级成为读锁。
 *
 * 如果一个线程占有了写锁，在不释放写锁的情况下，它还能占有读锁，即写锁降级为读锁。
 */
public class LockDownGradingDemo{
    public static void main(String[] args){
        ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

        ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();
        ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();


        writeLock.lock();
        System.out.println("-------正在写入");


        readLock.lock();
        System.out.println("-------正在读取");

        writeLock.unlock();

    }
}
```

**如果有线程在读，那么写线程是无法获取写锁的，是悲观锁的策略**

##### 不可锁升级

线程获取读锁是不能直接升级为写入锁的。

![image-20210807220527242](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220527242.png)

![image-20210807220532269](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220532269.png)

**在ReentrantReadWriteLock中，当读锁被使用时，如果有线程尝试获取写锁，该写线程会被阻塞。**
**所以，需要释放所有读锁，才可获取写锁，**

![image-20210807220545407](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220545407.png)

#### 写锁和读锁是互斥的


写锁和读锁是互斥的（这里的互斥是指线程间的互斥，
当前线程可以获取到写锁又获取到读锁，但是获取到了读锁不能继续获取写锁），这是因为读写锁要保持写操作的可见性。
因为，如果允许读锁在被获取的情况下对写锁的获取，那么正在运行的其他读线程无法感知到当前写线程的操作。

因此，
分析读写锁ReentrantReadWriteLock，会发现它有个潜在的问题：
读锁全完，写锁有望；写锁独占，读写全堵；
如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，见前面Case《code演示LockDownGradingDemo》
即ReadWriteLock读的过程中不允许写，只有等待线程都释放了读锁，当前线程才能获取写锁，
也就是写入必须等待，这是一种悲观的读锁，o(╥﹏╥)o，人家还在读着那，你先别去写，省的数据乱。

================================后续讲解StampedLock时再详细展开=======================
分析StampedLock(后面详细讲解)，会发现它改进之处在于：
读的过程中也允许获取写锁介入(相当牛B，读和写两个操作也让你“共享”(注意引号))，这样会导致我们读的数据就可能不一致！
所以，需要额外的方法来判断读的过程中是否有写入，这是一种乐观的读锁，O(∩_∩)O哈哈~。 
显然乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。

####  读写锁之读写规矩，再说降级

锁降级  下面的示例代码摘自ReentrantWriteReadLock源码中：
ReentrantWriteReadLock支持锁降级，遵循按照获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁，不支持锁升级。
解读在最下面:

![image-20210807220652812](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20210807220652812.png)

| 1 代码中声明了一个volatile类型的cacheValid变量，保证其可见性。<br/> <br/>2 首先获取读锁，如果cache不可用，则释放读锁，获取写锁，在更改数据之前，再检查一次cacheValid的值，然后修改数据，将cacheValid置为true，然后在释放写锁前获取读锁；此时，cache中数据可用，处理cache中数据，最后释放读锁。这个过程就是一个完整的锁降级的过程，目的是保证数据可见性。<br/> <br/>如果违背锁降级的步骤 <br/>如果当前的线程C在修改完cache中的数据后，没有获取读锁而是直接释放了写锁，那么假设此时另一个线程D获取了写锁并修改了数据，那么C线程无法感知到数据已被修改，则数据出现错误。<br/> <br/>如果遵循锁降级的步骤 <br/>线程C在释放写锁之前获取读锁，那么线程D在获取写锁时将被阻塞，直到线程C完成数据处理过程，释放读锁。这样可以保证返回的数据是这次更新的数据，该机制是专门为了缓存设计的。 |
| ------------------------------------------------------------ |

### 邮戳锁StampedLock

无锁→独占锁→读写锁→邮戳锁

StampedLock是JDK1.8中新增的一个读写锁，也是对JDK1.5中的读写锁ReentrantReadWriteLock的优化。

邮戳锁也叫票据锁

stamp（戳记，long类型）代表了锁的状态。当stamp返回零时，表示线程获取锁失败。并且，当释放锁或者转换锁的时候，都要传入最初获取的stamp值。

**锁饥饿问题：**

ReentrantReadWriteLock实现了读写分离，但是一旦读操作比较多的时候，想要获取写锁就变得比较困难了，
假如当前1000个线程，999个读，1个写，有可能999个读取线程长时间抢到了锁，那1个写线程就悲剧了 
因为当前有可能会一直存在读锁，而无法获得写锁，根本没机会写，o(╥﹏╥)o

 使用“公平”策略可以一定程度上缓解这个问题new ReentrantReadWriteLock(true);但是“公平”策略是以牺牲系统吞吐量为代价的

| ReentrantReadWriteLock<br/>允许多个线程同时读，但是只允许一个线程写，在线程获取到写锁的时候，其他写操作和读操作都会处于阻塞状态，<br/>读锁和写锁也是互斥的，所以在读的时候是不允许写的，读写锁比传统的synchronized速度要快很多，<br/>原因就是在于ReentrantReadWriteLock支持读并发<br/> <br/> <br/>StampedLock横空出世<br/>ReentrantReadWriteLock的读锁被占用的时候，其他线程尝试获取写锁的时候会被阻塞。<br/>但是，StampedLock采取乐观获取锁后，其他线程尝试获取写锁时不会被阻塞，这其实是对读锁的优化，<br/>所以，在获取乐观读锁后，还需要对结果进行校验。 |
| ------------------------------------------------------------ |

#### StampedLock的特点

- 所有获取锁的方法，都返回一个邮戳（Stamp），Stamp为零表示获取失败，其余都表示成功；
- 所有释放锁的方法，都需要一个邮戳（Stamp），这个Stamp必须是和成功获取锁时得到的Stamp一致；
- StampedLock是不可重入的，危险(如果一个线程已经持有了写锁，再去获取写锁的话就会造成死锁)

StampedLock有三种访问模式:

- ①Reading（读模式）：功能和ReentrantReadWriteLock的读锁类似
- ②Writing（写模式）：功能和ReentrantReadWriteLock的写锁类似
- ③Optimistic reading（乐观读模式）：无锁机制，类似于数据库中的乐观锁，支持读写并发，很乐观认为读取时没人修改，假如被修改再实现升级为悲观读模式

乐观读模式code演示

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.StampedLock;


public class StampedLockDemo{
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write(){
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName()+"\t"+"=====写线程准备修改");
        try
        {
            number = number + 13;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName()+"\t"+"=====写线程结束修改");
    }

    //悲观读
    public void read(){
        long stamp = stampedLock.readLock();
        System.out.println(Thread.currentThread().getName()+"\t come in readlock block,4 seconds continue...");
        //暂停几秒钟线程
        for (int i = 0; i <4 ; i++) {
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t 正在读取中......");
        }
        try
        {
            int result = number;
            System.out.println(Thread.currentThread().getName()+"\t"+" 获得成员变量值result：" + result);
            System.out.println("写线程没有修改值，因为 stampedLock.readLock()读的时候，不可以写，读写互斥");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            stampedLock.unlockRead(stamp);
        }
    }

    //乐观读
    public void tryOptimisticRead()
    {
        long stamp = stampedLock.tryOptimisticRead();
        int result = number;
        //间隔4秒钟，我们很乐观的认为没有其他线程修改过number值，实际靠判断。
        System.out.println("4秒前stampedLock.validate值(true无修改，false有修改)"+"\t"+stampedLock.validate(stamp));
        for (int i = 1; i <=4 ; i++) {
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t 正在读取中......"+i+
                    "秒后stampedLock.validate值(true无修改，false有修改)"+"\t"
                    +stampedLock.validate(stamp));
        }
        if(!stampedLock.validate(stamp)) {
            System.out.println("有人动过--------存在写操作！");
            stamp = stampedLock.readLock();
            try {
                System.out.println("从乐观读 升级为 悲观读");
                result = number;
                System.out.println("重新悲观读锁通过获取到的成员变量值result：" + result);
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                stampedLock.unlockRead(stamp);
            }
        }
        System.out.println(Thread.currentThread().getName()+"\t finally value: "+result);
    }

    public static void main(String[] args)
    {
        StampedLockDemo resource = new StampedLockDemo();

        new Thread(() -> {
            resource.read();
            //resource.tryOptimisticRead();
        },"readThread").start();

        // 2秒钟时乐观读失败，6秒钟乐观读取成功resource.tryOptimisticRead();，修改切换演示
        //try { TimeUnit.SECONDS.sleep(6); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            resource.write();
        },"writeThread").start();
    }
}
```

**读的过程中也允许获取写锁介入**

#### StampedLock的缺点

- StampedLock 不支持重入，没有Re开头
- StampedLock 的悲观读锁和写锁都不支持条件变量（Condition），这个也需要注意。
- 使用 StampedLock一定不要调用中断操作，即不要调用interrupt() 方法,如果需要支持中断功能，一定使用可中断的悲观读锁 readLockInterruptibly()和写锁writeLockInterruptibly()
