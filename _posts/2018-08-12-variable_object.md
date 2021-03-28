---
layout: default
title: 可变对象与不可变对象
#excerpt: 
---
　　在之前的一篇文章“python中的赋值 值传递？”中，因为在调用方法时与方法内部，输出id是相同，所以不能理解官方的那句话

> Remember that arguments are passed by assignment in Python.  要记住，Python 里的参数是通过赋值传递的。 

　　之后还是从内存模拟上理解的。其实是我有一点搞错了，官方的话是没问题的，我论证的姿势是错的---两个地方输出id是相同的，并不能说明python是传值还是传地址。官方说只有传值，那去反驳的话，得在调用方法的地方和方法内部，都输出值，两个地方的值不同，这才是正确的反驳姿势。调用id()方法，只是输出内存地址，两个内存地址相同，并不能说明python就不是传值。

　　好了，现在开始讲本文的主题--可变对象和不可变对象，在python中，可变对象包括：list、set、dict、自定义对象，不可对对象包括：int、float、str、bool、tuple等。不可变对象步允许自身内容进行修改。如果我们队一个不可变对象进行赋值，实际上是生成一个新对象，再让变量指向这个对象。哪怕这个对象简单到只是数字0和1

```python
	a=0
	print('a',id(a))
	a=1
	print('a',id(a))
```

输出：

```python
	a 4453151440
	a 4453151472
```
因为对象不可变，所以为了提高效率，python会使用一些公共对象，比如

```python
a=1
print('a',id(a))
b=1
print('b'),id(b))
print(a==b)
print(a is b )
c='hello world'
print('c',id(c))
d='hello world'
print('d',id(d))
print(c==d)
print(c is d)
```

输出：

```python
    a 442376776
    b 442376776
    True
    True
    c 4430180912
    c 4430180912
    True
    True
```

Tips: is 这个操作符，判断是否为同一个对象，也就是地址是否一致。 == 操作符只判断“值”是否相等

　　如果我们需要产生一个list对象的副本，可以通过[:]来获得，对副本赋值一个变量，进行修改，不会影响到之前的list，因为他们已经不是同一个对象了。

　　那么如果是下面请看呢：



```python
	m=[1,2,[3]]
	n=m[:]
	n[1]=4
	n[2][0]=5
	print(m)
```

输出结果是 [1,2,5]，为什么会是这样呢？因为对象的赋值是通过浅拷贝的。