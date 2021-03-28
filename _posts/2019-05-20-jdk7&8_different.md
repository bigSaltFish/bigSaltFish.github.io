---
layout: default
title: jdk7和8 内存可见性上的一个小细节
#excerpt: 
---

　　在看一篇关于jvm内存可见性的文章的时候，无意中发现了jdk7和jdk8在处理上的细微差别，导致了两种截然不同的效果：jdk7(文中使用的是1.7.0_67)会顺利执行完；jdk8(文中使用的是1.8.0_192)会一直卡住。  

　　下面是具体代码：  

```java
public class ThreadSafeCacheJDK7 {
    private int result;

    public Integer getResult() {
        return result;
    }

    public synchronized void setResult(Integer result) {
        this.result = result;
    }

    public static void main(String[] args) {
        System.out.println(System.getProperty("java.version"));
        final ThreadSafeCacheJDK7 t = new ThreadSafeCacheJDK7();

        new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (t.getResult() < 8) {
                    i++;
                }
                System.out.println(i);
            }
        }).start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        t.setResult(10);
    }
}
```

　　上面代码，能正常输出。  



```java
public class ThreadSafeCacheJDK8 {
    int result;

    public int getResult() {
        return result;
    }

    public synchronized void setResult(int result) {
        this.result = result;
    }

    public static void main(String[] args) {
        System.out.println(System.getProperty("java.version"));
        ThreadSafeCacheJDK8 threadSafeCache = new ThreadSafeCacheJDK8();

        for (int i = 0; i < 8; i++) {
            new Thread(() -> {
                int x = 0;
                while (threadSafeCache.getResult() < 100) {
                    x++;
                }
                System.out.println(x);
            }).start();
        }

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        threadSafeCache.setResult(200);
    }
}
```
　　上面的代码，就不能正常输出，会一直卡住。  



　　如果需要让jdk8的代码正常执行完，需要使用 *volatile* 修饰 result