## Java并发编程基础（五）

## 1.读写锁

1. 读写锁在同一个时刻可以拥有多个读线程访问，但是在写线程访问时，所有的读线程和其他写线程均被阻塞。读写锁维护了一对锁，一个读锁和一个写锁，通过读锁和写锁，事得并发性比一般的排它锁有了很大提升。

2. java并发包提供的读写锁实现是ReentranWriteLock(特性):
   - 公平性选则
   - 重入性
   - 锁降级

## 2.多写锁示例

ReadWriteLock仅定义了获取锁和写锁的两个方法，即readLock（）和writeLock方法。

```java
package cn.smallmartial.concurrency;

import jdk.nashorn.internal.ir.CallNode;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @Author smallmartial
 * @Date 2019/8/26
 * @Email smallmarital@qq.com
 */
public class Cache {

    static Map<String,Object> map= new HashMap<String,Object>();
    static ReentrantReadWriteLock rwl= new ReentrantReadWriteLock();
    static Lock r= rwl.readLock();
    static Lock w= rwl.writeLock();
    //获取一个key对应的value
    public static final Object get(String key){
        r.lock();
        try {
            return map.get(key);
        }finally {
            r.unlock();
        }
    }

    //设置key对应的value,并且返回旧的value
    public static final Object put(String key,Object values){
        w.lock();
        try {
            return map.put(key,values);
        } finally {
            w.unlock();
        }
    }
    
    //清空所有的内容
    public static final void clear(){
        w.lock();
        try {
            map.clear();
        }finally {
            w.unlock();
        }
    }
}

```

## 3.读写锁的实现

- 读写状态的设计

  读写锁依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。读写锁需要在同步状态上维护多个读线程和一个写线程的状态，使得改状态的设计成为读写锁实现的关键。

- 写锁的获取与释放

  写锁是一个支持重入的排它锁，如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态。（只用当其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦获取，则其他读写线程的后续访问均被阻塞）

- 读锁的获取与释放

  读锁是一个支持重进入的共享锁，它能够被多个线程同时获取，在没有其他写线程访问（或者写线程为0）时,读锁总会被成功的获取，而所做的也至少增加读状态。

- 锁降级

  锁降级是指写锁降级为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住写锁，在获取读锁，随后释放写锁。