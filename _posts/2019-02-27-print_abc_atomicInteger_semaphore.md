---
layout: default
title: 多线程 循环输出ABC AtomicInteger和Semaphore方案
#excerpt: 
---



　　看到一篇文章，说的是“多线程情况，如何循环输出ABC”，里面介绍了好几个方案，其中有一种是使用AtomicInteger原子性类去实现。  

# AtomicInteger方案，循环输出ABC

　　它的思路呢，是三个线程，同时去操作AtomicInteger类，然后对atomicInteger的值取膜操作，模的结果是0，输出“A”；膜的结果是1，输出“B”；膜的结果是2，输出“C”。   

　　代码如下： 
　　
```java
public class AtomicIntegerExample {
    AtomicInteger ai = new AtomicInteger(0);

    public static void main(String[] args) {
        AtomicIntegerExample aie = new AtomicIntegerExample();
        ExecutorService es = Executors.newFixedThreadPool(3);

        es.execute(aie.new A());
        es.execute(aie.new B());
        es.execute(aie.new C());

        es.shutdown();
    }

    class A implements Runnable {
        private int index;
        private String out;

        @Override
        public void run() {
            while (ai.get() < 30) {
                if (ai.get() % 3 == 0) {
                    System.out.println("A");
                    ai.getAndIncrement();
                }
            }
        }
    }

    class B implements Runnable {
        @Override
        public void run() {
            while (ai.get() < 30) {
                if (ai.get() % 3 == 1) {
                    System.out.println("B");
                    ai.getAndIncrement();
                }
            }
        }
    }

    class C implements Runnable {
        @Override
        public void run() {
            while (ai.get() < 30) {
                if (ai.get() % 3 == 2) {
                    System.out.println("C");
                    ai.getAndIncrement();
                }
            }
        }
    }
}
```

　　乍一看，思路是对的；代码放到编译器里，输出结果也是“正确”的。其实，多运行几次，就会看到不同的结果--会多出来一个“A”   



## AtomicInteger方案的不足

　　为什么有时候会多输出呢？因为AtomicInteger类是原子性的，但是，我们的操作不是原子性的。在线程的run()方法中，我们是先检查值是否小于30，如果小于30，再进行取膜和自增。这是两个操作，必然不是原子性，在线程竞争很大的时候，就会暴露问题。  



## AtomicInteger + 双检锁

　　该如何处理呢？使用AtomicInteger类就不能完成“循环输出ABC”这样的需求了嘛？答案是：当然可以。解决方案借鉴了单例模式中使用到的双检锁机制，在run()方法中的if语句内，再加一个判断即可。   

　　代码如下（之前的代码里需要三个类，是没必要的，共用一个类即可）：  

```java
public class AtomicIntegerExample {
    AtomicInteger ai = new AtomicInteger(0);
    final Integer maxValue = 30;

    public static void main(String[] args) {
        AtomicIntegerExample aie = new AtomicIntegerExample();
        ExecutorService es = Executors.newFixedThreadPool(3);

        es.execute(aie.new MyThread(0, "A"));
        es.execute(aie.new MyThread(1, "B"));
        es.execute(aie.new MyThread(2, "C"));

        es.shutdown();
    }

    class MyThread implements Runnable {
        private int index;
        private String out;

        MyThread(int index, String out) {
            this.index = index;
            this.out = out;
        }

        @Override
        public void run() {
            while (ai.get() < maxValue) {
                // int currentValue = ai.getAndIncrement(); 不可以在if外使用getAndIncrement()方法，因为只有在满足条件才可以做自增操作。线程是无序、不可控的，可能某一个线程多次连续执行
                int currentValue = ai.get();
                if (currentValue < maxValue && currentValue % 3 == index) {
                    System.out.print(out);

                    if (index == 2) {
                        System.out.println();
                    }

                    ai.getAndIncrement();
                }
            }
        }
    }
}
```



# Semaphore方案

　　我个人不是非常喜欢使用while无脑循环的这种方式，因为可能一个线程符合while内的条件，但是不符合if内的条件，其实这种是无效的。所以，我更倾向于使用Semaphore信号量的方式去实现，会更加优雅。

　　Semaphore方案，它的思路是：三个信号量，同时只有一个线程执行，输出完再开启下一个信号量，形成一个环状。具体点就是：   

　　信号量A，消费线程，输出"A"，再释放信号量B的线程；   

　　信号量B开启了，消费线程，输出“B”，再释放信号量C的线程；   

　　信号量C开启了，消费线程，输出"C"，在释放信号量A的线程；   

　　再次循环。。。

　　代码如下：   

```java
public class SemaphoreExample {
    public static void main(String[] args) {
        SemaphoreExample se = new SemaphoreExample();
        ExecutorService es = Executors.newFixedThreadPool(3);

        Semaphore semaphoreA = new Semaphore(1);
        Semaphore semaphoreB = new Semaphore(0);
        Semaphore semaphoreC = new Semaphore(0);

        es.execute(se.new MyThread(semaphoreA, semaphoreB, "A"));
        es.execute(se.new MyThread(semaphoreB, semaphoreC, "B"));
        es.execute(se.new MyThread(semaphoreC, semaphoreA, "C"));

        es.shutdown();
    }

    class MyThread implements Runnable {
        private Semaphore currentSemaphore;
        private Semaphore nextSemaphore;
        private String out;

        MyThread(Semaphore currentSemaphore, Semaphore nextSemaphore, String out) {
            this.currentSemaphore = currentSemaphore;
            this.nextSemaphore = nextSemaphore;
            this.out = out;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    currentSemaphore.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.print(out);
                nextSemaphore.release();

                if ("C".equals(out)) {
                    System.out.println();
                }
            }
        }

    }
}
```

　　很明显，信号量的这种方式，更加充分的利用资源。