# 1、Java语言的学习

​	Java基础->流程控制->Java集合->JavaIO流->异常->多线程->网络编程->反射

···>

>- 多线程
>- 网络编程
>- 反射机制

## 1.1线程

### 	1.1.1构建

#### 		1.1.1.1用 start 而不用 run() 执行线程的原因：

>- ​	run();是直接调用普通方法执行，此时相当于还是单线程执行；
>- ​	start 是 通知 CPU 以线程的方式执行，最后调用run()方法；

#### 		1.1.1.2 子线程要放在最开始（主线程的前面）

#### 		1.1.1.3如何实现多线程的：

>- 继承Thread对象
>- 重写run方法
>- 创建线程对象
>- 调用start()方法

#### 		1.1.1.4线程的匿名写法：

```java
 new Thread(
     () -> {
        for (int i = 0; i < 10; i++) {
            System.out.println("子线程执行输出" + i);
            }
        }
).start();
```

#### 		1.1.1.5Callable+FutureTask的写法：

>- 先构建一个Callable的继承类MyCallabel
>
>```java
>
>class MyCallable implements Callable<String> {
>    private int n;
>
>    public MyCallable(int n) {
>        this.n = n;
>    }
>
>    //重写call方法(任务方法)(求n的加和)
>    @Override
>    public String call() throws Exception {
>        int sum = 0;
>        for (int i = 0; i <= n; i++) {
>            sum += i;
>        }
>        return "子线程执行的结果是：" + sum;
>    }
>}
>```
>
>
>
>- 交给FutureTask
>
>```java 
>        FutureTask<String> f = new FutureTask<>(new MyCallable(100));
>```
>
>
>
>- 执行线程
>
>```java 
>	//执行线程
> new Thread(f).start();
>
>	//得到结果
>	//如果 f 任务没有执行完成，这里的代码会等待，知道 f 执行完采取；
>String result = f.get();
>
>	//输出结果
>System.out.println(result);
>```

#### 		1.1.1.6方式对比

| 方式                        |         优点         |          缺点          |
| :-------------------------- | :------------------: | :--------------------: |
| 继承Thread类                |     编程比较简单     |        扩展性差        |
| 实现Runnable接口            |       扩展性强       | 相对复杂，没有返回结果 |
| 实现Callable+FutureTask接口 | 扩展性强，可得到结果 |        编程复杂        |



### 	1.1.2线程安全问题

>多个线程同时共享同一个资源并修改该资源

#### 		1.1.2.1解决方法 ->线程同步

 线程同步的思想：

>加锁：让多个线程依次访问同一个资源

```java
synchronized (someObject){ //someObject 为 锁对象 -> 唯一
  //此处的代码只有占有了someObject后才可以执行
  //行为
}
```

>锁对象为 **任意的** 唯一对象到底好不好呢：
>
>- 不好，**影响其线程的进行**
>
>规范上：
>
>- 建议使用**共享资源**作为锁对象：
>  - 对于实例方法建议使用**this**作为锁对象
>  - 对于静态方法建议使用**字节码（类名：class）**对象作为锁对象
>
>
>```java
>//100个线程人过路
>   public static void run() {
>       synchronized (Account.class) {
>           //执行代码
>       }
>   }
>```



#### 		1.1.2.2同步方法

>直接在返回值类型(int void long···)前面 +  **synchronized**就可以了
>
>```java
>//取钱
>    public synchronized void drawMoney(double money) {
>        //谁来取钱
>        String name = Thread.currentThread().getName();
>
>        //同步代码块
>        synchronized (this) {         //this唯一即可
>            if (this.money >= money) {//钱够取
>                System.out.println(name + "取钱成功，取出" + money);
>                this.money -= money;//扣除
>                System.out.println(name + "取钱后剩余" + this.money);
>            } else {
>                System.out.println(name + "来取钱，没钱取");
>            }
>        }
>    }
>```
>
>如果一个类，其**方法都是有synchronized修饰的**，那么该类就叫做**线程安全的类**
>
>同一时间，只有一个线程能够进入 **这种类的一个实例** 的去修改数据，进而保证了这个实例中的数据的安全(不会同时被多线程修改而变成脏数据)

### 	1.1.3线程的通信*

>通过`flag`参数维持两个线程之间的通信
>
>```java
>public class Phone {
>//实现线程之间的通信，默认认为手机当前处于等待来电提醒
>private boolean flag = false;
>
>public void run() {
>   //来电线程
>   new Thread(new Runnable() {
>       @Override
>       public void run() {
>           try {
>               while (true) {
>                   synchronized (Phone.this) {
>                       if (!flag) {
>                           //代表要来电话提醒了
>                           System.out.println("您好，有新电话，请接听~");
>                           flag = true;
>                           Phone.this.notify();
>                           Phone.this.wait();
>                       }
>                   }
>               }
>           } catch (Exception e) {
>               e.printStackTrace();
>           }
>       }
>   }).start();
>
>
>   //接听线程
>   new Thread(new Runnable() {
>       @Override
>       public void run() {
>           try {
>               //不断的接听电话
>               while (true) {
>                   synchronized (Phone.this) {
>                       if (flag) {
>                           //可以接听电话
>                           System.out.println("电话接听中，通信5min结束~~~");
>                           Thread.sleep(2000);
>                           flag = false;//等待呼入电话
>                           //等待自己，唤醒其他线程
>                           Phone.this.notify();//先唤醒
>                           Phone.this.wait();//再等待
>                       } else {
>                           Phone.this.notify();//唤醒
>                           Phone.this.wait();//等待
>                       }
>                   }
>               }
>           } catch (Exception e) {
>               e.printStackTrace();
>           }
>       }
>   }).start();
>}
>
>public static void main(String[] args) {
>   Phone huawei = new Phone();
>   huawei.run(); //开机
>}
>}
>```





### 	1.1.4线程池【重要】

>- 线程池里固定依一些线程，并重复利用这些线程
>- 有利于系统进程，防止资源的耗尽



​	**如何得到线程池对象：**

>- 使用**ExecutorService（接口）**的实现类：**TreadPollExecutor** 自创建一个线程池对象
>
>```java
> ExecutorService pools = new ThreadPoolExecutor(
>                3,//核心线程个数 corePoolSize
>                5,//最大线程量 maximumPoolSize
>                8,//最大时间 keepAliveTime
>                TimeUnit.SECONDS,//
>                new ArrayBlockingQueue<>(6),//任务队列
>                Executors.defaultThreadFactory(),//线程工厂
>                new ThreadPoolExecutor.AbortPolicy());//任务拒绝策略
>```
>
>- 使用 **Executor** （线程池对象的工具类）调用方法返回不同特点的线程池对象 

>**临时线程什么时候创建？**
>
>- 新任务提交时发现核心线程都在忙，任务队列也满了，而且还可以创建新临时线程，此时才会创建临时线程

>**什么时候后拒绝新的任务？**
>
>- 核心线程和临时线程都在忙，任务队列也满了，新的任务过来，此时才会拒绝新的任务



#### 		1.1.4.1处理Runnable任务

>**如何丢入线程池(如何处理Runnable任务)：**
>
>```java
> ExecutorService pool = new ThreadPoolExecutor(3,5,6,
>                TimeUnit.SECONDS,new ArrayBlockingQueue<>(5),
>                new ThreadPoolExecutor.AbortPolicy());
>
> Runnable target = new MyRunnable();
>
> pool.execute(target);// pool 的 execute 方法
>```

>**如何关闭线程池：**
>
>```java
>//(开发中一般不会使用)。
>pool.shutdownNow();//立刻关闭，即使任务没有完成，丢失任务的。
>pool.shutdowm();//会等待全部任务执行完毕之后再关闭
>```



#### 		1.1.4.2处理Callable任务

>调用**ExecutorService**的方法：
>
>```java
>Future<T> submit = pool.submit(new MyCallable(···));
>T ··· = submit.get();
>```
>
>返回 **未来** 对象;
>
>再返回**信息**;



#### 		1.1.4.3**Executors**工具类得到线程池

>```java
>ExecutorService pool = Executors.newFixedThreadPool(int nThreads);
>//创建固定线程数量的线程池，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程来替代它
>```
>
>- 大型并发系统环境中使用 **Executors** 如果不注意可能会出现系统风险

### 	1.1.5定时器(扩展)

#### 		1.1.5.1**Timer**计时器

>- **创建**
>
>```java
>//创建
>Timer timer = new Timer();  //定时器本身就是一个单线程
>
>//调用方法，处理定时任务
>timer.schedule(new TimerTask() {
>    @Override
>    public void run() {
>        System.out.println(Thread.currentThread().getName()+"执行一次~");
>    }
>},3000,2000);
>
>```

>- **其中存在的问题：**
>
>  1. Timer是单线程，处理多个任务按照顺序执行，存在延时与设置定时器的时间有出入
>
>  1. 可能因为其中的某个任务的异常使Timer线程死掉，从而影响后续任务执行

#### 		1.1.5.2**ScheduleExecutorService**计时器

>- 为了弥补**Timer**的缺陷，
>- **ScheduleExecutorService**内部为线程池

>**优点：**
>
>1、基于线程池，某个任务的执行情况不会影响其他定时任务的执行
>
>2、**实际开发中也用这种方式做线程**

>**代码：**
>
>```java
>//1、创建ScheduleExecutorService线程池，做定时器
>	ScheduledExecutorService pool = Executors.newScheduledThreadPool(3);
>//2、开启定时任务
>    pool.scheduleAtFixedRate(new TimerTask() {
>        @Override
>        public void run() {
>            System.out.println(Thread.currentThread().getName() + "执行了AAA"+new Date());
>            try {
>                Thread.sleep(10000);//睡 10s
>            } catch (Exception e) {
>                e.printStackTrace();
>            }
>        }
>    }, 0, 1, TimeUnit.SECONDS);
>
>    pool.scheduleAtFixedRate(new TimerTask() {
>        @Override
>        public void run() {
>            System.out.println(Thread.currentThread().getName() + "执行了BBB" + new Date());
>        }
>    }, 0, 1, TimeUnit.SECONDS);
>```



## 1.2网络编程

>**常见的通信模式：**
>
>- **Client-Server(CS)**模式
>- **Browser/Server(BS)**模式（重心）

>**CS模式：**
>
>- 客户端：
>
>  - 需要程序员开发
>
>  - 需要用户安装
>
>- 服务端
>
>  - 需要程序员开发实现

>**BS模式：**
>
>- 浏览器：
>  - 不需要程序员开发实现
>  - 用户需要安装浏览器
>- 服务端
>  - 需要程序员开发实现

### 	1.2.1三要素之-IP地址

 >**IP**(Internet Protocol)：全称“互联网协议地址”，是分配给上网设备的唯一标志

>常见的IP分类为：**IPV4**和**IPV6**

- IP地址的形式：
  - 公网地址、和私有地址（局域网使用）
  - 192.168. 开头的就是常见的局域网地址，范围即为192.168.0.0--192.168.255.255，专门为组织机构内部使用

- IP常用命令：
  - **ipconfig**：查看本机IP地址
  - **ping** IP地址：检查网络是否连通

- 特殊IP地址：
  - 本机IP：**127.0.0.1**或者**localhost**：称为回送地址也可称本地回环地址，只会寻找当前所在本机

```java
  		//1、获取本机地址对象
        InetAddress ip1 = InetAddress.getLocalHost();
        System.out.println(ip1.getHostName());
        System.out.println(ip1.getHostAddress());

        //2、获取域名ip对象
        InetAddress ip2 = InetAddress.getByName("www.baidu.com");
        System.out.println(ip2.getHostName());
        System.out.println(ip2.getHostAddress());

        //3、判断是否能通：ping 5s之前测试是否可通
        InetAddress ip3 = InetAddress.getByName("14.215.177.38");
        System.out.println(ip3.getHostName());
        System.out.println(ip3.getHostAddress());
        System.out.println(ip3.isReachable(5000));

```



### 	1.2.2三要素之-端口

- 端口号：**标识正在计算机设备上运行的进程(程序)，**被规定为一个16位的二进制，范围是0-65535**(在主机里是唯一的)**
- 端口号类型：
  - 周知端口：0-1023，被预先定义的知名应用占用(如：HTTP占用80，FTP占用21)
  - **注册端口(自己可以用的)：**1024-49151，分配给用户进程或某些应用程序，(如：Tomcat占用8080，MySQL占用3306)



### 	1.2.3三大要素之-协议

- 通讯协议：**连接和通信数据的规则**被称为网络通信协议

- 参考模型：
  - OSI参考模型：全球通信规范，过于理想，未被推广
  - TCP/IP参考模型：事实上的国际标准

>**传输层的2个常见协议：**
>
>- **TCP(传输控制协议)**
>  - 面向连接，可靠通信
>  - 三次握手
>  - 大量数据
>  - 需确认完毕
>- **UDP(用户数据包协议)**
>  -  无连接，不可靠
>  - 将数据源IP、目的地IP和端口封装成数据包
>  - 每个数据包大小限制在64kb以内
>  - 可以广播发送(群发)，无需释放资源、开销小、速度快















# 2、JavaWeb基础

​	HTML/CSS/JavaScript ->Tomcat->XML->Servlet->HTTP->Filter->AJAX/JSON

## 2.1 前端知识

>- HTML
>- CSS
>- JavaScript

##  2.2JavaWeb

>- Tomcat（简单过一下）
>- XML（简单过一下）
>- Servlet（重点理解）
>- HTTP协议（重点理解）
>- Filter过滤器（重点理解）
>- AJAX（简单过一下）

##  2.3学习数据库 [点这里下载MySql](https://downloads.mysql.com/archives/installer/)

##  2.4学习Java连接数据库（JDBC）

##  2.5项目管理和框架的学习

​	项目管理和框架->Maven->SpringBoot













