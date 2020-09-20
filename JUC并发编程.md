

## JUC并发编程

### 1、线程池（重点）

> 重点 ： **三大方法、七大参数、七种拒绝策略**

   1. 程序的运行，本质：占用系统资源、优化资源的使用

   2. 池化技术: 事先准备一些资源、有人要用就来这里拿，拿完之后还给我

   3. 线程池的好处

      - 降低资源消耗、提高相应的速度
      - 方便管理

   4. 线程复用、可以控制最大并发数，管理线程

      - FixedThreadPool 和 SingleThreadPool : 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM
      - CachedThreadPool 和 ScheduledThreadPool : 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

   5. 三大方法

      ~~~java
      Executors.newSingleThreadExecutor();  //固定单个线程
      Executors.newFixedThreadPool(10);     //固定线程个数
      Executors.newCachedThreadPool();     //可伸缩的线程个数
      ~~~

   6. ThreadPoolExecutor 七大参数

      ~~~java
      new ThreadPoolExecutor(1, 1,
                             0L, TimeUnit.MILLISECONDS,
                             newLinkedBlockingQueue<Runnable>())
      new ThreadPoolExecutor(nThreads, nThreads,
                             0L, TimeUnit.MILLISECONDS, 
                             new LinkedBlockingQueue<Runnable>())
      new ThreadPoolExecutor(0, Integer.MAX_VALUE,                      //21亿   OOM
                             60L, TimeUnit.SECONDS,
                             new SynchronousQueue<Runnable>())
          
          //本质 ThreadPoolExecutor()
            public ThreadPoolExecutor(int corePoolSize,                             //核心线程数大小
                                    int maximumPoolSize,                                  //最大核心线程大小       
                                          long keepAliveTime,                                     //超时没有人用就会释放
                                          TimeUnit unit,                                               //超时单位
                                          BlockingQueue<Runnable> workQueue,    //阻塞队列
                                          ThreadFactory threadFactory,                      //线程工厂，一般不动
                                          RejectedExecutionHandler handler) {          //拒绝策略
                    if (corePoolSize < 0 ||
                        maximumPoolSize <= 0 ||
                        maximumPoolSize < corePoolSize ||
                        keepAliveTime < 0)
                        throw new IllegalArgumentException();
                    if (workQueue == null || threadFactory == null || handler == null)
                        throw new NullPointerException();
                    this.acc = System.getSecurityManager() == null ?
                            null :
                            AccessController.getContext();
                    this.corePoolSize = corePoolSize;
                    this.maximumPoolSize = maximumPoolSize;
                    this.workQueue = workQueue;
                    this.keepAliveTime = unit.toNanos(keepAliveTime);
                    this.threadFactory = threadFactory;
                    this.handler = handler;
                }
          
      ~~~

      ~~~java
      【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这
        样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
        说明：Executors 返回的线程池对象的弊端如下：
        1） FixedThreadPool 和 SingleThreadPool：
        允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
        2） CachedThreadPool：
        允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。 
      ~~~

7. 手动创建一个线程池

   ~~~java
   public static void main(String[] args) {
   
           ExecutorService threadPool = new ThreadPoolExecutor(
                   2,
                   Runtime.getRuntime().availableProcessors(),
                   3,
                   TimeUnit.SECONDS,
                   new LinkedBlockingDeque<>(3),
                   Executors.defaultThreadFactory(),
                  new ThreadPoolExecutor.AbortPolicy());
           try {
               //最大承载   Deque + max
               for (int i = 1; i <= 9; i++) {
                   threadPool.execute(() -> {
                       System.out.println(Thread.currentThread().getName() + "  OK");
                   });
               }
           } catch (Exception e) {
               e.printStackTrace();
           } finally {
               threadPool.shutdown();
           }
       }
   ~~~

8. 四种拒绝策略

   ~~~java
         ExecutorService threadPool = new ThreadPoolExecutor(
                   2,
                   Runtime.getRuntime().availableProcessors(),
                   3,
                   TimeUnit.SECONDS,
                   new LinkedBlockingDeque<>(3),
   //                Executors.defaultThreadFactory(),
   //                new ThreadPoolExecutor.AbortPolicy());        //队列满了，不处理，直接抛出异常
   //                new ThreadPoolExecutor.CallerRunsPolicy());   //队列满了，哪里来的去哪里
   //                new ThreadPoolExecutor.DiscardPolicy());      //队列满了，会丢掉任务，不会抛出异常
                   new ThreadPoolExecutor.DiscardOldestPolicy());  //队列满了，尝试和最早的竞争，不会抛出异常
   ~~~

   > 小结和拓展

   了解：IO密集型、CPU密集型：（调优）



### 2、四大函数式接口（必须掌握）

传统的程序员：泛型、枚举、反射

新时代的程序员：Lambda表达式、函数式编程、链式编程、Stream流式计算

~~~java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

// 超级多  FunctionalInterface
// 简化编程模型，在新版本的框架底层大量应用
// foreach(消费者类型的函数式接口)
~~~

![image-20200806102955177](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806102955177.png)

**代码测试**：

> 函数式接口

~~~java
public class functionDemo {

    //Function，有一个输入参数、有一个输出参数
    //Function，可以用Lmmbda表达式简化

    public static void main(String[] args) {
        Function function = new Function<String,String>() {
            @Override
            public String apply(String o) {
                return o;
            }
        };
        Function<String,String> function1 = (str)->{return str;};
        System.out.println(function.apply("Cc  你好"));
    }

}
~~~

> 断定型接口

~~~java
    public static void main(String[] args) {
        //断定型接口：有一个输入参数，返回值只能是布尔值
        Predicate<String> predicate = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.isEmpty();
            }
            //判断字符串是否为空
        };

        Predicate<String> predicate1 = (str) -> {
            return str.isEmpty();
        };
        System.out.println(predicate.test(""));
    }
~~~

> 消费性接口

~~~java
    public static void main(String[] args) {
        // Consume 只有输入参数，没有返回值
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        Consumer<String> consumer1 = (str) -> {
            System.out.println(str);
        };
        consumer.accept("Cc  你好");

    }
~~~

> 供给型接口

~~~java
    public static void main(String[] args) {
        //断定型接口：没有参数、只有返回值 
        Supplier<Integer> supplier = new Supplier<Integer>() {
            @Override
            public Integer get() {
                return 1024;
            }
        };

        Supplier<Integer> supplier1 = () -> {
            return 1024;
        };
        System.out.println(supplier.get());
    }
~~~



### 3、Stream流式计算

> 什么是Stream流式计算

大数据：存储 + 计算

存储：集合、Mysql

计算：都应该交给流来操作

~~~java
    public static void main(String[] args) {
        //1. ID必须是偶数
        //2. 年龄大于23
        //3. 用户名转为大写字母
        //4. 用户名字母倒序
        //5. 只输出一个用户
        User user1 = new User(1,"a",21);
        User user2 = new User(2,"b",22);
        User user3 = new User(3,"c",23);
        User user4 = new User(4,"d",24);
        User user5 = new User(5,"e",25);
        User user6 = new User(6,"f",26);
        List<User> users = Arrays.asList(user1, user2, user3, user4, user5, user6);
        System.out.println();

        //
        users.stream()
                .filter((user)->{return user.getId() % 2 ==0;})
                .filter((user -> {return user.getAge() > 23;}))
                .map(user -> {return user.getName().toUpperCase();})
                .sorted((o1, o2) -> {return o1.compareTo(o2);})
                .limit(1)
                .forEach(System.out::println);
    }
~~~



### 4、ForkJoin

ForkJoin在JDK1.7，并行执行任务！提高效率。大数据量

大数据：Map Reduce（把大任务拆分为小任务）

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806130324316.png" alt="image-20200806130324316" style="zoom: 67%;" />

> ForkJoin 特点：工作窃取

这个里边维护的都是双端队列

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806130556341.png" alt="image-20200806130556341" style="zoom: 67%;" />

> ForkJoin 操作：

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806142041039.png" alt="image-20200806142041039"  />



<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806142205448.png" alt="image-20200806142205448" style="float: left;" />

~~~java
public class ForkJoinDEmo extends RecursiveTask {
    //求和计算任务
    //如何使用forkJoin
    //1.通过forkJoin
    //如何使用forkJoin
    private Long start;
    private Long end;

    private Long temp = 10000L;

    public ForkJoinDEmo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    //计算方法
    @Override
    protected Long compute() {
        if ((end - start) < temp) {
            Long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {
            long middle = (start + end) / 2;
            ForkJoinDEmo task1 = new ForkJoinDEmo(start, middle);
            task1.fork();  //拆分任务、任务压入线程队列
            ForkJoinDEmo task2 = new ForkJoinDEmo(middle + 1, end);
            task2.fork();
            return (Long) task1.join() + (Long) task2.join();

        }
    }
}

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();               //5310
        test2();               //4184
        test3();               //146
    }


    public static void test1() {
        long start = System.currentTimeMillis();
        Long sum = 0L;
        for (Long i = 1L; i <= 10_0000_0000; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));
    }

    //会使用 FoekJoin
    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDEmo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);//提交任务
        Long sum = submit.get();

        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));
    }

    public static void test3() {
        long start = System.currentTimeMillis();
        //Stream并行流
        long sum = LongStream.rangeClosed(0, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));

    }
}
~~~



### 5、异步回调

> Future 设计的初衷 ： 可以对将来执行的某个事件的结果来建模

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806145217515.png" alt="image-20200806145217515" style="zoom: 50%;" />

> 实现类

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806150241186.png" alt="image-20200806150241186" style="zoom: 80%;" />

~~~java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        //没有返回值的异步回调
//        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
//            try {
//                TimeUnit.SECONDS.sleep(5);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName()+"runAsync=>Void");
//        });
//
//        System.out.println("============================>");
//        completableFuture.get();

        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "runAsync=>Integer");
            int i = 10/0;
            return 1024;
        });

        System.out.println(completableFuture.whenComplete((o1, o2) -> {
            System.out.println("o1=>" + o1);      //正常的返回结果
            System.out.println("o2=>" + o2);      //错误信息
        }).exceptionally((e) -> {
            e.printStackTrace();
            return 233;
        }).get());
    }
~~~



### 6、JMM

> 请你谈谈对Volatile的理解

Volatile 是 java 虚拟机提供的**轻量同步机制**

1. 保证可见性
2. ==不保证原子性==
3. 禁止指令重排

> 什么是 JMM

JMM Java内存模型，不存在的东西，概念，约定



关于JMM一些同步的约定

1. 线程解锁前，必须把共享变量==**立刻**==刷回主内存
2. 线程加锁钱，必须读取内存中的最新值到工作内存中
3. 加锁和解锁是同一把锁

线程：工作内存、主内存

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806153804403.png" alt="image-20200806153804403" style="zoom:80%;" />

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806154040982.png" alt="image-20200806154040982" style="zoom:80%;" />

8种操作

- **lock(锁定)**:作用于主内存的变量，一个变量在同一时间只能一个线程锁定，该操作表示这条线成独占这个变量
- **unlock(解锁)**:作用于主内存的变量，表示这个变量的状态由处于锁定状态被释放，这样其他线程才能对该变量进行锁定
- **read(读取)**:作用于主内存变量，表示把一个主内存变量的值传输到线程的工作内存，以便随后的load操作使用
- **load(载入)**:作用于线程的工作内存的变量，表示把read操作从主内存中读取的变量的值放到工作内存的变量副本中(副本是相对于主内存的变量而言的)
- **use(使用)**:作用于线程的工作内存中的变量，表示把工作内存中的一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时就会执行该操作
- **assign(赋值)**:作用于线程的工作内存的变量，表示把执行引擎返回的结果赋值给工作内存中的变量，每当虚拟机遇到一个给变量赋值的字节码指令时就会执行该操作
- **store(存储)**:作用于线程的工作内存中的变量，把工作内存中的一个变量的值传递给主内存，以便随后的write操作使用
- **write(写入)**:作用于主内存的变量，把store操作从工作内存中得到的变量的值放入主内存的变量中

**如果要把一个变量从主内存传输到工作内存，那就要顺序的执行read和load操作，如果要把一个变量从工作内存回写到主内存，就要顺序的执行store和write操作。对于普通变量，虚拟机只是要求顺序的执行，并没有要求连续的执行，所以如下也是正确的。对于两个线程，分别从主内存中读取变量a和b的值，并不一样要read a; load a; read b; load b; 也会出现如下执行顺序：read a; read b; load b; load a; (对于volatile修饰的变量会有一些其他规则,后边会详细列出)，对于这8中操作，虚拟机也规定了一系列规则，在执行这8中操作的时候必须遵循如下的规则：**

- **不允许read和load、store和write操作之一单独出现**，也就是不允许从主内存读取了变量的值但是工作内存不接收的情况，或者不允许从工作内存将变量的值回写到主内存但是主内存不接收的情况
- **不允许一个线程丢弃最近的assign操作**，也就是不允许线程在自己的工作线程中修改了变量的值却不同步/回写到主内存
- **不允许一个线程回写没有修改的变量到主内存**，也就是如果线程工作内存中变量没有发生过任何assign操作，是不允许将该变量的值回写到主内存
- **变量只能在主内存中产生**，不允许在工作内存中直接使用一个未被初始化的变量，也就是没有执行load或者assign操作。也就是说在执行use、store之前必须对相同的变量执行了load、assign操作
- **一个变量在同一时刻只能被一个线程对其进行lock操作**，也就是说一个线程一旦对一个变量加锁后，在该线程没有释放掉锁之前，其他线程是不能对其加锁的，但是同一个线程对一个变量加锁后，可以继续加锁，同时在释放锁的时候释放锁次数必须和加锁次数相同。
- **对变量执行lock操作，就会清空工作空间该变量的值**，执行引擎使用这个变量之前，需要重新load或者assign操作初始化变量的值
- **不允许对没有lock的变量执行unlock操作**，如果一个变量没有被lock操作，那也不能对其执行unlock操作，当然一个线程也不能对被其他线程lock的变量执行unlock操作
- **对一个变量执行unlock之前，必须先把变量同步回主内存中**，也就是执行store和write操作

问题：程序不知道主内存的值被修改过了

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806155006906.png" alt="image-20200806155006906" style="zoom:80%;" />



### 7、Volatile

> 1、保证可见性

~~~java
//加   volatile 保证可见性
    private volatile static int sum = 0;

    public static void main(String[] args) {
        new Thread(() -> {
            while (sum == 0) {

            }
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        sum = 1;
        System.out.println(sum);
    }
~~~

> 2、不保证原子性

原子性：不可分割

==线程A在 执行任务 的时候，不能被打扰，不能被分割，要么同时成功，要么同时失败==

~~~java
    //不保证原子性
    private volatile static int sum = 0;

    public static void add() {
        sum++;
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 20; i++) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();  //暂停正在制定的线程对象   去执行其他的线程
        }

        System.out.println(Thread.currentThread().getName() + "  " + sum);
    }
~~~

**如果不加 lock 和 synchronized 、如何保证原子性**

![image-20200806161235696](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806161235696.png)

使用 原子类、解决原子性问题

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200806161503657.png" alt="image-20200806161503657" style="zoom: 67%;" /> 

> 原子类为什么这么高级

~~~java
    //不保证原子性
    //private volatile static int sum = 0;
    
    //使用原子类来保证原子性
    private volatile static AtomicInteger sum = new AtomicInteger(0);

    public static void add() {
        sum.getAndIncrement();   //AtomicInteger + 1 方法,CAS
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 20; i++) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();  //暂停正在制定的线程对象   去执行其他的线程
        }

        System.out.println(Thread.currentThread().getName() + "  " + sum);
    }
~~~

**这些类的底层都直接和操作系统挂钩！在内存中修改值！Unsafe类 是一个很特殊的存在！**

> 指令重排

什么是 指令重排：计算机并不是按照你写的那样去执行的。

**源代码 --》 编译器优化的重排 --》指令并行会重排 --》内存系统也会重排 --》执行**

处理器在进行指令重排的时候，考虑：数据之间的依赖性！

~~~java
int x = 1;
int y = 2;
x = x + 5;
y = x * x;
~~~

可能造成的结果  a b x v 这四个默认都是0

| 线程A | 线程B |
| ----- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

正常结果 x = 0; y = 0；但是可能由于指令重排

| 线程A | 线程B |
| ----- | ----- |
| b = 1 | a = 2 |
| x = a | y = b |

指令重排导致的结果 ：x = 2;y = 1

> 非计算机专业

内存屏障 CPU指令 作用：

volatile 可以避免指令重排

1. 保证特定的操作的执行顺序
2. 可以保证某些变量的内存可见性（利用这些特性volatile 实现了可见性）

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200807090622184.png" alt="image-20200807090622184" style="zoom:80%;" />

**volatile 可以保证可见性。不能保证原子性，由于内存屏障，可以保证指令重排的现象产生**

**内存屏障在单例模式中使用最多**



### 8、彻底玩转单例模式

> 饿汉式单例

~~~java
public class HungrySingletonPattern {
    private HungrySingletonPattern() {

    }

    private final static HungrySingletonPattern singletonPattern = new HungrySingletonPattern();

    private static HungrySingletonPattern getInstance() {
        return singletonPattern;
    }
}
~~~

> DCL 懒汉式单例

~~~java
public class LazySingletonPattern {

    private static boolean flag = false;

    private LazySingletonPattern() {
//        synchronized (LazySingletonPattern.class) {
//            if (singletonPattern != null) {
//                throw new RuntimeException("不要试图使用反射破异常");
//            }
//        }
        synchronized (LazySingletonPattern.class) {
            if (flag == false) {
                flag = true;
            } else {
                throw new RuntimeException("不要试图使用反射破异常");
            }
        }
    }

    private volatile static LazySingletonPattern singletonPattern;//防止指令重排

    private static LazySingletonPattern getInstance() {
        //双重检测锁的 懒汉式单例 DCL模式
        if (singletonPattern == null) {
            synchronized (LazySingletonPattern.class) {
                if (singletonPattern == null) {
                    singletonPattern = new LazySingletonPattern();  //不是原子性操作
                    //1、分配内存空间
                    //2、执行构造方法，初始化对象
                    //3、把这个对象指向这个空间
                }
            }
        }
        return singletonPattern;  //此时singletonPattern没有完成构造
    }

    public static void main(String[] args) throws Exception {
//        LazySingletonPattern singletonPattern1 = LazySingletonPattern.getInstance();

        Field flag = LazySingletonPattern.class.getDeclaredField("flag");
        flag.setAccessible(true);

        Constructor<LazySingletonPattern> declaredConstructor = LazySingletonPattern.class.getDeclaredConstructor();
        declaredConstructor.setAccessible(true);
        LazySingletonPattern singletonPattern2 = declaredConstructor.newInstance();
        flag.set(singletonPattern2,false);
        LazySingletonPattern singletonPattern3 = declaredConstructor.newInstance();

        System.out.println(singletonPattern3 == singletonPattern2);
    }
}
~~~

> 静态内部类 

~~~java
public class InnerClass {
    private InnerClass() {

    }

    public static InnerClass () {
        return InnerClass.getInstance();
    }

    public static class Instance {
        private static final InnerClass innerClass = new InnerClass();
    }
}
~~~

> 单例不安全、因为有反射

> 枚举（两个参数的有参构造）

~~~java
public enum EnumDemo {
    INSTANCE;

    public EnumDemo getInstance() {
        return INSTANCE;
    }
}
~~~

> 枚举类型的最终反编译结果

~~~java
public final class EnumDemo extends Enum
{

    public static EnumDemo[] values()
    {
        return (EnumDemo[])$VALUES.clone();
    }

    public static EnumDemo valueOf(String name)
    {
        return (EnumDemo)Enum.valueOf(hs/demo/singletonPattern/enumDemo/EnumDemo, name);
    }

    private EnumDemo(String s, int i)
    {
        super(s, i);
    }

    public EnumDemo getInstance()
    {
        return INSTANCE;
    }

    public static final EnumDemo INSTANCE;
    private static final EnumDemo $VALUES[];

    static
    {
        INSTANCE = new EnumDemo("INSTANCE", 0);
        $VALUES = (new EnumDemo[] {
            INSTANCE
        });
    }
}
~~~



### 9、深入理解CAS

> 什么是 CAS

~~~java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);
        //public final boolean compareAndSet(int expect, int update)
        //如果期望值达到了就更新,否则就不更新 CAS并发原语
        atomicInteger.compareAndSet(2020,2021); //比较并交换
        System.out.println(atomicInteger.get());
        atomicInteger.getAndIncrement();

        atomicInteger.compareAndSet(2020,2021); //比较并交换
        System.out.println(atomicInteger.get());
    }
}
~~~

> Unsafe 类

~~~java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    // java 无法操作内存
    // java 可以调用c++ native
    // c++ 可以操作内存
    // java 的后门 、可以通过这个类操作内存
~~~

~~~java
public final int getAndIncrement() {
        return unsafe.getAndAddInt(this,  // 当前对象
                                   valueOffset,        // 内存地址偏移值
                                   1);
    }

public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            // 获取内存地址中的值
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
          // 自旋锁
          // 如果当前对象的值 和内存中取出来的值一样  就执行加一操作
          // 内存操作、效率很高
        return var5;
    }
~~~

**CAS ：比较当前工作内存中的值，如果这个值是期望的，就执行操作！不是就一直循环**

**缺点 ：**

1.  循环会耗时
2.  一次性只能保持一个共享变量的原子性
3.  存在 ABA 问题

> CAS ：ABA 问题（狸猫换太子）

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200807131140460.png" alt="image-20200807131140460"  />

~~~java
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        // 对于平时写的sql：乐观锁！
        //public final boolean compareAndSet(int expect, int update)
        //如果期望值达到了就更新,否则就不更新 CAS并发原语
        
        atomicInteger.compareAndSet(2020,2021); //比较并交换
        System.out.println(atomicInteger.get());

        atomicInteger.compareAndSet(2021,2020); //比较并交换
        System.out.println(atomicInteger.get());

        atomicInteger.compareAndSet(2020,6666); //比较并交换
        System.out.println(atomicInteger.get());
    }
~~~



### 10、原子引用

> 解决 ABA 问题、引入原子引用！对应的思想：乐观锁

带版本号的原子操作

注意：

~~~java
【强制】所有整型包装类对象之间值的比较，全部使用 equals 方法比较。
说明：对于 Integer var = ? 在-128 至 127 之间的赋值，Integer 对象是在 IntegerCache.cache 产生，
会复用已有对象，这个区间内的 Integer 值可以直接使用==进行判断，但是这个区间之外的所有数据，都
会在堆上产生，并不会复用已有对象，这是一个大坑，推荐使用 equals 方法进行判断。
~~~

~~~java
    public static void main(String[] args) {

        // AtomicStampedReference  注意 如果泛型是包装类   注意对象的引用问题
        AtomicStampedReference<Integer> reference = new AtomicStampedReference<>(1, 1);

        new Thread(() -> {
            System.out.println("A1=====" + reference.getStamp());

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(reference.compareAndSet(1, 2,
                    reference.getStamp(), reference.getStamp() + 1));
            System.out.println("A2=====" + reference.getStamp());

            System.out.println(reference.compareAndSet(2, 1,
                    reference.getStamp(), reference.getStamp() + 1));
            System.out.println("A3=====" + reference.getStamp());

        }, "A").start();

        // 乐观锁的原理相同

        new Thread(() -> {
            System.out.println("B1=====" + reference.getStamp());

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(reference.compareAndSet(1, 6,
                    reference.getStamp(), reference.getStamp() + 1));
            System.out.println("B2=====" + reference.getStamp());
        }, "B").start();
    }
~~~



### 11、各种锁的理解

#### 1、公平锁、非公平锁

- 公平锁：非常公平，不能够插队，必须先来后到

- 非公平锁：非常不公平、可以插队（默认都是非公平）

  ~~~java
  public ReentrantLock() {
          sync = new NonfairSync();
      }
      
  public ReentrantLock(boolean fair) {
          sync = fair ? new FairSync() : new NonfairSync();
      }    
  ~~~

  ![image-20200807141234953](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200807141234953.png)

#### 2、可重入锁

可重入锁（递归锁） 

> synchronized 版本

~~~java
public class Demo1 {
    public static void main(String[] args) {
        Phone phone = new Phone1();

        new Thread(()->{phone.sms();},"A").start();
        new Thread(()->{phone.sms();},"B").start();
    }
}

class Phone{
    public synchronized void sms() {
        System.out.println(Thread.currentThread().getName()+"发短信");
        call();
    }

    public synchronized void call() {
        System.out.println(Thread.currentThread().getName()+"打电话");
    }
}
}
~~~

> lock 版本

~~~java
public class Demo1 {
    public static void main(String[] args) {
        Phone1 phone = new Phone1();

        new Thread(()->{phone.sms();},"A").start();
        new Thread(()->{phone.sms();},"B").start();
    }
}

class Phone1{
    Lock lock = new ReentrantLock();

    public void sms() {
        lock.lock();  //细节问题：锁必须配对，否则就会死在里边
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"发短信");
            call();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }

    public void call() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"打电话");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
~~~

#### 3、自旋锁

~~~java
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            // 获取内存地址中的值
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
          // 自旋锁
          // 如果当前对象的值 和内存中取出来的值一样  就执行加一操作
          // 内存操作、效率很高
        return var5;
    }
~~~

~~~java
public class spinLockDemo {

    // int 0
    // Thread null
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    // 加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println((thread + "=========> mylock"));

        while (!atomicReference.compareAndSet(null,thread)) {

        }
    }
    // 解锁
    public void myUnLock(){
        Thread thread = Thread.currentThread();

        System.out.println((thread + "=========> mylock"));
        atomicReference.compareAndSet(thread,null);
    }
}
~~~

> 测试 

~~~java
public static void main(String[] args) throws Exception {

        SpinLockDemo lock = new SpinLockDemo();

        new Thread(()->{

            lock.myLock();

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }

        }).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            lock.myLock();

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }
        }).start();
    }
~~~

#### 4、死锁

> 死锁是什么

![image-20200807152347717](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200807152347717.png)

死锁测试、怎么排除死锁！

~~~java
public class MyThread implements Runnable {

    private String lockA;
    private String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println((Thread.currentThread().getName() + "--->lock:" + lockA + "--->get" + lockB));
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB) {
                System.out.println((Thread.currentThread().getName() + "--->lock:" + lockB + "--->get" + lockA));
            }
        }
    }
}
~~~

> 解决问题

1. 使用 ==jps -l== 定位进程号  

   ~~~java
   H:\sourceCode\JUC>jps -l
   12404 org.jetbrains.jps.cmdline.Launcher
   16472
   16584 hs.demo.lock.Test  // 对应目前的线程
   12140 sun.tools.jps.Jps
   15436 org.jetbrains.idea.maven.server.RemoteMavenServer36
   16780 org.jetbrains.kotlin.daemon.KotlinCompileDaemon
   
   ~~~

2. 使用 ==jstack 进程号== 找到死锁问题

   ~~~java
   Found one Java-level deadlock:
   =============================
   "T2":
     waiting to lock monitor 0x000000001d4df9f8 (object 0x000000076b3171e8, a java.lang.String),
     which is held by "T1"
   "T1":
     waiting to lock monitor 0x000000001d4e21d8 (object 0x000000076b317220, a java.lang.String),
     which is held by "T2"
   
   Java stack information for the threads listed above:
   ===================================================
   "T2":
           at hs.demo.lock.MyThread.run(MyThread.java:25)
           - waiting to lock <0x000000076b3171e8> (a java.lang.String)
           - locked <0x000000076b317220> (a java.lang.String)
           at java.lang.Thread.run(Thread.java:748)
   "T1":
           at hs.demo.lock.MyThread.run(MyThread.java:25)
           - waiting to lock <0x000000076b317220> (a java.lang.String)
           - locked <0x000000076b3171e8> (a java.lang.String)
           at java.lang.Thread.run(Thread.java:748)
   
   Found 1 deadlock.
   
   ~~~

   面试、工作中排查问题：

   1、日志

   2、堆栈信息



























