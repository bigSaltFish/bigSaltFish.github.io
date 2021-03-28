---
layout: default
title: 装饰器模式
#excerpt: 

---

　　装饰模式(Decorator Pattern)：动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活（不使用继承）。装饰模式是一种对象结构型模式。

　　java的IO模块，就大量的使用到了装饰模式。
# 简单业务下的代码

## 1.抽象组件

```java
/**
 * 支付的抽象组件
 * 
 * @author Frank.Z
 *
 */
public abstract class PayComponent {
    /**
     * 支付之前，校验参数，判断是否有余额等
     * 
     * @return
     */
    void beforePay() {
        System.out.println("支付之前，校验参数，判断是否有余额等");
    }

    /**
     * 去付款
     */
    abstract void pay();

    /**
     * 支付完毕，记录入库
     */
    void afterPay() {
        System.out.println("支付完毕，记录入库");
    }
}
```
## 2.组件具体实现

```java
/**
 * 支付宝支付，具体的支付，充当ConcreteComponent
 * 
 * @author Frank.Z
 *
 */
public class AliPayConcreteComponent extends PayComponent {

    // 这里的构造方法，是方便演示
    public AliPayConcreteComponent() {
    }

    @Override
    void pay() {
        System.out.println("支付宝支付成功");
    }

}
```
## 3.测试

```java
    /**
     * 一般情况，是下方这样，校验+支付+入库，三步走
     */
    PayComponent aliPay = new AliPayConcreteComponent();
    aliPay.beforePay();
    aliPay.pay();
    aliPay.afterPay();
```



# 需求变更

　　假设，业务逻辑有变动，需要在完成支付后，再进行其他的一个操作。比如，海关要求，海淘电商用支付宝支付必须去报关。那如果要满足需求，有两个方案  

1.硬编码，修改支付类的代码，即在AliPayConcreteComponent类中，重写afterPay()；

2.能否不修改旧代码？



# 装饰器模式 登场

　　使用装饰器模式，不修改支付类的代码，使用构造方法的方式，传入旧支付的对象引用，持有老的逻辑，在老的逻辑的基础上，再增加新的逻辑.这样的话，就不需要更改老代码，也避免了使用继承

## 1.抽象组件

　　同上

## 2.组件具体实现

　　同上

## 3.抽象的装饰器

```java
/**
 * 支付的装饰器
 * 
 * @author Frank.Z
 *
 */
public abstract class PayDecorator extends PayComponent {
    abstract void afterPay();
}
```

## 4.具体装饰器实现

```java
/**
 * 支付宝支付的装饰实现
 * 
 * @author Frank.Z
 *
 */
public class AliPayConcreteDecorator extends PayDecorator {
    PayComponent component;

    AliPayConcreteDecorator(PayComponent component) {
        this.component = component;
    }

    @Override
    void afterPay() {
        component.afterPay();
        System.out.println("海淘电商政策变动，支付宝支付，须在支付完毕后像海关报关");
    }

    @Override
    void pay() {
        component.pay();
    }

}
```
## 5.测试

```java
public class M {
    public static void main(String[] args) {
        PayComponent aliPay = new AliPayConcreteComponent();

        PayComponent aliPayPlus = new AliPayConcreteDecorator(aliPay);
        aliPayPlus.beforePay();
        aliPayPlus.pay();
        aliPayPlus.afterPay();
    }
}
```


tips:上面的步骤3 抽象的装饰器，假如业务没那么复杂的情况下，是可以省略的。