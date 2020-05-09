### 多线程 
### JMM(Java内存模型 Java Memory Model)

    本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范,通过这种规范定义了程序重各个变量的访问方式 
#### JMM线程安全三大特性(大多数线程安全开发需要遵守)

- 可见性
- 原子性(synchronized满足，volatile不满足)
- 有序性

### volatile

volatile是Java虚拟机提供的轻量级的同步机制(synchronized)。

### volatile三大特性
- 保证可见性

    JVM运行程序的实体是线程，每个线程创建时JVM会为其创建一个工作内存，Java内存模型重规定所有变量都存储在**主内存，主内存是共享内存区域，但线程对变量的操作必须在工作内存重进行，首先讲变量从主内存中拷贝到工作内存，操作结束后再将内存写回主内存。**当多个线程进行一个操作时，其中一个线程已经操作完后，写回主内存了，需要告诉其他线程不需要再进行操作，叫做保证可见性。
- 不保证原子性
    原子性:不可分割、完整性，也即某个线程正在做某个具体业务时，中间不可以被加载或者分割，要么同时成功，要么同时失败。
    解决:- 加sync
        - AtomicInteger(底层 CAS)
- 禁止指令重排

```java
class MyData {
    volatile int number = 0;

    public void addPlus() {
        number++;
    }
}

public class VolatileTest {
    public static void main(String[] args) {
        MyData myData = new MyData();
        for (int i = 0; i < 20; i++) {
            new Thread(
                    () -> {
                        for (int j = 0; j < 1000; j++) {
                            myData.addPlus();

                        }
                    }
                    , String.valueOf(i)).start();
        }
        while (Thread.activeCount()>2){
            Thread.yield();
        }
        System.out.println(myData.number);
    }
}
```


### volatile应用场景

DCL(double check lock) 双端检锁 机制不一定线程安全，原因是有指令重排的存在，加入volatile可以禁止指令重排。

- 单例模式
- 读写锁写入缓存
- cas里ouc包大规模使用了volatile



### CAS

#### 什么是CAS:

全称Compare and swap，字面意思:“比较并交换”。**它是一条CPU并发原语。**

Unsafe是CAS的核心类，Unsafe类存在于sun.misc包中，其内部方法可以像C的指针一样直接操作内存，Java中的CAS操作的执行依赖于Unsafe类的方法。

注意:Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行对应的任务。
(native关键字说明其修饰的方法是一个原生态方法，方法对应的实现不是在当前文件，而是在用其他语言（如C和C++）实现的文件中。Java语言本身不能对操作系统底层进行访问和操作，但是可以通过JNI接口调用其他语言来实现对底层的访问。)

![](/img/java/CAS.png)

Unsafe类中的compareAndSwapInt方法 是一个本地方法(底层汇编)

```java
  public final int getAndIncrement() {
      //this是当前对象 valueOffset 是内存偏移量
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }

```
假设两个线程同事执getAndAddInt操作(分别在不同的CPU上)。

- AtomicInteger里面的value原始值为3，即主内存中的value为3，根据JMM 模型，A，B两个线程同事拿到值为3的value副本到各自的工作内存。
- 这时 A 通过getIntVolatile(var1,var2) 拿到value值，这时 A挂起了，
- B通过getIntVolatile(var1,var2) 拿到value值,并执行compareAndSwapInt方法，比较内存值为3，成功修改内存值为4，
- 这时 A恢复了，执行compareAndSwapInt方法 发现3和4不一样，执行重新再来一遍。直至成功
  
```java
 public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

#### CAS的缺点
 - 循环时间长开销很大。 
  
    如果CAS失败，会一直进行尝试。如果CAS长时间一直不成功，可能会给CPU带来很大的开销。
 - 只能保证一个共享变量的原子操作。
    
    当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。
 - ABA问题。

    如果内存地址V初次读取的值是A，并且在准备赋值的时候检查到它的值仍然为A，那我们就能说它的值没有被其他线程改变过了吗？如果在这段期间它的值曾经被改成了B，后来又被改回为A，那CAS操作就会误认为它从来没有被改变过。这个漏洞称为CAS操作的“ABA”问题。Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性。因此，在使用CAS前要考虑清楚“ABA”问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。



### 集合类不安全

#### ArrayList

- 问题: 引发juc java.util.ConcurrentModificationException

- 原因:并发争抢修改导致 。

- 解决方案: 
  - 1: new Vector<>();
  - 2:Collections.synchronizedList(new ArrayList<>());
  - 3:new CopyOnWriteArrayList();(建议)

CopyOnWrite容器即写时复制的容器，往一个容器添加元素时，不直接往当前容器添加，而是复制出来一个新容器，插到最后，添加结束后，将原容器的引用只想新的容器。优点:可以对CopyOnWrite容器进行并发的读而不需要加锁，因为当前容器不会添加任何元素。所以这个容器是一种读写分离的思想。

```java
  /**
     * Appends the specified element to the end of this list. 插入最后
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    ```

