### AQS 是什么

`AQS(AbstractQueuedSynchronizer)`是一个抽象同步队列，`JUC(java.util.concurrent)`中很多同步锁都是基于`AQS`实现的。

`AQS`的基本原理就是当一个线程请求共享资源的时候会判断是否能够成功操作这个共享资源，如果可以就会把这个共享资源设置为锁定状态，如果当前共享资源已经被锁定了，那就把这个请求的线程阻塞住，也就是放到队列中等待。



### state变量

- `AQS`中有一个被`volatile`声明的变量用来表示同步状态
- 提供了`getState()`、`setState()`和`compareAndSetState()`方法来修改`state`状态的值

```java
// 返回同步状态的当前值
protected final int getState() {  
  return state;
}

// 设置同步状态的值
protected final void setState(int newState) { 
  state = newState;
}

// CAS操作修改state的值
protected final boolean compareAndSetState(int expect, int update) {
  return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```



### 对共享资源的操作方式

#### 获取锁

获取资源的时候会调用`acquire()`方法，在这里面会调用`tryAcquire()`方法去设置`state`变量，如果失败的话。就把当前线程封装成一个`Node`中存入 AQS 的CLH 队列。

#### 释放锁

释放资源的时候是调用`realase()`方法，会调用`tryRelease()`方法修改`state`变量，调用成功后会去唤醒队列中`Node`里的线程，`unparkSuccessor()`方法就是判断当前`state`变量是否符合唤醒的标准，如果合适就唤醒，否则继续放回队列。



### 条件变量Condition

`Condition`中的`signal()`和`await()`方法类似与`notify()`和`wait()`方法，需要和`AQS`锁配合使用。

```java
public static void main(String[] args) throws InterruptedException {

    ReentrantLock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    Thread thread1 = new Thread(new Runnable() {
        @Override
        public void run() {
            lock.lock();
            System.out.println(" t1 加锁");
            System.out.println("t1 start await");
            try {
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("t1 end await");
            lock.unlock();
        }
    });
    thread1.start();

    Thread thread2 = new Thread(new Runnable() {
        @Override
        public void run() {
            lock.lock();
            System.out.println(" t2 加锁");
            System.out.println("t2 start signal");
            condition.signal();
            System.out.println("t2 end signal");
            lock.unlock();
        }
    });
    thread2.start();
}
```



#### 在AQS中的原理

上面`lock.newCondition()`其实是`new`一个`AQS`中`ConditionObject`内部类的对象出来。

这个对象里面有一个队列，当调用`await()`方法的时候会存入一个`Node`节点到这个队列中，并且调用`park()`方法阻塞当前线程，释放当前线程的锁。

而调用`singal()`方法则会移除内部类中的队列头部的`Node`，然后放入`AQS`中的 CLH 队列中等待执行机会。