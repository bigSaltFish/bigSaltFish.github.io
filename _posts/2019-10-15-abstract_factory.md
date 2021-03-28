---
layout: default
title: 抽象工厂模式
#excerpt: 

---

　　抽象工厂，是生成一组相互关联的产品  
　　比较于工厂方法，工厂方法是多个产品单个层次的产品,比如生产单独的墨水或者笔，这时候就很适合工厂方法，它没有维护组件之间的关系  
　　假如，要生产一套文具，文具是由同一个系列的笔、墨、纸组成的，它们之间是相互关联的，中国的毛笔应该在宣纸上书写文字，中国的毛笔不能在英国的信纸上书写，因为信纸太薄了，这时候，就适合抽象工厂了  

# 1. 笔工厂
```java
public interface PenFactory {
    void createPen();
}

public class ChinesePen implements PenFactory {
    @Override
    public void createPen() {
        System.out.println("我是中国的毛笔");
    }
}

public class EnglishPen implements PenFactory {
    @Override
    public void createPen() {
        System.out.println("我是英国的鹅毛笔");
    }
}
```

# 2. 墨工厂
```java
public interface InkFactory {
    void createInk();
}

public class ChineseInk implements InkFactory {
    @Override
    public void createInk() {
        System.out.println("中国墨水");
    }
}

public class EnglishInk implements InkFactory {
    @Override
    public void createInk() {
        System.out.println("英国墨水");
    }
}
```

# 3.文具工厂


```java
/**
 * 一套文具
 * 
 *
 */
public interface StationeryFactory {
    void createInk();

    void createPen();
}

/**
 * 中式文具一套
 * 
 *
 */
public class ChineseStationeryFactory implements StationeryFactory {
    @Override
    public void createInk() {
        InkFactory chineseInk = new ChineseInk();
        chineseInk.createInk();
    }

    @Override
    public void createPen() {
        PenFactory chinesePen = new ChinesePen();
        chinesePen.createPen();
    }
}

/**
 * 得到一套英式风格的文具
 * 
 *
 */
public class EnglishStationeryFactory implements StationeryFactory {
    @Override
    public void createInk() {
        InkFactory englishInk = new EnglishInk();
        englishInk.createInk();

    }

    @Override
    public void createPen() {
        PenFactory englishPen = new EnglishPen();
        englishPen.createPen();
    }
}
```



# 4.测试

```java
public class M {
    public static void main(String[] args) {
        /**
         * 得到一套中式文具
         */
        StationeryFactory chineseStationery = new ChineseStationeryFactory();
        chineseStationery.createPen();
        chineseStationery.createInk();

        /**
         * 得到一套英式文具
         */
        StationeryFactory englishStationery = new ChineseStationeryFactory();
        englishStationery.createPen();
        englishStationery.createInk();
    }
}
```
