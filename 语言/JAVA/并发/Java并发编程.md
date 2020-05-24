## 一、java锁

### 1、java中的每一个对象都可以作为锁。（synchronize用的锁也是存在对象头里的）

- 对于普通方法，锁是当前实例对象
- 对于静态方法，锁是当前类的Class对象
- 对于同步代码块，锁是synchronized括号里配置的对象

### 2、synchronized与ReentrantLock对比

### 3、锁的种类

- 偏向锁
- 轻量锁
- 重量锁
- 自旋锁
  - 自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的。因此操作系统的实现在很多地方往往用自旋锁。Windows操作系统提供的轻型读写锁（SRW Lock）内部就用了自旋锁。显然，单核CPU不适于使用自旋锁，这里的单核CPU指的是单核单线程的CPU，因为，在同一时间只有一个线程是处在运行状态，假设运行线程A发现无法获取锁，只能等待解锁，但因为A自身不挂起，所以那个持有锁的线程B没有办法进入运行状态，只能等到操作系统分给A的时间片用完，才能有机会被调度。这种情况下使用自旋锁的代价很高。获取、释放自旋锁，实际上是读写自旋锁的存储内存或寄存器。因此这种读写操作必须是原子的。通常用test-and-set等原子操作来实现。[1]

### 4、CAS实现原子操作（更新失败时可以配置重试或者直接抛出异常）

- ABA问题：CAS操作是先比较旧值是否相等，相等再更新值为新值。假设一个值先是A，然后被更新为了B，又被更新成了A。进行比较时会认为没有变，但是其实变了。解决ABA问题可以使用AtomicStampedReference。
- 循环时间长开销大（？）
- 只能保证一个共享变量的原子操作（？）
## 二、经典问题

### 4.1读写者问题（公平竞争）

```java
package com.multiThread;

import java.util.Comparator;
import java.util.concurrent.Semaphore;
import java.util.concurrent.ThreadPoolExecutor;
import org.apache.poi.ss.formula.functions.T;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author : yanqing.pj  2019/10/5 14:20
 */
public class ReaderAndWriter {
    private static Logger logger = LoggerFactory.getLogger(ReaderAndWriter.class);

    static int readerCount = 0;
    /** 读写者、写写者互斥 */
    static Semaphore rw = new Semaphore(1,true);
    /** 控制readerCount 的并发修改 */
    static Semaphore readerCountSemaphore = new Semaphore(1,true);

    /** 实现读写者公平 */
    static Semaphore fair = new Semaphore(1,true);

    public static void main(String[] args) {
        ThreadExecutor.COMMON_EXECUTOR.execute(new Reader());
        ThreadExecutor.COMMON_EXECUTOR.execute(new Reader());
        ThreadExecutor.COMMON_EXECUTOR.execute(new Reader());

        ThreadExecutor.COMMON_EXECUTOR.execute(new Writer());
        ThreadExecutor.COMMON_EXECUTOR.execute(new Writer());


    }

    static class Writer implements Runnable{
        @Override
        public void run() {
            while (true) {
                try {
                    fair.acquire();
                    rw.acquire();
                    String name = Thread.currentThread().getName();
                    System.out.println(name + "begin-write");
                    Thread.sleep(2000);
                    System.out.println(name + "end-write");
                    rw.release();
                    fair.release();
                    Thread.sleep(9000);
                } catch (Exception e) {
                    logger.error("{} : {}", Thread.currentThread().getName(), e.getMessage());
                }
            }
        }
    }


    static class Reader implements Runnable{
        @Override
        public void run() {
            while (true) {
                try {
                    fair.acquire();
                    readerCountSemaphore.acquire();
                    if (readerCount == 0) {
                        rw.acquire();
                    }
                    readerCount++;
                    readerCountSemaphore.release();
                    fair.release();
                    String name = Thread.currentThread().getName();
                    System.out.println(name + "begin-read");
                    Thread.sleep(1000);
                    System.out.println(name + "end-read");
                    readerCountSemaphore.acquire();
                    readerCount--;
                    if (readerCount == 0) {
                        rw.release();
                    }
                    readerCountSemaphore.release();
                } catch (Exception e) {
                    logger.error("{} : {}", Thread.currentThread().getName(), e.getMessage());
                }
            }
        }
    }
}

```

## 三、volatile

保证变量的可见性和禁止指令重排

被volatile修饰的变量，会多出一个Lock前缀的指令，导致发生以下两件事：

1、将当前处理器缓存行的数据写回内存系统；

2、当1发生时，会使其他CPU中的缓存数据无效（每个处理器通过嗅探在总线上传播的数据进行检查实现）