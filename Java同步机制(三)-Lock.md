# Java同步机制(三)-Lock

在除了Synchronized关键字之外，Java还为我们提供了Lock接口，供我们实现一些更加灵活同步机制。下面我们就来分析一下。

```java

public interface Lock {
    
    //能够保证获得锁
    //如果锁再当前不是空闲状态，则会挂起当前线程
    void lock();
    
    //保证获得锁
    //但是在获取锁的过程中，能够响应线程中断，并且抛出中断异常
    void lockInterruptibly() throws InterruptedException;

    //尝试获取锁，不能保证获取成功
    //如果获取失败则返回false
    //如果获取成功则返回true
    boolean tryLock();
    
    //在一段时间内尝试获取锁
    //在这段时间内如果锁不是空闲状态则会挂起当前线程
    //在获取锁的过程中可以相应中断，抛出终端异常
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    
    void unlock();
    
    Condition newCondition();
}

```

## Synchronized和Lock的区别

### Lock更加灵活
我们都说Lock比Synchronized更灵活。但是具体怎么灵活呢，通过上面Lock接口定义的几个方法，我理解主要在下面几个方面。

1. Lock提供了尝试获取方法tryAcquire()
    - Lock提供了尝试获取方法，既拿不到锁立即返回
    - 相对应的Synchronized则没有，一旦使用，则必须获取成功才能继续往下执行

2. Lock提供了可响应中断的获取锁方法
    - Lock提供了可中断的获取锁方法，`lockInterruptibly()`和`tryLock(long time,TimeUnit unit)`。既线程在获取Lock的挂起过程中是可以响应中断的，开发者可以通过这种方法来终止获取锁。
    - Synchronized则不可以，线程在Synchronized的挂起过程中，不可响应中断操作，一旦开始，就无法停止。

3. Condition机制

### 其他特点
1. 性能
    - 我自己理解，性能方面已经不是Lock和Synchronized的主要区别了，我们知道Lock主要是依赖AQS来实现的，但是熟悉Synchronized应该也知道，Synchronized优化后，性能强了不少。并且Synchronized的重量级锁实现和AQS有很多类似的地方。
    
2. 方便性
    - 在使用便利性上Synchronized还有有一些优势的，因为Synchronized可以省掉一步手动释放锁的操作。JVM帮我们完成了释放的操作。
    - 另一点，除了开发上的便利性，在调试上Synchronized也是有一定优势的，因为毕竟是JVM原生提供的，所以在使用JVM的命令,比如jstack调试的时候，能够方便的看到Synchronized相关的信息。而Lock就不可以了。

3. 



