### 锁类型

是一个可重入锁。

ReentrantLock 内部持有一个 Sync 对象的引用，Sync 是一个抽象静态内部类继承 AQS。

所有的获取锁，释放锁都是通过 Sync 类来操作的。

ReentrantLock 分为**公平锁**和**非公平锁**，可以通过构造方法来指定具体类型：

```java
//默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
//公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

默认一般使用**非公平锁**，它的效率和吞吐量都比公平锁高的多。由于公平锁需要关心队列的情况，得按照队列里的先后顺序来获取锁(会造成大量的线程上下文切换)，而非公平锁则没有这个限制。



### 获取锁

调用 sync 方法，而这个方法是一个抽象方法，具体是由其子类(FairSync)来实现的。

第一步是尝试获取锁 tryAcquire(arg)，这个也是由其子类实现：

```java
protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```



首先会判断 `AQS` 中的 `state` 是否等于 0，0 表示目前没有其他线程获得锁，当前线程就可以尝试获取锁。

公平锁：尝试获取锁之前会利用 `hasQueuedPredecessors()` 方法来判断 AQS 的队列中中是否有其他线程，如果有则不会尝试获取锁。

非公平锁：不判断，直接尝试获取。

如果队列中没有线程就利用 CAS 来将 AQS 中的 state 修改为1，也就是获取锁，获取成功则将当前线程置为获得锁的独占线程。

如果 `state` 大于 0 时，说明锁已经被获取了，则需要判断获取锁的线程是否为当前线程(`ReentrantLock` 支持重入)，是则需要将 `state + 1`，并将值更新。



### 写入队列

如果 `tryAcquire(arg)` 获取锁失败。

* 一是 state = 0，但是是公平锁：CLH 队列中有其它线程等待，所以直接失败，不尝试获取。
* 二是 state > 0：判断如果获取锁的线程不是当前线程则直接失败。

失败之后则需要用 `addWaiter(Node.EXCLUSIVE)` 将当前线程通过 CAS 加自旋的方式写入 CLH 队列的队尾。

写入之前需要将当前线程包装为一个 `Node` 对象，写入之后将当前线程挂起，等待所释放被唤醒。



### 释放锁

公平锁和非公平锁的释放流程都是一样的。

```java
// 释放锁，设置释放值是 1
public void unlock() {
    sync.release(1);
}
public final boolean release(int arg) {
    if (tryRelease(arg)) {
      	// 获取 CLH 队列的头节点
        Node h = head;
        if (h != null && h.waitStatus != 0)
        	   //唤醒被挂起的头节点的线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
//尝试释放锁
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
  	// 必须 c == 0 才能释放锁	
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

首先会判断当前线程是否为获得锁的线程，由于是重入锁所以需要将 `state` 减到 0 才认为完全释放锁。

释放之后需要调用 `unparkSuccessor(h)` 来唤醒被挂起的线程。

