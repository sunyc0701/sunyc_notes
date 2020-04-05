##  LOCK接口

今天看的了公平锁和非公平锁，深入了解学习一下

###  定义 

公平锁: 多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁 
优点: 所有的线程都能得到资源，不会饿死在队列中。
缺点: 吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu幻想阻塞线程的开销会很大。

非公平锁:加锁时不考虑吧排队等待问题，直接尝试获取锁，获取不到自动到对位等待。
优点:减少cpu唤醒线程的开销，整体的吞吐效率会高点，CPU也不惜幻想所有线程，减少唤起现成的数量。
切点: 存在线程饿死。

### 底层:

![](/img/java/ReentrantLock.png)

通过代码可以发现 Sync类是ReentrantLock本身的一个内部类，他继承了AbstractQueueSynchroinzer(AQS),我们在操作锁的大部分操作，都是Sync本身去实现的。

Sync类有两个子类:FairSync和NofairSync。 需要说一下的是 ReentrantLock 默认非公平锁，如果想要成为公平锁 如下

![](/img/java/层次.png)

````java
ReentrantLock lock=new ReentrantLock(true);
// 构造方法 
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
````

看了下代码，自我理解是

非公平锁:当一个线程去操作这个资源时，先获取锁，判断状态state是否为0，如果为0，可以CAS成功，并修改当前的线程为自己，这时线程B也来，发现状态是1，只能去队列等待，等着唤醒。当A处理完后，准备释放锁，修改state状态，抹掉了持有锁线程的痕迹，准备叫醒B。这时 线程C来了，发现state为0，果断CAS为1，并将持有锁的线程改为自己。B线程唤醒后去获取锁，发现state为1，只能继续回去等待。

公平锁:与非公平锁相比，就是在比较完state时，如果为0，并不是直接去CAS,而是再观察下等待队列是否有线程，有的话去排队 

#### 代码演示公平锁的lock方法
````java
 final void lock() {
            acquire(1);
        }
```
acquire方法

````java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 调用tryAcquire尝试去获取锁，如果获取成功，则方法结束
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 如果获取锁失败，执行acquireQueued方法，将把当前线程排入队尾
        selfInterrupt();
}
````
tryAcquire方法：
```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState(); // 获取锁的状态
    if (c == 0) { // 如果状态是0，表示锁没有被占用
        if (!hasQueuedPredecessors() && // 判断是队列中是否有排队中的线程
            compareAndSetState(0, acquires)) { // 队列中没有排队的线程，则尝试用CAS去获取一下锁
            setExclusiveOwnerThread(current); // 获取锁成功，则将当前占有锁的线程设置为当前线程
            return true;
        }
    }
    // 锁被占用、队列中有排队的线程或者当前线程在获取锁的时候失败将执行下面的代码
    else if (current == getExclusiveOwnerThread()) { // 当前线程是否是占有锁的线程
        int nextc = c + acquires; // 是的话，表示当前线程是重入这把锁，将锁的状态进行加1
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded"); // 锁的重入次数超过int能够表示最大的值，抛出异常
        setState(nextc); // 设置锁的状态
        return true;
    }
    return false; // 没有获取到锁
}
````

#### unlock方法

````java
public void unlock() {
    sync.release(1); // 调用AQS的release方法
}
````
release方法：
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) { // 尝试去释放锁
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); // 释放锁成功，head不为空，并且head的waitStatus不为0的情况下，将唤醒后继节点
        return true;
    }
    return false;
}
````
tryRelease 方法 
```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases; // 将锁的状态减1
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException(); // 准备释放锁的线程不是持有锁的线程，抛出异常
    boolean free = false;
    if (c == 0) {
        free = true; // 锁的状态是0，说明不存在重入的情况了，可以直接释放了
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
````

唤醒后继节点unparkSuccessor()

```java
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus; // 注意，这个node是head节点
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0); // 当前node的状态是小于0，将其状态设置为0

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next; // head节点的后继节点
    if (s == null || s.waitStatus > 0) {
        s = null; // 执行到这表示head的后继节点是1，处于取消的状态
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t; // 从等待队列的队尾向前找，找到倒序的最后一个处于非取消状态的节点
    }
    if (s != null)
        LockSupport.unpark(s.thread); // 唤醒head后面的处于非取消状态的第一个（正序）节点
}
````


非公平锁的代码演示:

#### 非公平锁的lock方法
````java
final void lock() {
    if (compareAndSetState(0, 1)) // 注意！！！在非公平锁中，会先用CAS的方式去尝试更改锁的状态，即尝试去获取锁，不管锁是否被其他线程持有，也不理会等待队列中是否有等待的线程
        setExclusiveOwnerThread(Thread.currentThread()); // 获取锁成功，则将当前占有锁的线程设置为当前线程
    else
        acquire(1); // CAS获取锁失败，则进入acquire方法
}
````

acquire 方法
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 这里将调用NonfairSync的tryAcquire方法
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) // 如果获取锁失败，执行acquireQueued方法，将把当前线程排入队尾
        selfInterrupt();
}
````

tryAcquire方法 
````java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
````
nonfairTryAcquire 方法:
````java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState(); // 获取锁的状态
    if (c == 0) { // 如果状态是0，表示锁没有被占用
        if (compareAndSetState(0, acquires)) { // 注意！！！在非公平锁中，这里不会判断队列中是否有等待的线程，非公平锁会直接去抢占锁
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
````

unlock方法与公平锁一样 

####  公平锁与非公平锁的异同:

- 在公平锁（FairSync）的lock方法中，会直接调用aquire方法；但是在非公平锁（NonfairSync）的lock方法中，会先采用CAS的方式去获取锁，不管是否有其他线程已经占有锁或者是否有其他线程在等待队列中。
- 公平锁（FairSync）调用的tryAcquire方法中，会先去检查等待队列中是否有等待的线程；但是在非公平锁（NonfairSync）调用的nonfairTryAcquire不会去检查等待队列。
- 无论公平锁还是非公平锁，对于排队中的线程，都能保证排在前面的线程一定比排在后面的线程优先获得锁。