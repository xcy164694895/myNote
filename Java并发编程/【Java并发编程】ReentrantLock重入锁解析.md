### 概述
重入锁ReentrantLock,顾名思义，就是支持重进入的锁，它表示能够支持一个线程对资源的重复加锁。除此之外，该锁还支持获取锁时的公平和非公平选择。Synchronized关键字通过获取自增、释放递减的方式来隐式的支持重入，那么Reentrant是如何支持重入的呢？又是怎么实现公平和非公平选择的呢？接下来我们带着这些问题来看ReentrantLock的源码

### 重进入的实现原理
重进入是指任意线程在获取到锁之后，能够再次获取该锁，而不会因为再次获取该锁被阻塞，该特性的实现主要需要解决的是一下两个问题：
1. **线程再次获取锁。锁需要识别来尝试获取锁的线程，是不是当前占有锁的线程，如果是，那么获取成功，如果不是，那么获取失败。**
2. **锁的最终释放。线程重复n次获取了锁，随后在n次释放该锁后，其他线程能够正常获取到锁。锁最终能够正常释放，要求锁的获取进行自增计数，计数表示该锁被重复获取的次数，而锁被释放时，计数自减，当计数归零时，表示该锁已经成功释放。**

ReentrantLock是通过组合自定义同步器来实现锁的获取和释放的，下面我们以默认的非公平锁源码来深入分析下实现原理，非公平锁获取锁的源码如下：
```
final boolean nonfairTryAcquire(int acquires) {
    //获取当前线程
    final Thread current = Thread.currentThread();
    //获取当前同步状态
    int c = getState();
    //如果同步状态等于0，说明该锁未被任何线程占用
    if (c == 0) {
        //CAS方法修改同步状态
        if (compareAndSetState(0, acquires)) {
            //将当前线程赋值给exclusiveOwnerThread
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //否则，判断占有该锁的线程是不是当前线程
    else if (current == getExclusiveOwnerThread()) {
        //再次获取，状态值加上acquires
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
相信大家很容易看懂这段代码，该方法增加了重入锁再次获取同步状态的逻辑：判断占有锁的线程是否为当前线程来决定获取操作是否成功，如果是获取锁的线程再次请求，那么将同步状态值增加并返回true，来表示获取同步状态成功。成功获取锁的线程再次获取锁，只是增加了同步状态值，这也就要求ReentrantLock在释放同步状态的时候，减少同步状态的值，其对应的tryRelease()方法源码如下：
```
protected final boolean tryRelease(int releases) {
    //同步状态递减
    int c = getState() - releases;
    //如果当前线程不是占有该锁的线程，那么抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    //如果递减后同步状态为0，那么将释放锁，返回true
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    //如果状态不为0，那么返回false
    setState(c);
    return free;
}

```

从代码中我们可以看出，只有同步状态为0的时候，锁才会被完全释放。如果该锁被获取了n次，那么前（n-1）次tryRelease(int releases)方法必须返回false，只有最后一次才会返回true。


### 公平锁与非公平锁
ReentrantLock支持两种锁：公平锁与非公平锁，分别对应实现为FairSync和NonfairSync。**公平性与否是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求的绝对时间顺序，也就是FIFO。** 回顾上一小节中非公平锁获取的nonfairTryAcquire(int acquires)方法，只要CAS自旋获取同步状态成功，则表示当前线程获取了锁，不会要求FIFO，而公平锁则不然。ReentrantLock的构造方法为无参构造方法时，构造的就是非公平锁，源码为：
```
public ReentrantLock() {
        sync = new NonfairSync();
}

```
而ReentrantLock还有另外一种构造方法，有参构造方法：
```
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

```
当传入参数为true时，创建公平锁；否则创建非公平锁。下面我们来看下公平锁的处理逻辑是怎么样的，核心方法如下：
```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //如果当前同步状态为0
    if (c == 0) {
        //判断同步队列中当前节点前面是否还有其余节点，如果没有，那么尝试获取同步状态，成功则返回true
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //否则，判断占有锁的线程是否为当前线程，如果是，那么同步状态加1，返回true。
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    //否则返回false
    return false;
}

public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    //判断同步队列中，当前节点前面是否还有其余节点
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}

```
从上面的源码中我们看到，nonfailSync中的tryAcquire方法，多调用了hasQueuedPredecessors()方法来判断同步队列中，当前节点前是否还有其余节点。如果有，则说明有线程比当前线程更早请求资源，根据公平性要求，当前节点获取资源失败；如果当前节点没有前驱节点，才会做后续操作。

公平锁和非公平锁可以总结出一下几点：
1. **公平锁保证了锁的获取按照FIFO原则，每次获取到锁的都是同步队列的第一个节点，保证了请求资源时间上的绝对顺序；非公平锁，有可能刚刚释放锁的线程立马又获取到锁，而有的线程会一直获取不到锁，这也就可能造成“饥饿”现象。**
2. **公平锁为了保证请求资源上的绝对顺序，需要频繁的更换线程，切换上下文，而非公平锁则一定程度上降低了上下文的切换，降低了性能开销。所以ReentrantLock默认选择非公平锁，这样做是为了减少上下文切换，保证系统有更大的吞吐量。**


>注：本文参考《Java并发编程的艺术》