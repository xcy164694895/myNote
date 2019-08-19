## 线程的状态及基本操作 ##

### 概述 ###
先了解一下基本概念。线程是操作系统能够进行运算调度的最小单位。它被包含在进程中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务（多核CPU下才能实现线程并行）。
在单核CPU中，多线程的并发从宏观角度看，是多个线程同时执行，但是从微观角度看，多线程还是需要通过CPU的时间片切换来实现的，同一时间是无法做到多个线程在单个CPU中执行的。在多核CPU中，才能实现多个线程并行执行。在实际应用场景中，具体使用
单线程还是多线程， 需要根据实际场景来做衡量，并不是所有场景都更适合多线程。

### 新建线程的几种方式 ###
Java中新建线程有三种方式：继承Tread类；实现Runnable接口；通过callable和Future实现；

#### 继承Thread ####
1. 定义Thread类的子类，并重写run方法，该类中run方法的方法体就代表了该线程要执行的内容。
2. 创建Thread子类的实例，即创建线程对象。
3. 调用线程对象的start()方法启动线程。
```
    public static void main(String[] args) {
        //第一种创建线程实例的方式
        Thread threadDemo1 = new ThreadDemo1();
        threadDemo1.start();
        //第二种创建线程实例的方式
        Thread threadDemo2 = new Thread(){
            @Override
            public void run(){
                System.out.println("新建线程Demo2！");
            }
        };
        threadDemo2.start();
    }

    static class ThreadDemo1 extends Thread{

        @Override
        public void run() {
            System.out.println("继承Thread类，新建线程Demo1！");
        }
    }
```

#### 实现Runnable ####
1. 实现Runnable接口，重写该接口的run()方法。
2. 创建Runnable实现类的实例，并将此实例作为创建Thread的Target来创建thread的实例，该tread实例才是真正的线程对象。
3. 调用thread实例的start()方法启动线程。
```
public static void main(String[] args) {
        //第一种实现方式
        Runnable runnable = new RunnableDemo1();
        Thread runnableDemo1 = new Thread(runnable);
        runnableDemo1.start();
        //第二种实现方式
        Thread runnableDemo2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"实现Runnable接口，新建RunnableDemo2!");
            }
        });
        runnableDemo2.start();
    }

    static class RunnableDemo1 implements  Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "实现Runnable接口，新建线程RunnableDemo1!");
        }
    }
```

#### 通过Callable和Future ####
1. 实现Callable接口，并重写改接口的call()方法，call()方法的方法体即该类的执行内容。
2. 创建Callable接口实现类的实例，并使用FutureTask来包装callable实例，该FutureTask封装了callable实例的call()方法的返回值。（FutureTask是一个包装器，它通过接受Callable来创建，它同时实现了Future和Runnable接口。）
3. 使用FutureTask实例作为thread的target创建线程。
4. 调用tread的start()方法启动线程。
```
public static void main(String[] args) {
        CallableDemo1 callableDemo1 = new CallableDemo1();
        FutureTask<Integer> futureTask = new FutureTask<>(callableDemo1);
        Thread thread = new Thread(futureTask);
        thread.start();
        try {
            Integer i = futureTask.get();
            System.out.println(Thread.currentThread().getName() + "获取到线程的返回值为:" + i);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }


    static class CallableDemo1 implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            int i = 100;
            System.out.println(Thread.currentThread().getName() +" " + i);
            return i;
        }
    }

```

#### 三种创建方式的比较 ####
+ 由于Java是单继承多实现的，所以尽量使用接口实现的方式创建线程，这样还可以继承其余的类
+ callable接口实现方式，较其余两种相对复杂，但是该实现方式线程执行后有返回值，其余方式没有
+ Thread类实现了Runnable接口

### 线程状态的转换 ###
我们通过查看Thread类的源码，发现线程只有六种状态:NEW(新建)、RUNNABLE(运行)、BLOCKED(阻塞装填)、WAITING(等待状态)、TIMED_WAITING(超时等待状态)、TERMINATED(终止状态)，具体源码如下：
```
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }

```
从源代码注释中可以整理出线程状态转换的过程。当一个线程创建之后，就处于**NEW(新建状态)**,调用Thread.start()后，线程并不会立马执行，而是进入**REDAY(就绪状态)**，等待系统调度该线程之后，进入**RUNNING(运行中状态)**。
CPU执行每个线程是有一个时间限制的，这个时间段被称为时间片，当一个时间片结束后，线程仍在执行中，那么系统会将线程重新置为REDAY（就绪状态）,或者在线程运行时，调用Thread.yeild()方法，该线程一样会被置为READY状态，等待系统再次调用。
当系统调用Object.wait()、Thread.join()、LockSupport.park()方法后，线程状态转换为WAITTING(等待状态)。而同样的，如果调用的是Object.wait(long)、Thread.sleep(long)、Thread.join(long)、LockSupport.parkNanos()、LockSupport.parkUntil()方法时，会进入TIME_WAITTING(超时等待)状态

>注：本文参考：https://www.jianshu.com/p/f65ea68a4a7f ，该文章作者有一系列关于java并发包知识的讲解，值得学习