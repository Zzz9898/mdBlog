
>- ## Java之synchronized相关
>  
- #### synchronized使用
  - 修饰普通方法（对象锁）
  - 修饰静态方法（类锁）
  - 修饰代码块（也分类锁、对象锁）
- #### synchronization的对象锁和类锁
  - 类锁：`所有该类实例对象共用一把锁`
  - 对象锁: `一个对象一把锁，多个对象多把锁`
  - 小结
    - 有线程访问对象的同步代码块时，另外的线程可以访问该对象的非同步代码块
    - 若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问该对象的同步代码块的线程会被阻塞
    - 若锁住的是同一个对象，一个线程在访问对象的同步方法时，另一个访问该对象的同步方法的线程会被阻塞
    - 若锁住的时同一个对象，一个线程在访问对象的同步代码块时，另一个访问该对象的同步方法的线程会被阻塞，反之亦然
    - 同一个类的不同对象的对象锁互不干扰
    - 类锁由于也是一种特殊的对象锁，因此表现和上述一致，而由于一个类只有一把对象锁，所以用一个类的不同对象使用类锁将会是同步的
    - 类锁和对象锁互不干扰
- #### synchronization原理
  `synchronized进过编译，会在同步块的前后分别形成monitorenter和monitorexit这个两个字节码指令，在执行monitorenter指令时，首先要尝试获取对象锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了，如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。在修饰方法时，JVM通过该ACC_SYNCHRONIZED访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用`
  - 每个对象有一个监视器锁（monitor），当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权
  - Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高
  - Java 6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁
- #### ReentrantLock
  - ReentrantLock主要利用CAS+AQS队列来实现。
    - ReentrantLock 有三个内部类 Sync，NonfairSync，FairSync
      - Sync是一个抽象类，继承自抽象类 AbstractQueuedSynchronizer
      - ReentrantLock就是对Sync某个实现的包装，公平锁为FairSync非公平锁为NonfairSync，FairSync和NonfairSync都继承Sync
    - CAS：`Compare and Swap，比较并交换。CAS有3个操作数：内存值V、预期值A、要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。该操作是一个原子操作，被广泛的应用在Java的底层实现中。在Java中，CAS主要是由sun.misc.Unsafe这个类通过JNI调用CPU底层指令实现`
    - AQS：`AbstractQueuedSynchronizer简称AQS`
      - AQS使用一个FIFO队列表示排队等待锁的线程，队列头结点称作“哨兵节点”或者“哑结点”，它不与任何线程关联。其他的节点与等待线程关联，每个阶段维护一个等待状态waitStatus
      ![](http://wxf.zcoder.top/server/files/aqsmx.png)
    - ReentrantLock的流程
      - ReentrantLock先通过CAS尝试获取锁
      - 如果此时锁已经被占用，该线程加入AQS队列并wait()
      - 当前驱线程的锁被释放，挂在CLH队列为首的线程就会被notify()，然后继续CAS尝试获取锁
        - 非公平锁(默认)，如果有其他线程尝试lock()，有可能被其他刚好申请锁的线程抢占
        - 公平锁，只有在CLH队列头的线程才可以获取锁，新来的线程只能插入到队尾
- #### synchronized与ReentrantLock区别
  - synchronized 竞争锁时会一直等待  
    ReentrantLock 可以尝试获取锁，并得到获取结果
  - synchronized 获取锁无法设置超时  
    ReentrantLock 可以设置获取锁的超时时间
  - synchronized 无法实现公平锁  
    ReentrantLock 可以满足公平锁，即先等待先获取到锁
  - synchronized 控制等待和唤醒需要结合加锁对象的 wait() 和 notify()、notifyAll()  
    ReentrantLock 控制等待和唤醒需要结合 Condition 的 await() 和 signal()、signalAll() 方法
  - synchronized 是 JVM 层面实现的  
    ReentrantLock 是 JDK 代码层面实现
  - synchronized 在加锁代码块执行完或者出现异常，自动释放锁  
    ReentrantLock 不会自动释放锁，需要在 finally{} 代码块显示释放
- #### volatile：`修饰变量的改变对其他线程立即可见`
  - 当写线程写一个volatile变量时，JMM(Java内存模型)会把该线程对应的本地内存中的共享变量值刷新到主内存
  - 当读线程读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，线程接下来将从主内存读取共享变量
- #### synchronized与volatile区别
  - volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取  
    synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住
  - volatile仅能使用在变量级别  
    synchronized则可以使用在变量、方法、和类级别的
  - volatile仅能实现变量的修改可见性，不能保证原子性
    synchronized则可以保证变量的修改可见性和原子性
  - volatile不会造成线程的阻塞
    synchronized可能会造成线程的阻塞
- #### Happens-Before原则：  
  `（先行发生原则）是判断数据是否存在竞争、线程是否安全的主要依据`
  - 如果两个操作之间具有happens-before关系，那么前一个操作的结果就会对后面的一个操作可见，且第一个操作的执行顺序在第二个操作之前。
  - 两个操作之间存在happens-before关系，并不意味着必须按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，这种重排序是合法的
  - 常见的happen-before规则
    - <font size=2>程序顺序规则：一个线程中的每个操作，happen-before在该线程中的任意后续操作。(注解：如果只有一个线程的操作，那么前一个操作的结果肯定会对后续的操作可见。)</font>
    - <font size=2>锁规则：对一个锁的解锁，happen-before在随后对这个锁的加锁。(注解：这个最常见的就是synchronized方法和syncronized块)</font>
    - <font size=2>volatile变量规则：对一个volatile域的写，happen-before在任意后续对这个volatile域的读。该规则在CurrentHashMap的读操作中不需要加锁有很好的体现。</font>
    - <font size=2>传递性：如果A happen-before B，且B happen-before C，那么A happen - before C。</font>
    - <font size=2>线程启动规则：Thread对象的start()方法happen-before此线程的每一个动作。</font>
    - <font size=2>线程终止规则：线程的所有操作都happen-before对此线程的终止检测，可以通过Thread.join()方法结束，Thread.isAlive()的返回值等手段检测到线程已经终止执行。</font>
    - <font size=2>线程中断规则：对线程interrupt()方法的调用happen-before发生于被中断线程的代码检测到中断时事件的发生。</font>